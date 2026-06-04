# Port Scanning

Port scanning est le processus d'identification des **ports ouverts** sur un hôte cible. Une fois que vous savez qu'un hôte est actif (du host discovery), port scanning vous dit **quels services tournent** et potentiellement vulnérables.

## Qu'est-ce que Port Scanning?

En pentesting et reconnaissance, port scanning vous aide à:

- Découvrir les **ports ouverts** (services acceptant les connexions).
- Identifier les **services en cours d'exécution** (HTTP, SSH, SMTP, SMB, etc.).
- Déterminir les **versions de services** (pour l'exploitation).
- Mapper la **surface d'attaque** de la cible.

Nmap supporte de nombreuses techniques de scan, chacune avec différents avantages selon le réseau et le pare-feu:

- Certains scans sont **furtifs** (plus difficiles à détecter).
- Certains scans fonctionnent mieux **derrière les pare-feux**.
- Certains nécessitent des **privilèges root**, d'autres non.

---

## Techniques de Port Scanning

### 1. TCP SYN Scan (Stealth Scan)

**TCP SYN scan** est le scan **par défaut et le plus populaire** dans Nmap. Il envoie des paquets TCP SYN et analyse les réponses sans compléter le handshake TCP. C'est pourquoi c'est appelé un **half-open scan**.

- **Option Nmap**: `-sS`
- **Requiert**: privilèges root/administrateur (accès paquets bruts)
- **Comment ça marche**:
  1. Envoyer paquet TCP SYN au port cible.
  2. Si port **ouvert**: recevoir **SYN/ACK** → port ouvert.
  3. Si port **fermé**: recevoir **RST** → port fermé.
  4. Si **filtré**: pas de réponse ou erreur ICMP → port filtré.
  5. Nmap envoie **RST** pour couper (n'envoie jamais ACK).

- **Avantages**:
  - **Rapide** et **furtif** (ne complète pas les connexions).
  - Fonctionne dans la plupart des environnements.
  - Standard pour le pentesting.
- **Inconvénients**:
  - Requiert **root** sur Unix/Linux.
  - Peut encore être enregistré par IDS/IPS.

Exemple:
```bash
nmap -sS 192.168.1.5
nmap -sS -p 22,80,443 192.168.1.5
nmap -sS -p- 192.168.1.5        # scanner tous les 65535 ports
```

---

### 2. TCP Connect Scan

**TCP connect scan** utilise la fonction **connect()** du système pour compléter le handshake TCP complet. Ceci est utilisé quand vous **n'avez pas de privilèges root**.

- **Option Nmap**: `-sT`
- **Requiert**: pas de privilèges root (niveau utilisateur)
- **Comment ça marche**:
  1. Envoyer TCP SYN.
  2. Si port **ouvert**: recevoir SYN/ACK → envoyer ACK → connexion établie.
  3. Si port **fermé**: recevoir RST.
  4. Nmap ferme la connexion immédiatement après.

- **Avantages**:
  - Fonctionne **sans privilèges root**.
  - Fiable et bien supporté.
- **Inconvénients**:
  - **Plus bruyant** (complète connexions complètes, plus facile à enregistrer).
  - **Plus lent** que SYN scan.

Exemple:
```bash
nmap -sT 192.168.1.5
nmap -sT -p 80,443 192.168.1.5
```

---

### 3. UDP Scan

**UDP scan** sonde les ports UDP pour trouver des services UDP ouverts. UDP est **sans connexion**, donc le scan est plus lent et moins fiable que TCP.

- **Option Nmap**: `-sU`
- **Requiert**: privilèges root (habituellement)
- **Comment ça marche**:
  1. Envoyer paquet UDP vide (ou spécifique au protocole) au port.
  2. Si recevoir **ICMP port unreachable** → port **fermé**.
  3. Si recevoir **réponse UDP** → port **ouvert**.
  4. Si **pas de réponse** → port **open|filtered** (incertain).

- **Avantages**:
  - Découvre les **services UDP** (DNS, DHCP, SNMP, NTP, etc.).
  - Important pour la reconnaissance complète.
- **Inconvénients**:
  - **Très lent** (beaucoup de timeouts).
  - Moins fiable (beaucoup d'hôtes n'envoient pas d'erreurs ICMP).
  - Souvent filtré par les pare-feux.

Exemple:
```bash
nmap -sU 192.168.1.5
nmap -sU -p 53,161,123 192.168.1.5      # DNS, SNMP, NTP
nmap -sU --top-ports 100 192.168.1.5    # top 100 ports UDP
```

---

### 4. TCP ACK Scan

**TCP ACK scan** envoie des paquets TCP avec le drapeau **ACK** défini. Il est principalement utilisé pour **mapper les règles de pare-feu**, pas pour trouver des ports ouverts [web:50].

- **Option Nmap**: `-sA`
- **Requiert**: privilèges root
- **Comment ça marche**:
  1. Envoyer paquet TCP ACK au port.
  2. Si recevoir **RST** → port **unfiltered**.
  3. Si **pas de réponse** ou erreur ICMP → port **filtré**.

- **Avantages**:
  - Bon pour **mapper les pare-feux**.
  - Bypasse certains **pare-feux stateless**.
- **Inconvénients**:
  - **Ne peut pas déterminer ouvert vs fermé** (seulement unfiltered vs filtered).
  - Pas utile pour trouver des services.

Exemple:
```bash
nmap -sA 192.168.1.5
nmap -sA -p 80,443 192.168.1.5
```

---

### 5. TCP Window Scan

**TCP window scan** est similaire à ACK scan mais vérifie la **taille de fenêtre TCP** dans la réponse RST pour déterminer ports ouverts vs fermés.

- **Option Nmap**: `-sW`
- **Requiert**: privilèges root
- **Comment ça marche**:
  1. Envoyer paquet TCP ACK.
  2. Si RST a **fenêtre non-zéro** → port **ouvert**.
  3. Si RST a **fenêtre zéro** → port **fermé**.

- **Avantages**:
  - Peut détecter ports ouverts (contrairement à ACK scan).
  - Parfois bypass les pare-feux.
- **Inconvénients**:
  - Rarement plus utile que SYN scan.
  - Pas fiable sur tous les systèmes.

Exemple:
```bash
nmap -sW 192.168.1.5
```

---

### 6. TCP NULL, FIN et Xmas Scans

Ce sont des **scans furtifs** qui envoient des paquets **sans drapeaux** (NULL), **seulement FIN**, ou **FIN+URG+PSH** (Xmas). Ils exploitent comment les systèmes gèrent les paquets malformés.

- **Options Nmap**:
  - `-sN` (NULL scan)
  - `-sF` (FIN scan)
  - `-sX` (Xmas scan)
- **Requiert**: privilèges root
- **Comment ça marche**:
  - Envoyer paquet **sans SYN** (bypasse certains pare-feux stateful).
  - Si recevoir **RST** → port **fermé**.
  - Si **pas de réponse** → port **open|filtered**.

- **Avantages**:
  - **Furtif** (pas de drapeau SYN, plus difficile à détecter).
  - Peut bypass certains pare-feux.
- **Inconvénients**:
  - Fonctionne seulement sur **Unix/Linux** (Windows répond différemment).
  - Pas fiable; beaucoup de systèmes modernes répondent avec RST de toute façon.

Exemple:
```bash
nmap -sN 192.168.1.5
nmap -sF 192.168.1.5
nmap -sX 192.168.1.5
```

---

## Paramètres de Temps Nmap (`-T`)

Les paramètres de temps de Nmap contrôlent la **vitesse et l'agressivité** du scan. Des valeurs plus élevées sont plus rapides mais plus susceptibles d'être détectées.

| Option | Nom | Description | Cas d'Usage |
|--------|-----|-------------|-------------|
| `-T0` | **Paranoid** | Extrêmement lent; focalisé sur l'évasion | Évasion IDS, très furtif |
| `-T1` | **Sneaky** | Très lent; évite la détection | Scan furtif |
| `-T2` | **Polite** | Lent; réduit l'utilisation de bande passante | Réseaux à faible bande passante |
| `-T3` | **Normal** | Vitesse par défaut | Scan standard |
| `-T4` | **Aggressive** | Rapide; suppose réseau moderne | Scans rapides sur réseaux fiables |
| `-T5` | **Insane** | Très rapide; risque élevé d'erreurs | Vitesse sur précision |

---

## Sélection de Ports

Vous pouvez spécifier quels ports scanner en utilisant `-p` :

| Option | Description |
|--------|-------------|
| `-p 80,443,22` | Scanner ports spécifiques |
| `-p 1-1000` | Scanner ports 1 à 1000 |
| `-p-` | Scanner **tous les 65535 ports** |
| `--top-ports 100` | Scanner **top 100 ports les plus courants** |
| `-p U:53,T:80,443` | Scanner UDP 53 + TCP 80,443 |

Exemple:
```bash
nmap -sS -p 22,80,443 192.168.1.5
nmap -sS -p- 192.168.1.5
nmap -sS --top-ports 1000 192.168.1.5
```

---

## Détection de Version de Service

**-sV** sonde les ports ouverts pour déterminer la **version du service** (ex. Apache 2.4.49, OpenSSH 8.4) :

- **Option Nmap**: `-sV`
- **Usage**: quand vous avez besoin de savoir les versions exactes pour l'exploitation.

Exemple:
```bash
nmap -sS -sV 192.168.1.5
nmap -sS -sV --version-intensity 5 192.168.1.5   # intensité maximale
```

---

## Détection d'OS

**-O** tente de détecter le **système d'exploitation** de la cible:

- **Option Nmap**: `-O`
- **Requiert**: privilèges root
- **Comment ça marche**: Analyse les empreintes de pile TCP/IP.

Exemple:
```bash
nmap -sS -O 192.168.1.5
```

---

## Scan Aggressif (A)

**-A** active **détection d'OS, détection de version, scan de scripts et traceroute** tous en même temps:

- **Option Nmap**: `-A`
- **Équivalent à**: `-O -sV -sC --traceroute`
- **Usage**: quand vous voulez l'information maximale.

Exemple:
```bash
nmap -A 192.168.1.5
```

---

## Nmap Scripting Engine (NSE)

**-sC** exécute les **scripts NSE par défaut** pour reconnaissance supplémentaire (vulnérabilités, configs, etc.) :

- **Option Nmap**: `-sC`
- **Usage**: quand vous voulez exécuter des vérifications communes de vulnérabilités.

Exemple:
```bash
nmap -sS -sC 192.168.1.5
nmap --script vuln 192.168.1.5
nmap --script http-enum 192.168.1.5
```

---

## Combinaisons Courantes de Scan

| Scénario | Commande |
|----------|----------|
| **Top 100 ports rapide** | `nmap -sS --top-ports 100 <cible>` |
| **Scan TCP complet** | `nmap -sS -p- <cible>` |
| **Scan complet + version + OS** | `nmap -sS -sV -O -p- <cible>` |
| **Scan agressif** | `nmap -A <cible>` |
| **Scan UDP (top 100)** | `nmap -sU --top-ports 100 <cible>` |
| **TCP + UDP complet** | `nmap -sS -sU -p- <cible>` |
| **Sans root** | `nmap -sT -p 80,443 <cible>` |
| **Mapping de pare-feu** | `nmap -sA <cible>` |
| **Sauter découverte d'hôte (-Pn)** | `nmap -sS -Pn <cible>` |

---

## États des Ports

Nmap rapporte les ports dans ces états:

| État | Signification |
|------|---------------|
| **open** | Service accepte les connexions |
| **closed** | Hôte est atteignable, mais pas de service sur le port |
| **filtered** | Pare-feu/IDS bloque le scan |
| **unfiltered** | Port est accessible, mais ouvert/fermé inconnu |
| **open\|filtered** | Nmap ne peut pas déterminer si ouvert ou filtré |
| **closed\|filtered** | Nmap ne peut pas déterminer si fermé ou filtré |

---

## Points Clés

- **TCP SYN scan (`-sS`)** est le par défaut; rapide, furtif, nécessite root.
- **TCP Connect scan (`-sT`)** est pour les utilisateurs sans root; plus lent, plus visible.
- **UDP scan (`-sU`)** est lent mais nécessaire pour les services UDP (DNS, SNMP).
- **ACK scan (`-sA`)** mappe les règles de pare-feu, pas les ports ouverts.
- **NULL/FIN/Xmas scans** sont furtifs mais peu fiables sur Windows.
- **`-sV`** sonde les versions de services pour l'exploitation.
- **`-O`** détecte l'OS de la cible.
- **`-A`** est agressif: OS + version + scripts + traceroute.
- **`-sC`** exécute les scripts NSE par défaut pour vérifications de vulnérabilités.
- Utilisez **`-p-`** pour scan complet de ports (tous les 65535 ports).
- Utilisez **`--top-ports 100`** pour scan rapide des ports les plus courants.
