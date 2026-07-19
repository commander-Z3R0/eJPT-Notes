# Networking

Le but des réseaux est d'échanger des informations entre les hôtes. Les paquets sont simplement des bits (signaux électriques) qui, traduits par un ordinateur, forment de l'information.

---

## Networking Fundamentals

Cette leçon couvre les concepts fondamentaux des réseaux nécessaires pour les tests d'intrusion basés sur le réseau — notamment le flux des données, le modèle OSI, les protocoles clés des couches Réseau et Transport, et les concepts essentiels d'adressage IP.

### Comment les Réseaux Communiquent

Dans un réseau informatique, les hôtes communiquent via des **protocoles réseau** — des règles standardisées garantissant que différents systèmes peuvent échanger des informations. Cette communication est transportée par des **paquets**.

Chaque paquet comporte deux parties :

| Partie | Description |
|--------|-------------|
| **En-tête** | Structure spécifique au protocole (TCP, UDP, IP) indiquant à l'hôte destination comment interpréter les données. Contient IPs source/destination, version, TTL, type de protocole. |
| **Payload** | Les données ou informations réelles transmises. |

---

### Le Modèle OSI

Le modèle **OSI (Open Systems Interconnection)** standardise la communication réseau en 7 couches. Ce n'est pas un plan strict, mais il aide à comprendre et concevoir l'architecture réseau.

| Couche | Nom | Fonction | Protocoles Clés |
|--------|-----|----------|----------------|
| 7 | **Application** | Services et interfaces orientés utilisateur | HTTP, FTP, SMTP, DNS |
| 6 | **Présentation** | Formatage des données, chiffrement, compression | SSL/TLS, JPEG |
| 5 | **Session** | Gère les sessions entre applications | NetBIOS, RPC |
| 4 | **Transport** | Communication de bout en bout, contrôle des erreurs | TCP, UDP |
| 3 | **Réseau** | Adressage logique et routage | IP, ICMP, DHCP |
| 2 | **Liaison de Données** | Adressage physique, livraison des trames | Ethernet, MAC, ARP |
| 1 | **Physique** | Transmission brute de bits sur le support | Câbles, Wi-Fi, fibre |

> Pour les tests d'intrusion, les **couches 3 (Réseau) et 4 (Transport)** sont les plus critiques.

---

### Couche 3 — La Couche Réseau

Responsable de l'**adressage logique**, du **routage** et du **transfert de paquets** entre appareils sur différents réseaux.

#### Protocoles Clés

| Protocole | Utilité |
|-----------|---------|
| **IPv4** | Version IP la plus utilisée — adresses 32 bits (ex. `192.168.1.1`) |
| **IPv6** | Successeur d'IPv4 — adresses 128 bits |
| **ICMP** | Protocole de Messages de Contrôle Internet — diagnostics et rapports d'erreurs |
| **DHCP** | Protocole de Configuration Dynamique des Hôtes — assigne les IPs automatiquement |

#### Fonctions Principales d'IP

| Fonction | Description |
|----------|-------------|
| **Adressage logique** | Les IPs identifient de manière unique chaque appareil |
| **Structure des paquets** | Données organisées en paquets avec en-tête et payload |
| **Fragmentation et réassemblage** | Grands paquets divisés pour s'adapter au MTU, réassemblés à destination |
| **Routage** | Paquets transférés saut par saut vers l'IP destination |

#### Types d'Adressage IP

| Type | Description |
|------|-------------|
| **Unicast** | Communication d'un à un |
| **Broadcast** | Communication d'un à tous dans un sous-réseau |
| **Multicast** | Communication d'un à plusieurs pour un groupe sélectionné |

#### Adresses IPv4 Réservées (RFC 5735)

| Plage | Utilité |
|-------|---------|
| `0.0.0.0 – 0.255.255.255` | Représente le réseau auquel vous êtes connecté |
| `127.0.0.0 – 127.255.255.255` | Loopback / localhost (votre propre machine) |
| `192.168.0.0 – 192.168.255.255` | Réseaux privés (LAN) |
| `10.0.0.0 – 10.255.255.255` | Réseaux privés (LAN) |
| `172.16.0.0 – 172.31.255.255` | Réseaux privés (LAN) |

#### Champs de l'En-tête de Paquet IPv4

| Champ | Taille | Description |
|-------|--------|-------------|
| **IP Source** | 32 bits | Adresse source du paquet |
| **IP Destination** | 32 bits | Adresse cible du paquet |
| **TTL** | 8 bits | Décrémenté à chaque saut — paquet supprimé quand TTL atteint 0 |
| **Protocole** | 8 bits | Protocole de transport (6 = TCP, 17 = UDP, 1 = ICMP) |
| **ToS** | 8 bits | Priorité du paquet pour la QoS |
| **Version** | 4 bits | IPv4 ou IPv6 |

#### ICMP — Protocole de Messages de Contrôle Internet

| Message ICMP | Type | Code | Utilisation |
|-------------|------|------|------------|
| **Echo Request** | 8 | 0 | Envoyé par `ping` pour tester l'accessibilité de l'hôte |
| **Echo Reply** | 0 | 0 | Réponse confirmant que l'hôte est actif |
| **Destination Inaccessible** | 3 | — | Hôte ou port inaccessible |
| **Temps Dépassé** | 11 | 0 | TTL expiré en transit (utilisé par `traceroute`) |

---

### Couche 4 — La Couche Transport

Fournit une **communication de bout en bout** entre les applications. Gère la détection des erreurs, le contrôle de flux et la segmentation des données.

#### TCP — Protocole de Contrôle de Transmission

**Orienté connexion** — établit une connexion fiable avant le transfert de données.

| Fonctionnalité | Description |
|---------------|-------------|
| **Fiabilité** | Données envoyées précisément et dans l'ordre — utilise ACK |
| **Retransmission** | Segments perdus automatiquement renvoyés |
| **Ordonnancement** | Segments hors ordre réordonnés avant la livraison |
| **Connexion** | Nécessite un handshake à 3 voies avant l'échange de données |

#### Le Handshake TCP à 3 Voies

```
Client  →  SYN      →  Serveur    (Je veux me connecter)
Client  ←  SYN-ACK  ←  Serveur    (Confirmé, j'accepte)
Client  →  ACK      →  Serveur    (Connexion établie)
```

> Les **scans SYN** (half-open) envoient un SYN mais ne complètent jamais le handshake — plus rapides et évitent la journalisation sur la cible.

#### Flags de Contrôle TCP

| Flag | Description |
|------|-------------|
| **SYN** | Initie une demande de connexion |
| **ACK** | Accuse réception des données |
| **FIN** | Initie la terminaison ordonnée de la connexion |
| **RST** | Réinitialise/termine abruptement une connexion |
| **PSH** | Envoie immédiatement les données à l'application |
| **URG** | Marque les données comme urgentes |

#### Plages de Ports TCP

| Plage | Type | Description |
|-------|------|-------------|
| **0 – 1023** | Bien connus | Réservés par l'IANA pour les services standards |
| **1024 – 49151** | Enregistrés | Attribués par l'IANA pour les applications/éditeurs |
| **49152 – 65535** | Dynamiques/éphémères | Ports temporaires utilisés par les clients |

#### Ports Bien Connus Courants

| Port | Protocole | Service |
|------|-----------|---------|
| 21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB |
| 3306 | TCP | MySQL |
| 3389 | TCP | RDP |
| 8080 | TCP | HTTP Alternatif |

#### UDP — Protocole de Datagrammes Utilisateur

**Sans connexion** — envoie des données sans établir de connexion au préalable.

| Fonctionnalité | Description |
|---------------|-------------|
| **Vitesse** | Plus rapide que TCP — pas de surcharge de handshake |
| **Fiabilité** | Aucune garantie de livraison, d'ordonnancement ni de correction d'erreurs |
| **Indépendance** | Chaque datagramme traité indépendamment |
| **Cas d'utilisation** | DNS, DHCP, SNMP, VoIP, streaming, jeux en ligne |

#### TCP vs UDP

| Fonctionnalité | TCP | UDP |
|---------------|-----|-----|
| **Connexion** | Orienté connexion (handshake 3 voies) | Sans connexion |
| **Fiabilité** | Livraison garantie avec ACK | Aucune garantie |
| **Vitesse** | Plus lent | Plus rapide |
| **Ordonnancement** | Données dans l'ordre | Pas d'ordonnancement |
| **Correction d'erreurs** | Oui (retransmission) | Non |
| **Cas d'utilisation** | HTTP, HTTPS, SSH, FTP, SMB | DNS, VoIP, streaming |

---

### Sous-réseaux et CIDR

Le **sous-réseautage** divise un grand réseau IP en sous-réseaux plus petits — améliore l'efficacité et la sécurité en isolant le trafic.

**CIDR (Classless Inter-Domain Routing)** exprime une adresse réseau avec sa longueur de préfixe :

| CIDR | Masque de Sous-réseau | Hôtes Disponibles |
|------|----------------------|------------------|
| `/8` | 255.0.0.0 | 16 777 214 |
| `/16` | 255.255.0.0 | 65 534 |
| `/24` | 255.255.255.0 | 254 |
| `/28` | 255.255.255.240 | 14 |
| `/30` | 255.255.255.252 | 2 |
| `/32` | 255.255.255.255 | 1 (hôte unique) |

**Exemple** : Un périmètre de `200.200.0.0/16` = jusqu'à 65 536 adresses IP. Lors d'un pentest, votre premier travail est de déterminer lesquelles sont actives.

---

### Points Clés — Networking Fundamentals

- Les réseaux communiquent via des **paquets** — en-tête (infos de routage) + payload (données).
- Le **modèle OSI** a 7 couches — les couches 3 et 4 sont les plus pertinentes pour le pentesting.
- **Couche 3 (IP)** : adressage logique, routage — protocoles clés : IPv4, IPv6, ICMP, DHCP.
- Le **TTL** est décrémenté à chaque saut — atteint 0 et le paquet est supprimé. Utilisé par `traceroute`.
- **ICMP** : Echo Request (type 8) / Echo Reply (type 0) = `ping` ; type 11 = TTL expiré = `traceroute`.
- **TCP** : nécessite un handshake à 3 voies — fiable, ordonné, plus lent. Les scans SYN exploitent le handshake.
- **UDP** : sans connexion — plus rapide, non fiable. Utilisé par DNS, DHCP, SNMP.
- **Flags TCP** : SYN, ACK, FIN, RST — chacun joue un rôle dans les types de scan.
- Ports **0–1023** = bien connus ; **1024–49151** = enregistrés ; **49152–65535** = éphémères.
- Un périmètre `/16` = jusqu'à **65 534 hôtes** — commencez toujours par la découverte d'hôtes avant le scan de ports.

---

## Firewall Detection & IDS Evasion

Cette leçon couvre comment détecter les firewalls et systèmes IDS/IPS lors d'un scan, et les techniques Nmap pour contourner le filtrage de base.

> ⚠️ **Note** : Toutes les commandes supposent que vous opérez depuis une machine d'attaque **Kali Linux**. Remplacez `<target_ip>` par votre IP cible réelle.

---

### Que sont les Firewalls et les IDS/IPS ?

| Système | Nom Complet | Utilité |
|---------|------------|---------|
| **Firewall** | — | Filtre le trafic selon des règles — bloque ou autorise les paquets par port, IP ou protocole |
| **IDS** | Système de Détection d'Intrusions | Surveille le trafic et **alerte** sur les activités suspectes — ne bloque pas |
| **IPS** | Système de Prévention d'Intrusions | Surveille le trafic et **bloque activement** les activités suspectes |

---

### Détection de Firewalls

#### Méthode 1 : Analyse de l'État des Ports

| État du Port | Signification | Firewall Présent ? |
|-------------|--------------|-------------------|
| **Open** | Le service écoute et répond | Non (ou port autorisé) |
| **Closed** | L'hôte répond par RST — port non utilisé | Pas de firewall sur ce port |
| **Filtered** | Pas de réponse ou ICMP inaccessible | **Oui — un firewall bloque** |

```bash
nmap -sS -p 80,443,22,3389 <target_ip>
```

#### Méthode 2 : Scan ACK (`-sA`)

Conçu pour **cartographier les règles de firewall** — ne détecte pas les ports ouverts, détecte s'ils sont filtrés.

```bash
nmap -sA -p 80,443,22 <target_ip>
```

| Réponse | Interprétation |
|---------|---------------|
| **RST reçu** | Port non filtré — le firewall laisse passer les paquets ACK |
| **Pas de réponse / ICMP inaccessible** | Port filtré — le firewall bloque |

> Le scan ACK distingue les firewalls **stateful** (rejette les ACK inattendus) des **stateless** (peut les laisser passer).

#### Méthode 3 : Scan de Fenêtre (`-sW`)

Analyse la **taille de la fenêtre TCP** dans les réponses RST pour différencier les ports ouverts des fermés.

```bash
nmap -sW -p 80,443 <target_ip>
```

#### Méthode 4 : Test Rapide

```bash
nmap -sS -p 80 <target_ip>   # État du port avec scan SYN
nmap -sA -p 80 <target_ip>   # Filtré/non filtré avec scan ACK
```

SYN = **filtered** + ACK = **unfiltered** → firewall stateful rejetant les SYN mais pas les ACK.

---

### Techniques d'Évasion IDS/IPS

#### 1. Fragmentation de Paquets (`-f`, `--mtu`)

Divise les sondes en fragments IP plus petits — les anciens IDS/firewalls peuvent échouer à les inspecter.

```bash
nmap -sS -f <target_ip>           # Fragments de 8 octets
nmap -sS -ff <target_ip>          # Fragments de 16 octets
nmap -sS --mtu 24 <target_ip>     # Taille personnalisée (multiple de 8)
```

#### 2. Scan avec Leurres (`-D`)

Mélange de fausses IPs sources avec le scan réel — la cible voit du trafic provenant de nombreuses IPs.

```bash
nmap -sS -D 10.10.10.5,10.10.10.6,ME <target_ip>   # Leurres manuels
nmap -sS -D RND:5 <target_ip>                        # 5 leurres aléatoires
```

#### 3. Falsification du Port Source (`-g` / `--source-port`)

Certains firewalls autorisent le trafic de ports de confiance (DNS 53, HTTP 80) sans inspection approfondie.

```bash
nmap -sS -g 53 <target_ip>     # Sembler venir du port DNS
nmap -sS -g 80 <target_ip>     # Sembler venir du port HTTP
```

#### 4. Longueur de Données Aléatoire (`--data-length`)

Ajoute des octets aléatoires aux paquets — plus difficile pour les IDS à signatures de les détecter.

```bash
nmap -sS --data-length 25 <target_ip>
```

#### 5. Désactiver la Résolution DNS (`-n`)

Évite les requêtes DNS — chaque requête est un point de détection potentiel.

```bash
nmap -sS -n <target_ip>
```

#### 6. Falsification d'IP Source (`-S`)

Le trafic semble provenir d'un autre hôte. Les réponses vont à l'IP falsifiée — principalement pour **tester les règles d'alerte IDS**.

```bash
nmap -sS -S 192.168.1.50 -e eth0 <target_ip>
```

#### 7. Falsification d'Adresse MAC (`--spoof-mac`)

Falsifie la MAC source — utile sur les LANs avec filtrage basé sur MAC.

```bash
nmap -sS --spoof-mac 0 <target_ip>            # MAC aléatoire
nmap -sS --spoof-mac Apple <target_ip>        # MAC d'un fabricant
nmap -sS --spoof-mac 00:11:22:33:44:55 <target_ip>  # MAC spécifique
```

---

### Évasion Basée sur le Temps

#### Modèles de Temporisation (`-T`)

| Modèle | Nom | Vitesse | Discrétion | Cas d'Utilisation |
|--------|-----|---------|-----------|------------------|
| `-T0` | Paranoid | Extrêmement lent | Maximum | Contourner la plupart des IDS |
| `-T1` | Sneaky | Très lent | Élevée | Scans furtifs |
| `-T2` | Polite | Lent | Moyenne | Réduire la charge réseau |
| `-T3` | Normal | Par défaut | Par défaut | Point de départ équilibré |
| `-T4` | Aggressive | Rapide | Faible | Réseaux internes fiables |
| `-T5` | Insane | Très rapide | Aucune | ⚠️ Risque élevé de perte de paquets |

```bash
nmap -sS -T1 <target_ip>    # Scan furtif lent
nmap -sS -T3 <target_ip>    # Scan standard
nmap -sS -T4 <target_ip>    # Scan rapide interne
```

#### Contrôles de Temporisation Manuels

```bash
nmap -sS --scan-delay 200ms <target_ip>   # Délai entre les sondes
nmap --min-rate 50 <target_ip>             # Minimum 50 paquets/sec
nmap --host-timeout 5m <target_ip>         # Abandonner après 5 min par hôte
```

---

### Formats de Sortie

Sauvegardez toujours les résultats pour l'analyse, les rapports et l'import dans Metasploit.

| Flag | Format | Utilisation |
|------|--------|------------|
| `-oN` | Texte normal | Notes lisibles par l'humain |
| `-oX` | XML | Import Metasploit, intégration d'outils |
| `-oG` | Grepable | Analyse shell et scripts |
| `-oA` | Tous les formats | Sauvegarde les trois à la fois |

```bash
nmap -sS -oN scan.txt <target_ip>       # Texte
nmap -sS -oX scan.xml <target_ip>       # XML
nmap -sS -oA scan_results <target_ip>   # Tous les formats
```

#### Importer le XML dans Metasploit

```bash
nmap -sS -sV -oX scan.xml <target_ip>
msfconsole
msf6 > db_import scan.xml
msf6 > hosts
msf6 > services
```

---

### Flux de Travail d'Évasion Combiné

```bash
# Fragmentation + leurres + port source + sans DNS + temporisation lente + sortie XML
nmap -sS -Pn -f -D RND:5 -g 53 -n -T2 --data-length 15 -oX evasion_scan.xml <target_ip>
```

| Flag | Utilité |
|------|---------|
| `-sS` | Scan SYN furtif |
| `-Pn` | Ignorer la découverte d'hôtes |
| `-f` | Fragmenter les paquets |
| `-D RND:5` | 5 IPs leurres aléatoires |
| `-g 53` | Port source 53 (DNS) |
| `-n` | Sans résolution DNS |
| `-T2` | Temporisation polie |
| `--data-length 15` | 15 octets aléatoires ajoutés |
| `-oX` | Sauvegarder en XML |

---

### Résumé des Techniques d'Évasion

| Technique | Flag | Contourne |
|-----------|------|----------|
| Fragmentation de paquets | `-f`, `--mtu` | Filtres simples, anciens IDS |
| Scan avec leurres | `-D RND:n` | Détection basée sur IP |
| Falsification port source | `-g <port>` | Règles firewall basées sur les ports |
| Longueur de données aléatoire | `--data-length` | IDS à signatures |
| Désactiver DNS | `-n` | Détection basée sur DNS |
| Falsification IP source | `-S` | Test de règles d'alerte IDS |
| Falsification MAC | `--spoof-mac` | Filtrage basé sur MAC (LAN) |
| Temporisation lente | `-T0`, `-T1`, `--scan-delay` | Seuils IDS basés sur le débit |
| Ignorer découverte d'hôtes | `-Pn` | Firewalls bloquant ICMP |

---

### Points Clés — Firewall Detection & IDS Evasion

- Les **firewalls** filtrent par port/IP/protocole — détectés par les états **filtered** dans la sortie Nmap.
- Les **IDS** alertent ; les **IPS** bloquent — les deux analysent les patterns et débits de paquets.
- **Scan ACK** (`-sA`) : RST = non filtré, pas de réponse = filtré — meilleur outil pour cartographier les règles firewall.
- Les firewalls **stateful** rejettent les ACK inattendus ; les **stateless** peuvent les laisser passer.
- La **fragmentation** (`-f`) = morceaux de 8 octets — efficace contre les anciens systèmes, pas les firewalls stateful modernes.
- Les **leurres** (`-D RND:5`) cachent le vrai scanner parmi des IPs sources aléatoires.
- La **falsification du port source** (`-g 53`) contourne les règles firewall qui font confiance au port 53 ou 80.
- La **temporisation lente** (`-T0`, `-T1`, `--scan-delay`) contourne les IDS basés sur le débit.
- **`-Pn`** ignore la découverte d'hôtes ICMP — à utiliser quand la cible bloque les pings.
- **N'utilisez jamais `-T5`** en pentest réel — provoque des pertes de paquets et des résultats inexacts.
- Meilleure combinaison d'évasion : `-f -D RND:5 -g 53 -n -T2 --data-length 15`.
- Sauvegardez toujours avec **`-oX`** (XML) pour importer dans Metasploit via `db_import`.
