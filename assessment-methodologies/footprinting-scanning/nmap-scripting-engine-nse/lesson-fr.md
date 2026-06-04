# Nmap Scripting Engine (NSE)

Le **Nmap Scripting Engine (NSE)** est l'une des fonctionnalités les plus puissantes de Nmap. Il te permet d'exécuter des **scripts** contre des hôtes cibles pour effectuer une **enumération avancée de services**, une **détection de vulnérabilités** et une **collecte d'informations** de manière automatisée et personnalisable.

## Qu'est-ce que NSE ?

NSE utilise des **scripts Lua** pour étendre les capacités de Nmap au-delà du simple scan de ports. Chaque script est conçu pour effectuer une tâche spécifique, comme :

- Énumérer des **utilisateurs** (FTP, SMTP, SMB, SSH).
- Détecter les **versions de services** et configurations.
- Trouver des **vulnérabilités** (CVEs, malconfigurations).
- Collecter des structures de **dossiers web**.
- Extraire des **enregistrements DNS** et informations réseau.

Les scripts NSE sont stockés dans `/usr/share/nmap/scripts/` sur la plupart des systèmes Linux et sont catégorisés par fonction (default, discovery, vuln, brute, auth, etc.).

---

## Exécuter des Scripts NSE

### Scripts par Défaut (`-sC`)
L'option `-sC` exécute la **catégorie par défaut de scripts NSE**, qui inclut des scripts sûrs et couramment utiles pour l'énumération de base :

```bash
nmap -sS -sC 192.168.1.10
```

### Scripts Spécifiques (`--script`)
Tu peux exécuter des scripts ou catégories de scripts spécifiques par nom :

```bash
nmap -sS --script http-enum 192.168.1.10
nmap -sS --script vuln 192.168.1.10
nmap -sS --script ftp-anon,ftp-bounce 192.168.1.10
nmap -sS --script smb-enum-shares,smb-enum-users 192.168.1.10
```

### Catégories de Scripts
Les scripts NSE sont organisés en catégories :

| Catégorie | Objectif |
|-----------|----------|
| **default** | Scripts sûrs activés avec `-sC` |
| **discovery** | Découverte de services et hôtes |
| **vuln** | Détection de vulnérabilités |
| **auth** | Tests d'authentification |
| **brute** | Attaques par force brute |
| **http** | Énumération de serveurs web |
| **smb** | Énumération SMB |
| **ftp** | Énumération FTP |
| **ssh** | Énumération SSH |
| **smtp** | Énumération SMTP |

Exemple :
```bash
nmap -sS --script default 192.168.1.10
nmap -sS --script discovery 192.168.1.10
nmap -sS --script vuln 192.168.1.10
```

---

## Scripts NSE Courants par Service

### Énumération FTP
```bash
nmap -sS --script ftp-anon 192.168.1.10
nmap -sS --script ftp-bounce,ftp-libopie 192.168.1.10
```

### Énumération SMB
```bash
nmap -sS --script smb-enum-shares,smb-enum-users 192.168.1.10
nmap -sS --script smb-security-mode,smb-os-discovery 192.168.1.10
```

### Énumération SSH
```bash
nmap -sS --script ssh2-enum-algos,ssh-hostkey 192.168.1.10
```

### Énumération SMTP
```bash
nmap -sS --script smtp-enum-users,smtp-commands 192.168.1.10
```

### Énumération de Serveur Web
```bash
nmap -sS --script http-enum,http-headers 192.168.1.10
nmap -sS --script http-sql-injection,xss 192.168.1.10
```

### Énumération MySQL
```bash
nmap -sS --script mysql-enum,mysql-info 192.168.1.10
```

---

## Méthodologie Complète d'Énumération (IP → Services → Metasploit)

Cette méthodologie montre comment passer de l'**énumération brute d'IP** à un **workspace organisé Metasploit** avec des données de scan importées, tout cela uniquement pour du **footprinting et reconnaissance** (sans exploitation).

---

### Étape 1 : Scan Initial d'IP avec NSE

Commence par un scan de base pour découvrir les ports ouverts et services :

```bash
nmap -sS -sV -O -p- -sC -oX initial_scan.xml 192.168.1.10
```

Ce que cela fait :
- `-sS` : Scan TCP SYN (discrèt, par défaut)
- `-sV` : Détecter les versions de services
- `-O` : Tenter la détection OS
- `-p-` : Scanner les 65535 ports
- `-sC` : Exécuter les scripts NSE par défaut
- `-oX` : Exporter les résultats en XML pour import dans Metasploit

Cela te donne les ports, services, versions, empreinte OS et énumération de base depuis les scripts NSE.

---

### Étape 2 : Énumération NSE Avancée par Service

Basé sur ce que tu as trouvé, exécute des scripts NSE ciblés :

```bash
# Vérification d'accès anonyme FTP
nmap -sS --script ftp-anon -p 21 192.168.1.10

# Partages et utilisateurs SMB
nmap -sS --script smb-enum-shares,smb-enum-users -p 139,445 192.168.1.10

# Dossiers web et vulnérabilités
nmap -sS --script http-enum,http-vuln* -p 80,443 192.168.1.10

# Énumération d'utilisateurs SMTP
nmap -sS --script smtp-enum-users -p 25 192.168.1.10

# Algorithmes et hostkey SSH
nmap -sS --script ssh2-enum-algos -p 22 192.168.1.10
```

Sauvegarde chaque résultat pour référence ultérieure :
```bash
nmap -sS --script ftp-anon -p 21 -oN ftp_enum.txt 192.168.1.10
nmap -sS --script smb-enum-shares -p 445 -oN smb_enum.txt 192.168.1.10
```

---

### Étape 3 : Exporter Résultats en XML

Tu as déjà utilisé `-oX` à l'Étape 1, mais tu peux aussi exporter des scans supplémentaires :

```bash
nmap -sS --script smb-enum-shares -oX smb_scan.xml 192.168.1.10
nmap -sS --script http-enum -oX web_scan.xml 192.168.1.10
```

Le format XML est critique car **Metasploit peut l'importer** et stocker tous les hôtes, ports et données de services dans sa base de données.

---

### Étape 4 : Lancer Metasploit et Créer un Workspace

Ouvre la console Metasploit :
```bash
msfconsole
```

Crée un nouveau workspace pour cette cible (pour garder tout organisé) :
```bash
workspace -a enumeration_target_192.168.1.10
```

Options de workspace :
- `-a` : Ajouter un nouveau workspace
- `-w` : Changer vers workspace existant
- `-d` : Supprimer workspace
- `-l` : Lister workspaces

Cela assure que toutes tes données de scan, notes et modules futurs restent séparés des autres évaluations.

---

### Étape 5 : Importer XML Nmap dans la Base de Données Metasploit

Importe ton fichier XML de scan :
```bash
db_import initial_scan.xml
```

Cette commande analyse le XML et stocke **hôtes, ports, services et scripts** dans la base de données Metasploit automatiquement.

Tu peux importer plusieurs fichiers à la fois :
```bash
db_import *.xml
```

---

### Étape 6 : Lister Hôtes et Services

Maintenant vérifie ce qui a été importé :

```bash
# Lister tous les hôtes dans la base de données
hosts

# Lister tous les services dans la base de données
services

# Filtrer par port
services -p 22

# Filtrer par protocole
services -p 445 -p 3389
```

Tu peux aussi utiliser `db_nmap` pour exécuter Nmap directement depuis Metasploit avec stockage automatique en base de données :

```bash
db_nmap -sS -sV -oX db_scan.xml 192.168.1.10
db_nmap -sS -Pn -p 80,443 192.168.1.10
```

L'option `-Pn` est utile quand la cible bloque le ping mais a encore des ports ouverts.

---

### Étape 7 : Lister Vulnérabilités

Metasploit peut afficher les vulnérabilités détectées depuis les scans importés :

```bash
vulns
```

Cela listera toutes les vulnérabilités trouvées par les scripts NSE (comme les scripts de catégorie `vuln`) qui ont été stockées dans la base de données.

---

### Étape 8 : Utiliser les Modules Auxiliaires Metasploit pour Énumération

Metasploit a des **modules auxiliaires** conçus spécifiquement pour la **reconnaissance et énumération** (pas exploitation). Ils sont parfaits pour la phase de footprinting :

```bash
# Rechercher modules auxiliaires
search auxiliary

# Vérification login anonyme FTP
use auxiliary/scanner/ftp/anonymous
set RHOSTS 192.168.1.10
run

# Énumération SMB
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS 192.168.1.10
run

# Énumération d'utilisateurs SMB
use auxiliary/scanner/smb/smb_enumusers
set RHOSTS 192.168.1.10
run

# Vérification version SSH
use auxiliary/scanner/ssh/ssh_version
set RHOSTS 192.168.1.10
run

# Énumération d'utilisateurs SMTP
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS 192.168.1.10
run

# Énumération de dossiers HTTP
use auxiliary/scanner/http/dir_scanner
set RHOSTS 192.168.1.10
set THREADS 10
run
```

Catégories clés d'escanners auxiliaires :
- `auxiliary/scanner/ftp/`
- `auxiliary/scanner/smb/`
- `auxiliary/scanner/ssh/`
- `auxiliary/scanner/smtp/`
- `auxiliary/scanner/http/`
- `auxiliary/scanner/mysql/`

Ces modules effectuent une **énumération passive ou active** sans exploiter de vulnérabilités, te maintenant dans la **phase de footprinting/reconnaissance**.

---

### Étape 9 : Organiser et Documenter

Après l'énumération, documente tes découvertes :

```bash
# Exporter données du workspace
db_export -f xml workspace_export.xml

# Lister tous les hôtes découverts
hosts -d

# Lister tous les services avec détails
services -v
```

Sauvegarde tes résultats de scan, sortie NSE et exportations de base de données Metasploit pour ton rapport final.

---

## NSE vs Modules Auxiliaires Metasploit

| Caractéristique | Scripts NSE | Auxiliaires Metasploit |
|-----------------|-------------|-----------------------|
| **Emplacement** | Dans Nmap | Dans Metasploit |
| **Vitesse** | Très rapide (outil unique) | Plus lent (plus d'options) |
| **Intégration** | Export manuel nécessaire | Stockage automatique en base de données |
| **Scripting** | Basé sur Lua | Basé sur Ruby |
| **Meilleur pour** | Scans rapides, énumération rapide | Évaluations organisées, flux de travail avec base de données |
| **Cas d'usage** | Footprinting initial | Énumération détaillée, rapports |

---

## Points Clés

- **NSE** utilise des scripts Lua pour automatiser l'énumération de services, la détection de vulnérabilités et la collecte d'informations.
- `-sC` exécute les scripts par défaut ; `--script` exécute des scripts ou catégories spécifiques.
- Scripts courants : `ftp-anon`, `smb-enum-shares`, `http-enum`, `smtp-enum-users`, `ssh2-enum-algos`.
- Exporte les résultats de scan avec **`-oX`** en XML pour import dans Metasploit.
- Crée un **workspace Metasploit** pour garder les évaluations organisées.
- Utilise **`db_import`** pour charger le XML Nmap dans la base de données Metasploit.
- Liste les hôtes avec **`hosts`**, les services avec **`services`**, et les vulnérabilités avec **`vulns`**.
- Utilise **`db_nmap`** pour exécuter Nmap directement depuis Metasploit avec stockage automatique en base de données.
- Les **modules auxiliaires Metasploit** (`auxiliary/scanner/*`) effectuent l'énumération sans exploitation.
