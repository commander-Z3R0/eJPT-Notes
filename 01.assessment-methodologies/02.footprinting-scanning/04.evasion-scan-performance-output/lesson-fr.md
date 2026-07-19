# Évasion, performances de scan et sortie

Nmap ne sert pas seulement à scanner des ports. Il offre aussi des options pour **contourner des filtres basiques**, **contrôler la vitesse du scan** et **enregistrer les résultats dans différents formats** pour les analyser plus tard.

---

## Évasion

L’**évasion** désigne les techniques utilisées pour rendre les scans plus difficiles à détecter ou à bloquer par des pare-feu, des IDS ou des IPS. En pentest, l’objectif n’est pas de “pirater plus vite”, mais de comprendre comment les défenses réagissent à des paquets et à des modèles de scan inhabituels.

Nmap inclut plusieurs options liées à l’évasion qui peuvent être utiles lorsqu’un réseau cible filtre ou inspecte le trafic :

- **La fragmentation** divise les paquets en fragments IP plus petits.
- **Les decoys** masquent la vraie source en mélangeant de fausses adresses IP au scan.
- **La taille de données personnalisée** modifie l’apparence du paquet sur le réseau.
- **L’usurpation de source** peut faire croire que le trafic vient d’une autre adresse, même si les réponses ne reviendront généralement pas correctement.

Ces techniques sont utiles dans des tests contrôlés, car elles permettent d’évaluer si un réseau peut détecter ou bloquer un trafic de scan non standard.

---

## Fragmentation des paquets

La **fragmentation des paquets** divise les probes de Nmap en plus petits fragments IP afin que certains filtres de paquets ou outils IDS aient plus de difficulté à inspecter tout l’en-tête TCP/UDP d’un seul bloc.

### `-f`
L’option `-f` demande à Nmap de fragmenter les paquets en **petits fragments**. Nmap découpe le paquet en fragments de **8 octets ou moins après l’en-tête IP**. Si tu utilises `-f` deux fois, Nmap augmente la taille des fragments à **16 octets**.

Exemple :
```bash
nmap -sS -f <target_ip>
nmap -sS -ff <target_ip>
```

### `--mtu`
L’option `--mtu` permet de définir manuellement la taille du fragment. La valeur doit être un **multiple de 8**, et tu ne dois pas l’utiliser avec `-f`.

Exemple :
```bash
nmap -sS --mtu 24 <target_ip>
```

### `--data-length`
L’option `--data-length` ajoute des octets aléatoires aux paquets envoyés. Cela rend les probes moins minimales et peut légèrement modifier leur apparence pour les outils de surveillance.

Exemple :
```bash
nmap -sS --data-length 16 <target_ip>
```

### `-D`
L’option `-D` utilise des **decoys**. Nmap fait en sorte que la cible voie plusieurs adresses IP sources en train de scanner en même temps, ce qui rend plus difficile l’identification du vrai scanner.

Exemple :
```bash
nmap -sS -D 10.10.10.10,10.10.10.11,ME <target_ip>
nmap -sS -D RND:5 <target_ip>
```

### Quand utiliser ces options
Ces fonctions sont utiles lorsque tu veux tester si un pare-feu ou un IDS détecte un trafic de scan non standard, ou lorsque tu veux comprendre comment les contrôles défensifs se comportent face à des probes inhabituels. Elles peuvent aussi ralentir le scan et le rendre moins fiable, donc elles ne sont pas toujours le meilleur choix pour un scan de base classique.

---

## Performance du scan

La **performance du scan** correspond à l’équilibre entre vitesse, précision et impact sur le réseau. En pentest, elle est importante parce qu’un scan trop lent fait perdre du temps, tandis qu’un scan trop agressif peut déclencher des alertes, perdre des paquets ou produire des résultats inexacts.

Une bonne stratégie de scan dépend de l’environnement :

- Sur des réseaux internes stables, les scans rapides sont souvent acceptables.
- Sur des réseaux bruyants, fragiles ou fortement surveillés, des scans plus lents peuvent être plus fiables.
- Dans les évaluations réelles, les choix de timing influencent à la fois le **risque de détection** et la **qualité du rapport**.

Nmap fournit plusieurs options pour ajuster ce comportement.

### `-T`
Le modèle de timing `-T` contrôle le niveau d’agressivité de Nmap. Nmap définit six profils : **paranoid (`-T0`)**, **sneaky (`-T1`)**, **polite (`-T2`)**, **normal (`-T3`)**, **aggressive (`-T4`)** et **insane (`-T5`)**.

Exemples :
```bash
nmap -T3 <target_ip>
nmap -T4 <target_ip>
```

- `-T0` et `-T1` sont plus lents et sont utilisés lorsque la discrétion est plus importante que la vitesse.
- `-T3` est la valeur par défaut et constitue généralement un bon point de départ.
- `-T4` et `-T5` sont plus rapides, mais augmentent aussi le bruit et le risque de perdre des réponses.

### `--host-timeout`
`--host-timeout` définit le temps maximum que Nmap consacrera à un hôte avant d’abandonner. C’est utile lorsqu’une cible est lente, filtrée ou fait bloquer le scan.

Exemple :
```bash
nmap --host-timeout 2m <target_ip>
```

### `--min-rate`
`--min-rate` oblige Nmap à envoyer des probes à une vitesse minimale en paquets par seconde. Cela peut accélérer les grands scans, mais aussi augmenter la perte de paquets ou le risque de détection.

Exemple :
```bash
nmap --min-rate 100 <target_ip>
```

### `--scan-delay`
`--scan-delay` insère une pause entre les probes envoyés vers un même hôte. C’est utile si tu veux ralentir le trafic et réduire le risque de déclencher des alertes.

Exemple :
```bash
nmap --scan-delay 200ms <target_ip>
```

### Vision pratique de la performance
En footprinting, la performance ne concerne pas seulement la vitesse. Elle concerne aussi le **contrôle** : savoir quand scanner vite, quand ralentir, et quand accepter un temps plus long en échange de résultats plus propres. Les contrôles de timing de Nmap te permettent d’adapter le scan à l’environnement au lieu d’utiliser la même vitesse partout.

---

## Formats de sortie

Nmap peut enregistrer les résultats dans différents formats pour les relire, les filtrer ou les importer plus tard. C’est particulièrement utile en footprinting, car la sortie du scan devient souvent la base des notes, des rapports et des analyses ultérieures.

### `-oN`
`-oN` enregistre le scan dans un format texte **normal**. Il ressemble à la sortie affichée dans le terminal, mais il est écrit dans un fichier.

Exemple :
```bash
nmap -sS -oN scan.txt <target_ip>
```

### `-oX`
`-oX` enregistre le scan au format **XML**. C’est le format le plus utile si tu veux réutiliser les résultats dans d’autres outils ou automatiser leur traitement.

Exemple :
```bash
nmap -sS -oX scan.xml <target_ip>
```

### `-oG`
`-oG` enregistre le scan dans un format **grepable**. Ce format ancien est pensé pour être facile à analyser avec des outils shell et des scripts simples.

Exemple :
```bash
nmap -sS -oG scan.gnmap <target_ip>
```

### `-oA`
`-oA` enregistre la sortie dans **tous les formats principaux à la fois** : normal, XML et grepable. C’est pratique lorsqu’un seul scan doit produire plusieurs types de rapport.

Exemple :
```bash
nmap -sS -oA scan_results <target_ip>
```

### Pourquoi XML est important
Le XML est particulièrement important parce que beaucoup d’outils peuvent l’importer directement. Il conserve des données structurées du scan comme les hôtes, les ports, les services et les métadonnées.

---

## Importation dans Metasploit

En footprinting, tu peux utiliser les fonctions de base de données de **Metasploit** pour importer des résultats XML de Nmap et organiser les hôtes et services découverts pour les revoir plus tard. L’idée ici est de rester dans la phase de reconnaissance et de reporting, pas dans l’exploitation.

Flux typique :
```bash
nmap -sS -sV -oX scan.xml <target_ip>
msfconsole
db_import scan.xml
hosts
services
```

Ce qui se passe ici :

- `nmap -oX` génère un rapport XML.
- `msfconsole` ouvre Metasploit.
- `db_import` charge le XML dans la base de données de Metasploit.
- `hosts` et `services` permettent de revoir les données de footprinting importées.

C’est utile parce que cela centralise les résultats de reconnaissance et facilite l’analyse ultérieure, surtout dans les évaluations de grande taille.

---

## Usage recommandé

Un flux simple de footprinting pourrait ressembler à ceci :

```bash
nmap -sS -T3 -oN scan.txt <target_ip>
nmap -sS -T4 --scan-delay 100ms -oX scan.xml <target_ip>
nmap -sS -f -D RND:5 -oG scan.gnmap <target_ip>
```

Utilise des réglages rapides lorsque le réseau est stable, et des options plus lentes ou plus prudentes lorsque tu veux réduire le bruit ou tester des contrôles défensifs.

---

## Points clés

- **L’évasion** consiste à modifier l’apparence des probes pour que les pare-feu et IDS aient plus de difficulté à les détecter.
- `-f` fragmente les paquets, `--mtu` définit la taille du fragment, `--data-length` ajoute un payload aléatoire et `-D` utilise des decoys.
- La **performance** consiste à trouver un équilibre entre vitesse, précision et discrétion pendant le scan.
- `-T`, `--host-timeout`, `--min-rate` et `--scan-delay` sont les principaux contrôles de timing.
- `-oN`, `-oX` et `-oG` enregistrent les résultats dans différents formats pour l’analyse et le reporting.
- La sortie XML de Nmap est particulièrement utile car elle peut être importée dans `msfconsole` avec `db_import` pendant le footprinting.
