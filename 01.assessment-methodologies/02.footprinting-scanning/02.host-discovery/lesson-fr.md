# Host Discovery

Host discovery est le processus d'identification des hôtes **en ligne/actifs** dans un réseau. Au lieu de scanner tous les ports de toutes les IPs, nous trouvons d'abord les machines actives pour concentrer nos efforts.

## Qu'est-ce que Host Discovery?

En pentesting et reconnaissance réseau, host discovery (aussi appelé **ping scan**) réduit une grande plage d'IPs à une liste plus petite d'hôtes actifs. Nmap supporte de nombreuses techniques de découverte au-delà du simple ping ICMP:

- Certains hôtes bloquent les demandes ICMP echo (ping classique).
- Les pare-feux peuvent filtrer certains types de trafic.
- Différentes techniques fonctionnent mieux dans différents environnements réseau.

---

## Techniques de Host Discovery

### 1. Ping Sweeps (Ping ICMP Echo)

Un **ping sweep** envoie des paquets ICMP echo request (type 8) à plusieurs hôtes et attend des réponses ICMP echo reply (type 0). Si un hôte répond, il est actif.

- **Option Nmap**: `-PE`
- **Comment ça marche**:
  - Envoyer ICMP Echo Request → si Echo Reply → hôte est actif.
- **Avantages**:
  - Simple, largement compris.
  - Rapide sur les réseaux internes.
- **Inconvénients**:
  - Beaucoup d'hôtes et pare-feux **bloquent ICMP echo** maintenant.
  - Pas fiable sur Internet contre des cibles inconnues.

Exemple:
```bash
nmap -sn -PE <target_network>/24
```

---

### 2. ARP Scanning (ARP Ping)

**ARP scanning** est utilisé sur les **réseaux Ethernet locaux** (LANs). Il envoie des requêtes ARP demandant "Qui a l'IP X?" et attend des réponses ARP.

- **Option Nmap**: `-PR` (par défaut sur Ethernet local)
- **Comment ça marche**:
  - Envoyer requête ARP: "Qui a 192.168.1.5?"
  - Si l'hôte répond avec ARP response → hôte est actif.
- **Avantages**:
  - Très **rapide** et **précis** sur les LANs.
  - Les hôtes ne peuvent généralement **pas bloquer ARP** et communiquer encore sur le réseau.
- **Inconvénients**:
  - Ne fonctionne que sur les **réseaux locaux** (même sous-réseau).
  - Ne peut pas être utilisé pour le scan distant à travers les routeurs.

Exemple:
```bash
nmap -sn -PR <target_network>/24
```

---

### 3. TCP SYN Ping (Half-Open Scan)

**TCP SYN ping** envoie un paquet TCP avec le drapeau **SYN** défini (comme la première étape du handshake TCP), mais ne complète jamais la connexion. C'est pourquoi c'est appelé un **half-open scan**.

- **Option Nmap**: `-PS<port>` (ex. `-PS80`, `-PS22,80,443`)
- **Comment ça marche**:
  1. Envoyer paquet TCP SYN à un port cible.
  2. Si le port est **fermé**: l'hôte répond avec **RST** → hôte est actif.
  3. Si le port est **ouvert**: l'hôte répond avec **SYN/ACK** → hôte est actif.
  4. Nmap envoie ensuite **RST** pour couper la connexion (n'envoie jamais ACK).

- **Avantages**:
  - Souvent **bypasse les pare-feux** qui bloquent ICMP.
  - Fonctionne bien quand on cible des services courants (HTTP port 80, HTTPS 443, SSH 22).
- **Inconvénients**:
  - Requiert la capacité de **paquets bruts** (normalement root sur Unix).
  - Les utilisateurs non-privilégiés utilisent le workaround `connect()` (moins efficace).

Exemple:
```bash
nmap -sn -PS80,443 <target_network>/24
```

Cette technique s'appelle **half-open** car nous ne complétons jamais le handshake TCP à 3 voies.

---

### 4. UDP Ping

**UDP ping** envoie des paquets UDP à des ports spécifiques. Si un port est **fermé**, la cible répond typiquement avec un message **ICMP port unreachable**, ce qui dit à Nmap que l'hôte est actif.

- **Option Nmap**: `-PU<port>` (ex. `-PU53`, `-PU161`)
- Port par défaut: **40125** (port peu courant).
- **Comment ça marche**:
  1. Envoyer paquet UDP à un port.
  2. Si le port est **fermé**: recevoir **ICMP port unreachable** → hôte est actif.
  3. Si le port est **ouvert**: la plupart des services ignorent le paquet → pas de réponse.
  4. Manque de réponse ou certaines erreurs ICMP → l'hôte peut être hors ligne/inaccessible.

- **Avantages**:
  - Bypasse les pare-feux qui **filtrent TCP mais autorisent UDP**.
  - Utile quand les probes TCP sont bloqués.
- **Inconvénients**:
  - Plus lent que TCP (UDP est connectionless, beaucoup de timeouts).
  - Moins fiable car beaucoup d'hôtes n'envoient pas d'erreurs ICMP.

Exemple:
```bash
nmap -sn -PU53,161,40125 <target_network>/24
```

---

### 5. TCP ACK Ping

**TCP ACK ping** envoie un paquet TCP avec le drapeau **ACK** défini. Ce paquet ressemble à une acquittement de données sur une **connexion existante**, mais aucune telle connexion n'existe.

- **Option Nmap**: `-PA<port>` (ex. `-PA80`, `-PA443`)
- **Comment ça marche**:
  1. Envoyer paquet TCP ACK à un port.
  2. L'hôte devrait répondre avec **RST** (car aucune connexion n'existe) → hôte est actif.
- **Avantages**:
  - Peut **bypasser les pare-feux stateless** qui bloquent SYN entrant mais autorisent ACK.
  - Utile quand SYN ping est filtré.
- **Inconvénients**:
  - Moins efficace contre les **pare-feux stateful** qui rejettent les paquets ACK inattendus.
  - Comme SYN ping, nécessite normalement root sur Unix.

Exemple:
```bash
nmap -sn -PA80,443 <target_network>/24
```

---

## Comparaison des Techniques de Host Discovery

| Technique            | Type de Paquet      | Option Nmap | Fonctionne Mieux sur          | Bypasse Blocage ICMP? |
|----------------------|---------------------|-------------|-------------------------------|-----------------------|
| Ping Sweep (ICMP)    | ICMP Echo Request   | `-PE`       | Réseaux internes              | Non                   |
| ARP Scan             | ARP Request         | `-PR`       | Ethernet local (LAN)          | N/A (pas d'ICMP)      |
| TCP SYN Ping         | TCP SYN             | `-PS`       | Internet, services courants   | Oui                   |
| TCP ACK Ping         | TCP ACK             | `-PA`       | Pare-feux stateless           | Oui                   |
| UDP Ping             | UDP                 | `-PU`       | Pare-feux autorisant UDP      | Oui                   |

---

## Host Discovery par Défaut dans Nmap

Si vous ne spécifiez aucun type de ping, Nmap utilise une **combinaison par défaut**:

- Pour les **utilisateurs privilégiés** (root):
  - `-PE -PS443 -PA80 -PP`
  - ICMP echo + TCP SYN au 443 + TCP ACK au 80 + ICMP timestamp.
- Pour **Ethernet local**:
  - Utilise **ARP scan** par défaut (`-PR`), même si vous spécifiez d'autres options.
- Pour les **utilisateurs non-privilégiés**:
  - `-PS80,443` utilisant l'appel système `connect()`.

Exemple de host discovery seulement (sans scan de ports):
```bash
nmap -sn <target_network>/24
```

L'option `-sn` dit à Nmap: **seulement faire host discovery, pas de scan de ports**.

---

## Points Clés

- **Ping sweeps** (ICMP) sont simples mais souvent bloqués.
- **ARP scanning** est le plus fiable sur les LANs locaux.
- **TCP SYN ping** est un half-open scan; excellent pour bypasser les filtres ICMP.
- **UDP ping** est utile quand TCP est filtré mais UDP autorisé.
- **TCP ACK ping** peut bypasser les pare-feux stateless qui bloquent SYN.
- En pratique, utilisez **plusieurs techniques** ensemble pour maximiser la précision de host discovery.
