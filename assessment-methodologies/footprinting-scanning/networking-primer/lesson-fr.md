# Networking Primer

## 1. Modèle OSI (7 Couches)

Le modèle OSI (Open Systems Interconnection) est un cadre conceptuel qui décrit comment les données circulent dans un réseau. Il comporte **7 couches**:

| Couche | Nom            | PDU       | Fonction principale                                      | Exemples de protocoles/dispositifs        |
|--------|----------------|-----------|----------------------------------------------------------|-------------------------------------------|
| 7      | Application    | Données   | Services utilisateur, applications réseau                | HTTP, FTP, DNS, SMTP                      |
| 6      | Présentation   | Données   | Mise en forme, chiffrement, compression des données      | SSL/TLS, JPEG, ASCII                      |
| 5      | Session        | Données   | Gère les sessions entre applications                     | NetBIOS, RPC                              |
| 4      | Transport      | Segment   | Communication de bout en bout, fiabilité, contrôle       | TCP, UDP                                  |
| 3      | Réseau         | Paquet    | Routage, adressage logique (IP)                          | IP, ICMP, routeurs                        |
| 2      | Liaison de données| Trame   | Adressage MAC, détection d'erreurs, commutateurs        | Ethernet, commutateurs, MAC               |
| 1      | Physique       | Bits      | Transmission physique (câbles, signaux)                  | Câbles, hubs, cartes réseau, radio        |

Les données descendent les couches chez l'expéditeur (7 → 1) et remontent chez le récepteur (1 → 7).

---

## 2. IP (Protocole Internet)

IP est un protocole de la **couche réseau (couche 3)** responsable du routage des paquets de la source vers la destination à l'aide d'adresses IP.

### Structure du paquet (général)

Un paquet IP est composé de:

- **En-tête IP** – informations de contrôle (IP source/destination, TTL, protocole, etc.)
- **Charge utile (payload)** – les données transportées par le paquet IP (par exemple, segment TCP, datagramme UDP)

```text
+---------------------+----------------------+
|      En-tête IP     |      Charge utile    |
| (20–60 octets)      | (TCP/UDP + données)  |
+---------------------+----------------------+
```

La charge utile contient généralement un **segment de la couche transport** (TCP ou UDP) plus les données de l'application.

---

## 3. TCP vs UDP

### TCP (Protocole de Contrôle de Transmission)

- **Couche**: Transport (couche 4)
- **Orienté connexion**: établit une connexion avant d'envoyer des données
- **Fiable**: garantit la livraison, l'ordre et la retransmission
- **Contrôle de flux et de congestion**
- Utilisé par: HTTP, HTTPS, SSH, FTP, SMTP

### UDP (Protocole de Datagrammes Utilisateur)

- **Couche**: Transport (couche 4)
- **Non orienté connexion**: pas de handshake, envoie simplement des paquets
- **Non fiable**: pas de garantie de livraison ni d'ordre
- **Plus rapide**, moins de surcharge
- Utilisé par: DNS, DHCP, VoIP, streaming, SNMP

---

## 4. En-tête IPv4 en détail

Un datagramme IPv4 possède un **en-tête de longueur variable** (minimum 20 octets, jusqu'à 60 octets avec options).

### Champs de l'en-tête IPv4

| Champ                  | Taille (bits) | Description                                                                 |
|------------------------|---------------|-----------------------------------------------------------------------------|
| Version                | 4             | Version IP; pour IPv4 c'est `4`                                           |
| IHL (Longueur En-tête) | 4             | Longueur de l'en-tête en mots de 32 bits; min 5 (20 octets), max 15 (60)  |
| Type de Service        | 8             | QoS: priorité, délai, débit, fiabilité                                    |
| Longueur Totale        | 16            | Longueur totale de l'en-tête + données (min 20, max 65 535 octets)        |
| Identification         | 16            | ID unique pour fragmentation/réassemblage                                 |
| Flags                  | 3             | Réservé (0), Ne pas fragmenter (DF), Plus de fragments (MF)              |
| Décalage de Fragment   | 13            | Décalage de ce fragment en unités de 8 octets                            |
| TTL (Time to Live)     | 8             | Nombre maximal de sauts avant que le paquet soit rejeté                   |
| Protocole              | 8             | Protocole suivant: `1=ICMP`, `6=TCP`, `17=UDP`                            |
| Somme de contrôle      | 16            | Somme de contrôle de l'en-tête uniquement (pas de la charge utile)        |
| IP Source              | 32            | Adresse IP de l'expéditeur                                                |
| IP Destination         | 32            | Adresse IP du récepteur                                                   |
| Options (optionnel)    | variable      | Fonctionnalités supplémentaires (ex. routage source) + remplissage        |

### Structure du paquet IPv4

```text
+------------------+-------------------+
|   En-tête IPv4   |    Charge utile   |
| (20–60 octets)   | (TCP/UDP + données)|
+------------------+-------------------+
```

Le champ **Protocole** indique au récepteur ce qu'il y a dans la charge utile:
- `1` → ICMP
- `6` → TCP
- `17` → UDP

---

## 5. TCP 3-Way Handshake (Poignée de main à 3 voies)

TCP est **orienté connexion**. Avant le transfert de données, une connexion fiable est établie via un **handshake à 3 voies**:

### Étapes

1. **SYN**  
   - Le client envoie un segment avec `SYN=1`, `ACK=0`.
   - Choisit un **Numéro de Séquence Initial (ISN)**, par ex. `Seq = 1001`.

2. **SYN-ACK**  
   - Le serveur répond avec `SYN=1`, `ACK=1`.
   - Le serveur choisit son propre ISN, par ex. `Seq = 2001`.
   - Accuse réception de l'ISN du client: `Ack = 1002` (ISN client + 1).

3. **ACK**  
   - Le client envoie `ACK=1`, `SYN=0`.
   - `Seq = 1002` (ISN client + 1).
   - `Ack = 2002` (ISN serveur + 1).

Après cela, la connexion est **établie** et le transfert de données commence.

```text
Client                      Serveur
  | SYN (Seq=1001)            |
  |-------------------------> |
  |                           |
  |<------------------------- |
  | SYN-ACK (Seq=2001, Ack=1002)
  |                           |
  | SYN-ACK (Seq=1002, Ack=2002)
  |-------------------------> |
  |                           |
  |   Connexion établie       |
```

---

## 6. Ports TCP et Plages

TCP utilise des **numéros de port sur 16 bits** (0–65535) pour identifier les applications/services.

### Plages de ports

| Plage              | Nom                  | Description                              |
|--------------------|----------------------|------------------------------------------|
| 0–1023             | **Bien connus**      | Services système/privilégiés (root)       |
| 1024–49151         | **Enregistrés**      | Applications utilisateur, services courants|
| 49152–65535        | **Dynamiques/Privés**| Ports éphémères pour connexions client   |

### Ports bien connus courants

| Port | Protocole | Service        |
|------|-----------|----------------|
| 20   | TCP       | FTP (données)  |
| 21   | TCP       | FTP (contrôle) |
| 22   | TCP       | SSH            |
| 23   | TCP       | Telnet         |
| 25   | TCP       | SMTP           |
| 53   | TCP/UDP   | DNS            |
| 80   | TCP       | HTTP           |
| 443  | TCP       | HTTPS          |
| 3389 | TCP       | RDP            |

Les clients utilisent généralement des **ports éphémères** (plage dynamique) comme ports source lors de la connexion aux serveurs.

---

## 7. Résumé rapide

- **Modèle OSI**: 7 couches de Physique (1) à Application (7).
- **IP (couche 3)**: route les paquets en utilisant IP source/destination.
- **En-tête IPv4**: contient version, longueur, TTL, protocole, checksum, adresses IP, etc.
- **TCP**: fiable, orienté connexion, utilise un handshake à 3 voies.
- **UDP**: rapide, non orienté connexion, sans garantie de livraison.
- **Ports**: identifient les services; bien connus (0–1023), enregistrés (1024–49151), dynamiques (49152–65535).
