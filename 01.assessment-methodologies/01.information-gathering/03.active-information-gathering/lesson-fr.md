# Reconnaissance

## Reconnaissance active

La collecte active d’informations implique une interaction avec le système cible et nécessite une autorisation.

### Transferts de zone DNS
Le DNS (Domain Name System) résout les noms de domaine et d’hôtes en adresses IP. Les serveurs DNS stockent des enregistrements pour la majorité des domaines sur Internet.

Types d’enregistrements DNS courants :
- `A` — Nom d’hôte ou domaine vers une adresse IPv4.
- `AAAA` — Nom d’hôte ou domaine vers une adresse IPv6.
- `NS` — Serveur de noms du domaine.
- `MX` — Domaine vers un serveur de messagerie.
- `CNAME` — Alias de domaine.
- `TXT` — Enregistrement texte.
- `HINFO` — Informations sur l’hôte.
- `SOA` — Autorité du domaine.
- `SRV` — Enregistrements de service.
- `PTR` — Adresse IP vers un nom d’hôte.

L’interrogation DNS est le processus d’énumération des enregistrements DNS pour un domaine spécifique. Les transferts de zone peuvent être exploités si les serveurs DNS sont mal configurés.

Vous pouvez effectuer un transfert de zone avec :

```bash
dig axfr @<dns-server> <domain>
dnsenum <domain>
```

`Fierce` est un autre outil utile, souvent utilisé avant Nmap.

### Nmap
Pour trouver votre propre adresse IP :

```bash
ip a
```

Pour la découverte d’hôtes :

```bash
nmap -sn <subnet>
netdiscover
```

N’oubliez pas d’utiliser `sudo` lorsque nécessaire.

## Options Nmap

| Option    | Utilisation                                                              |
| --------- | ------------------------------------------------------------------------ |
| -v        | Mode verbeux, affiche ce qui se passe en temps réel.                     |
| -sV       | Détecte la version d’un service sur un port.                             |
| -O        | Détecte le système d’exploitation en cours d’exécution.                  |
| -A        | Active toutes les détections (OS, version, scripts, etc.).               |
| -T0 à -T5 | Définit l’intensité du scan de lent à rapide.                            |
| -sC       | Scan avec les scripts NSE par défaut.                                    |
| -F        | Scan plus rapide avec un ensemble réduit de ports courants.              |
| -p        | Scanne tous les ports ou des ports spécifiques.                          |
| -Pn       | Désactive la découverte d’hôtes et scanne uniquement les ports.          |
| -sn       | Désactive le scan de ports et effectue uniquement la découverte d’hôtes. |
| -sU       | Scan des ports UDP.                                                      |
| -sS       | Scan TCP SYN.                                                            |
