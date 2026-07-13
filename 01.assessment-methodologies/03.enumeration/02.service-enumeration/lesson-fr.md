# Service Enumeration

L'**énumération de services** est le processus d'identification et d'analyse des services réseau en cours d'exécution sur un système cible. En pentest, cette phase vient après le **footprinting et le scan**, et se concentre sur la collecte d'informations détaillées sur des services comme **FTP, SMB, Web, MySQL, SSH et SMTP** pour trouver des mauvaises configurations, des credentials faibles et des vulnérabilités potentielles.

Cette leçon couvre l'**énumération de services utilisant les modules auxiliaires Metasploit**, y compris le scan de ports, la sélection de module, la configuration RHOST et les tests de credentials avec des wordlists, tout cela dans la **phase de reconnaissance/footprinting** (sans exploitation).

---

## Pourquoi l'Énumération de Services Est Importante

L'énumération de services t'aide à :

- Identifier les **services en cours d'exécution** et leurs versions.
- Découvrir les **partages ouverts** (SMB), l'**accès FTP anonyme** et les **dossiers web**.
- Trouver des **credentials faibles ou par défaut** (SSH, MySQL, FTP).
- Détecter les **malconfigurations** qui pourraient mener à un accès non autorisé.
- Collecter des informations pour les **phases suivantes de pentest**.

Metasploit fournit des **modules auxiliaires de scanner** conçus spécifiquement pour l'énumération de services sans exploitation.

---

## Flux de Travail Metasploit pour l'Énumération de Services

### Flux Général

1. **Lancer Metasploit**: `msfconsole`
2. **Créer un workspace**: `workspace -a service_enum_target`
3. **Rechercher des modules auxiliaires**: `search auxiliary/scanner`
4. **Sélectionner le module**: `use auxiliary/scanner/...`
5. **Définir la cible**: `set RHOSTS <target_ip>`
6. **Configurer les options** (ports, threads, wordlists): `set ...`
7. **Exécuter**: `run` ou `exploit`

---

## 1. Énumération FTP

FTP est l'un des services les plus courants à énumérer. Tu vérifies le **login anonyme**, les **partages ouverts** et les **credentials faibles**.

### Étape 1: Scan de Ports pour Trouver FTP
```bash
db_nmap -sS -sV -p 21 <target_ip>
```

### Étape 2: Vérifier l'Accès FTP Anonyme
```bash
msf6 > use auxiliary/scanner/ftp/anonymous
msf6 auxiliary(scanner/ftp/anonymous) > set RHOSTS <target_ip>
RHOSTS => <target_ip>
msf6 auxiliary(scanner/ftp/anonymous) > run
```

Si le login anonyme réussit, tu as trouvé un partage FTP ouvert.

### Étape 3: Brute-force de Login FTP (Test de Credentials)
```bash
msf6 > use auxiliary/scanner/ftp/ftp_login
msf6 auxiliary(scanner/ftp/ftp_login) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/ftp/ftp_login) > set USERPASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
msf6 auxiliary(scanner/ftp/ftp_login) > run
```

Ou utilise des wordlists séparées pour utilisateur et mot de passe :
```bash
msf6 > use auxiliary/scanner/ftp/ftp_login
msf6 auxiliary(scanner/ftp/ftp_login) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/ftp/ftp_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
msf6 auxiliary(scanner/ftp/ftp_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
msf6 auxiliary(scanner/ftp/ftp_login) > run
```

**Sorties clés** :
- `+` indique un login réussi.
- `-` indique un login échoué.

---

## 2. Énumération SMB

SMB (Server Message Block) est utilisé pour le partage de fichiers dans Windows et Linux (Samba). L'énumération se concentre sur les **partages**, les **utilisateurs** et le **mode de sécurité**.

### Étape 1: Scan de Ports pour Trouver SMB
```bash
db_nmap -sS -sV -p 139,445 <target_ip>
```

### Étape 2: Énumérer les Partages SMB
```bash
msf6 > use auxiliary/scanner/smb/smb_enumshares
msf6 auxiliary(scanner/smb/smb_enumshares) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/smb/smb_enumshares) > run
```

Cela liste tous les dossiers partagés accessibles sur la cible.

### Étape 3: Énumérer les Utilisateurs SMB
```bash
msf6 > use auxiliary/scanner/smb/smb_enumusers
msf6 auxiliary(scanner/smb/smb_enumusers) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/smb/smb_enumusers) > run
```

Cela liste les comptes utilisateur sur le serveur SMB.

### Étape 4: Mode de Sécurité SMB
```bash
msf6 > use auxiliary/scanner/smb/smb_security_mode
msf6 auxiliary(scanner/smb/smb_security_mode) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/smb/smb_security_mode) > run
```

Cela révèle la configuration de sécurité SMB (ex. NTLMv1 vs NTLMv2).

### Étape 5: Brute-force de Login SMB
```bash
msf6 > use auxiliary/scanner/smb/smb_login
msf6 auxiliary(scanner/smb/smb_login) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/smb/smb_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
msf6 auxiliary(scanner/smb/smb_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
msf6 auxiliary(scanner/smb/smb_login) > run
```

---

## 3. Énumération de Serveur Web (Apache)

L'énumération de serveur web se concentre sur les **dossiers**, les **en-têtes**, les **versions** et les **vulnérabilités**.

### Étape 1: Scan de Ports pour Trouver Web
```bash
db_nmap -sS -sV -p 80,443 <target_ip>
```

### Étape 2: Énumération des En-têtes HTTP
```bash
msf6 > use auxiliary/scanner/http/http_header
msf6 auxiliary(scanner/http/http_header) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/http/http_header) > set RPORT 80
msf6 auxiliary(scanner/http/http_header) > run
```

Cela montre les en-têtes du serveur (ex. version Apache, en-têtes de sécurité).

### Étape 3: Brute-force de Dossiers (Énumération Web)
```bash
msf6 > use auxiliary/scanner/http/dir_scanner
msf6 auxiliary(scanner/http/dir_scanner) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/http/dir_scanner) > set RPORT 80
msf6 auxiliary(scanner/http/dir_scanner) > set WORDLIST /usr/share/metasploit-framework/data/wordlists/dirbuster.txt
msf6 auxiliary(scanner/http/dir_scanner) > set THREADS 10
msf6 auxiliary(scanner/http/dir_scanner) > run
```

Cela découvre des dossiers cachés comme `/admin`, `/backup`, `/config`.

### Étape 4: Détection de Version Apache
```bash
msf6 > use auxiliary/scanner/http/http_version
msf6 auxiliary(scanner/http/http_version) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/http/http_version) > run
```

---

## 4. Énumération MySQL

L'énumération MySQL se concentre sur la **détection de version**, la **liste de bases de données** et les **tests d'authentification**.

### Étape 1: Scan de Ports pour Trouver MySQL
```bash
db_nmap -sS -sV -p 3306 <target_ip>
```

### Étape 2: Énumération d'Informations MySQL
```bash
msf6 > use auxiliary/scanner/mysql/mysql_info
msf6 auxiliary(scanner/mysql/mysql_info) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/mysql/mysql_info) > run
```

Cela montre la version MySQL et les détails de configuration.

### Étape 3: Brute-force de Login MySQL
```bash
msf6 > use auxiliary/scanner/mysql/mysql_login
msf6 auxiliary(scanner/mysql/mysql_login) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/mysql/mysql_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
msf6 auxiliary(scanner/mysql/mysql_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
msf6 auxiliary(scanner/mysql/mysql_login) > run
```

Credentials par défaut courants : `root:`, `root:root`, `root:password`.

---

## 5. Énumération SSH

L'énumération SSH se concentre sur la **détection d'algorithmes**, la **vérification de hostkey** et les **tests de login**.

### Étape 1: Scan de Ports pour Trouver SSH
```bash
db_nmap -sS -sV -p 22 <target_ip>
```

### Étape 2: Détection de Version et Algorithme SSH
```bash
msf6 > use auxiliary/scanner/ssh/ssh_version
msf6 auxiliary(scanner/ssh/ssh_version) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/ssh/ssh_version) > run
```

Cela montre la version SSH (ex. SSH-2.0-OpenSSH_8.4).

### Étape 3: Énumération de Hostkey SSH
```bash
msf6 > use auxiliary/scanner/ssh/ssh_hostkey
msf6 auxiliary(scanner/ssh/ssh_hostkey) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/ssh/ssh_hostkey) > run
```

Cela récupère l'empreinte (fingerprint) du hostkey SSH.

### Étape 4: Brute-force de Login SSH (Test de Credentials)
```bash
msf6 > use auxiliary/scanner/ssh/ssh_login
msf6 auxiliary(scanner/ssh/ssh_login) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/ssh/ssh_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
msf6 auxiliary(scanner/ssh/ssh_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
msf6 auxiliary(scanner/ssh/ssh_login) > THREADS 10
msf6 auxiliary(scanner/ssh/ssh_login) > run
```

Cela teste les combinaisons utilisateur/mot de passe contre SSH. Les logins réussis montrent `+`.

**Chemins des wordlists** :
- `/usr/share/metasploit-framework/data/wordlists/default_users.txt`
- `/usr/share/metasploit-framework/data/wordlists/default_passwords.txt`
- `/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt`

---

## 6. Énumération SMTP (Bonus)

L'énumération SMTP se concentre sur l'**énumération d'utilisateurs** et les **tests de commandes**.

### Étape 1: Scan de Ports pour Trouver SMTP
```bash
db_nmap -sS -sV -p 25 <target_ip>
```

### Étape 2: Énumération d'Utilisateurs SMTP
```bash
msf6 > use auxiliary/scanner/smtp/smtp_enum
msf6 auxiliary(scanner/smtp/smtp_enum) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/smtp/smtp_enum) > run
```

Cela liste les utilisateurs de courrier sur le serveur SMTP en utilisant les commandes VRFY/EXPN.

---

## Exemple de Flux Complet d'Énumération de Services

Voici un flux complet du scan à l'énumération de services pour une seule cible :

```bash
# 1. Lancer Metasploit
msfconsole

# 2. Créer workspace
workspace -a service_enum_192.168.1.10

# 3. Scan initial avec stockage en base de données
db_nmap -sS -sV -p- -oX full_scan.xml 192.168.1.10

# 4. Importer résultats de scan
db_import full_scan.xml

# 5. Lister services découverts
services

# 6. Énumération FTP
use auxiliary/scanner/ftp/anonymous
set RHOSTS 192.168.1.10
run

# 7. Énumération SMB
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS 192.168.1.10
run

use auxiliary/scanner/smb/smb_enumusers
set RHOSTS 192.168.1.10
run

# 8. Énumération Web
use auxiliary/scanner/http/dir_scanner
set RHOSTS 192.168.1.10
set RPORT 80
set THREADS 10
run

# 9. Énumération MySQL
use auxiliary/scanner/mysql/mysql_login
set RHOSTS 192.168.1.10
set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
run

# 10. Énumération SSH
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 192.168.1.10
set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
set THREADS 10
run

# 11. Exporter résultats
db_export -f xml enumeration_results.xml
```

---

## Résumé des Modules Auxiliaires de Scanner Metasploit

| Service | Port | Module Auxiliaire | Objectif |
|---------|------|-------------------|----------|
| **FTP** | 21 | `auxiliary/scanner/ftp/anonymous` | Vérifier login anonyme |
| **FTP** | 21 | `auxiliary/scanner/ftp/ftp_login` | Brute-force de login |
| **SMB** | 139,445 | `auxiliary/scanner/smb/smb_enumshares` | Énumérer partages |
| **SMB** | 139,445 | `auxiliary/scanner/smb/smb_enumusers` | Énumérer utilisateurs |
| **SMB** | 139,445 | `auxiliary/scanner/smb/smb_login` | Brute-force de login |
| **Web** | 80,443 | `auxiliary/scanner/http/dir_scanner` | Brute-force de dossiers |
| **Web** | 80,443 | `auxiliary/scanner/http/http_header` | Énumération d'en-têtes |
| **Web** | 80,443 | `auxiliary/scanner/http/http_version` | Détection de version |
| **MySQL** | 3306 | `auxiliary/scanner/mysql/mysql_info` | Informations service |
| **MySQL** | 3306 | `auxiliary/scanner/mysql/mysql_login` | Brute-force de login |
| **SSH** | 22 | `auxiliary/scanner/ssh/ssh_version` | Détection de version |
| **SSH** | 22 | `auxiliary/scanner/ssh/ssh_hostkey` | Énumération de hostkey |
| **SSH** | 22 | `auxiliary/scanner/ssh/ssh_login` | Brute-force de login |
| **SMTP** | 25 | `auxiliary/scanner/smtp/smtp_enum` | Énumération d'utilisateurs |

---

## Commandes Clés Metasploit pour l'Énumération de Services

| Commande | Objectif |
|----------|----------|
| `msfconsole` | Lancer console Metasploit |
| `workspace -a <nom>` | Créer nouveau workspace |
| `db_nmap <options> <target>` | Exécuter Nmap avec stockage en BD |
| `db_import <fichier>` | Importer résultats de scan |
| `hosts` | Lister hôtes découverts |
| `services` | Lister services découverts |
| `search auxiliary/scanner` | Rechercher modules de scanner |
| `use auxiliary/scanner/...` | Sélectionner module |
| `set RHOSTS <ip>` | Définir IP de la cible |
| `set RPORT <port>` | Définir port de la cible |
| `set USER_FILE <chemin>` | Définir wordlist utilisateurs |
| `set PASS_FILE <chemin>` | Définir wordlist mots de passe |
| `set THREADS <nombre>` | Définir nombre de threads |
| `run` ou `exploit` | Exécuter module |
| `db_export -f xml <fichier>` | Exporter BD en XML |

---

## Chemins des Wordlists dans Metasploit

Metasploit inclut des wordlists intégrées pour les tests de credentials :

```bash
/usr/share/metasploit-framework/data/wordlists/default_users.txt
/usr/share/metasploit-framework/data/wordlists/default_passwords.txt
/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
/usr/share/metasploit-framework/data/wordlists/dirbuster.txt
```

Utilise-les avec les options `USER_FILE`, `PASS_FILE` et `WORDLIST`.

---

## Points Clés

- L'**énumération de services** identifie et analyse les services en cours d'exécution (FTP, SMB, Web, MySQL, SSH, SMTP).
- Les **modules auxiliaires de scanner** Metasploit sont conçus pour la reconnaissance sans exploitation.
- Commence toujours par le **scan de ports** (`db_nmap`) pour identifier les ports ouverts et services.
- Utilise **`set RHOSTS`** pour spécifier l'IP de la cible pour chaque module.
- **FTP**: Vérifie login anonyme avec `auxiliary/scanner/ftp/anonymous`.
- **SMB**: Énumère partages et utilisateurs avec `smb_enumshares` et `smb_enumusers`.
- **Web (Apache)**: Utilise `dir_scanner` pour brute-force de dossiers et `http_header` pour informations de version.
- **MySQL**: Utilise `mysql_info` pour détails de service et `mysql_login` pour tests de credentials.
- **SSH**: Utilise `ssh_version`, `ssh_hostkey` et `ssh_login` pour version, hostkey et tests de credentials.
- Le **brute-force de login SSH** utilise `auxiliary/scanner/ssh/ssh_login` avec wordlists utilisateur et mot de passe.
- **SMTP**: Utilise `smtp_enum` pour énumération d'utilisateurs.
- Les wordlists sont dans `/usr/share/metasploit-framework/data/wordlists/`.
- Utilise des **workspaces** pour garder les évaluations organisées.
