# Enumeration

## Aperçu

L’énumération est le processus qui consiste à recueillir des informations détaillées sur un service cible après l’avoir identifié. À cette étape, l’objectif est d’extraire des données utiles comme les versions de services, les utilisateurs, les partages, les répertoires, les détails de configuration et d’autres informations pouvant aider lors de l’exploitation ou d’une phase ultérieure de post-exploitation.

Les modules auxiliaires sont particulièrement utiles pour l’énumération, car ils permettent de réaliser des scans, de la découverte, du fuzzing et des vérifications spécifiques à chaque service. Ils peuvent être utilisés pendant la collecte d’informations, après l’accès initial, et même après un pivot vers un autre sous-réseau.

## Scan de ports avec les modules auxiliaires

Les modules auxiliaires peuvent être utilisés pour scanner les ports TCP et UDP, découvrir des hôtes et énumérer les services sur une cible ou sur un réseau interne.

### Commandes utiles

Scanner un seul hôte :

```bash
use auxiliary/scanner/portscan/tcp
set RHOSTS <target_ip>
run
```

Scanner un sous-réseau :

```bash
use auxiliary/scanner/portscan/tcp
set RHOSTS <target_network>/24
run
```

Exemple de scan UDP :

```bash
use auxiliary/scanner/discovery/udp_probe
set RHOSTS <target_ip>
run
```

## Énumération FTP

FTP utilise le port TCP 21 et sert au transfert de fichiers entre un client et un serveur. Il est aussi parfois utilisé pour transférer des fichiers vers et depuis le répertoire d’un serveur web.

L’authentification FTP nécessite normalement un nom d’utilisateur et un mot de passe, mais certains serveurs autorisent une connexion anonyme s’ils sont mal configurés.

### Commandes utiles

Vérifier la version de FTP :

```bash
use auxiliary/scanner/ftp/ftp_version
set RHOSTS <target_ip>
run
```

Vérifier la connexion anonyme :

```bash
use auxiliary/scanner/ftp/anonymous
set RHOSTS <target_ip>
run
```

Force brute des identifiants FTP :

```bash
use auxiliary/scanner/ftp/ftp_login
set RHOSTS <target_ip>
set USERNAME <user>
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

## Énumération SMB

SMB est un protocole de partage de fichiers utilisé sur les réseaux locaux. Il fonctionne normalement sur le port TCP 445, bien qu’il utilisait à l’origine aussi le port 139 via NetBIOS. SAMBA est l’implémentation Linux de SMB.

Avec les modules auxiliaires SMB, tu peux énumérer la version de SMB, les partages, les utilisateurs, et aussi effectuer des attaques par force brute.

### Commandes utiles

Vérifier la version de SMB :

```bash
use auxiliary/scanner/smb/smb_version
set RHOSTS <target_ip>
run
```

Énumérer les partages :

```bash
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS <target_ip>
run
```

Énumérer les utilisateurs :

```bash
use auxiliary/scanner/smb/smb_enumusers
set RHOSTS <target_ip>
run
```

Force brute des identifiants SMB :

```bash
use auxiliary/scanner/smb/smb_login
set RHOSTS <target_ip>
set USER_FILE users.txt
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

## Énumération des serveurs web

Un serveur web est un logiciel qui fournit des données de site web via HTTP. HTTP utilise le port TCP 80, et les serveurs web les plus connus incluent Apache, Nginx et Microsoft IIS.

Avec les modules auxiliaires, tu peux identifier la version du serveur web, inspecter les en-têtes HTTP et effectuer du brute-force sur les répertoires.

### Commandes utiles

Vérifier le titre HTTP :

```bash
use auxiliary/scanner/http/title
set RHOSTS <target_ip>
run
```

Vérifier les en-têtes HTTP :

```bash
use auxiliary/scanner/http/http_header
set RHOSTS <target_ip>
run
```

Brute-force des répertoires :

```bash
use auxiliary/scanner/http/dir_scanner
set RHOSTS <target_ip>
set PATH /usr/share/wordlists/dirb/common.txt
run
```

## Énumération MySQL

MySQL est un système de gestion de base de données relationnelle open source basé sur SQL. Il est couramment utilisé pour stocker des données d’application et de site web, et fonctionne généralement sur le port TCP 3306.

Les modules auxiliaires peuvent être utilisés pour identifier la version de MySQL, effectuer une force brute sur les mots de passe et exécuter des requêtes SQL.

### Commandes utiles

Vérifier la version de MySQL :

```bash
use auxiliary/scanner/mysql/mysql_version
set RHOSTS <target_ip>
run
```

Force brute des identifiants MySQL :

```bash
use auxiliary/scanner/mysql/mysql_login
set RHOSTS <target_ip>
set USERNAME root
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

## Énumération SSH

SSH est un protocole d’administration à distance qui fournit un accès chiffré et qui succède à Telnet. Il utilise généralement le port TCP 22.

Les modules auxiliaires peuvent être utilisés pour identifier la version de SSH et effectuer une force brute sur les identifiants.

### Commandes utiles

Vérifier la version de SSH :

```bash
use auxiliary/scanner/ssh/ssh_version
set RHOSTS <target_ip>
run
```

Force brute des identifiants SSH :

```bash
use auxiliary/scanner/ssh/ssh_login
set RHOSTS <target_ip>
set USERNAME <user>
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

## Énumération SMTP

SMTP est le protocole utilisé pour l’envoi d’e-mails. Il fonctionne généralement sur le port TCP 25, et peut aussi fonctionner sur les ports 465 et 587.

Les modules auxiliaires peuvent être utilisés pour identifier la version de SMTP et énumérer les utilisateurs.

### Commandes utiles

Vérifier la version de SMTP :

```bash
use auxiliary/scanner/smtp/smtp_version
set RHOSTS <target_ip>
run
```

Énumérer les utilisateurs :

```bash
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS <target_ip>
run
```

## Pourquoi c’est important

L’énumération t’aide à recueillir les détails nécessaires pour décider quel service attaquer et comment l’attaquer. Les modules auxiliaires rendent cela plus rapide car ils automatisent les scans et les vérifications spécifiques aux services.

## Idée clé

L’énumération consiste à transformer un service découvert en informations utiles. Une fois que tu identifies la version du service, les utilisateurs, les partages, les en-têtes ou les identifiants, tu peux passer à l’exploitation avec beaucoup plus de contexte.
