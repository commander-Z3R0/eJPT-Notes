# Reconnaissance

## Reconnaissance passive

La collecte passive d’informations consiste à recueillir autant de données que possible sans interagir activement avec le système cible.

### Informations à collecter
- Adresses IP et informations DNS.
- Noms de domaine et informations de propriété.
- Adresses e-mail et profils sur les réseaux sociaux.
- Technologies web utilisées sur les sites cibles.
- Sous-domaines.

### Commande Host
La commande `host` est un utilitaire simple de requête DNS sous Unix/Linux. Elle permet de trouver l’adresse IP associée à un domaine, d’identifier les serveurs de messagerie, etc.

```bash
host -a <domain>
```

Si plusieurs adresses IP sont renvoyées, la cible peut utiliser un proxy ou un CDN comme Cloudflare.

### robots.txt
Une fois sur un site web, consultez le fichier `/robots.txt`. Il indique aux robots d’indexation quelles parties du site ne doivent pas être indexées et peut révéler des chemins cachés ou sensibles.

### sitemap.xml
Consultez le fichier `/sitemap.xml` pour voir toutes les pages indexées du site. Cela permet de comprendre plus rapidement la structure du site.

### Technologies web
Utilisez des outils tels que :
- `BuiltWith`
- `Wappalyzer`
- `whatweb`

Ces outils permettent d’identifier les technologies utilisées par un site cible.

```bash
whatweb <url>
```

### HTTrack
Utilisez `httrack` pour télécharger un site web complet et inspecter localement ses répertoires et fichiers.

```bash
httrack <url>
```

### Énumération Whois
`whois` est un protocole utilisé pour interroger des bases de données contenant des informations d’enregistrement sur les noms de domaine, les blocs d’adresses IP et les systèmes autonomes.

```bash
whois <domain>
whois <ip>
```

Vous pouvez également utiliser :
- https://who.is/
- https://www.whois.com/

### Netcraft
Netcraft (https://sitereport.netcraft.com/) est utile pour la reconnaissance passive des sites web. Il peut révéler les technologies, certificats et informations côté serveur.

### DNS Recon
`dnsrecon` est un outil d’énumération DNS capable de découvrir :
- Les enregistrements `A`, `AAAA`, `NS`, `MX`, `SOA`, `TXT` et `SPF`.
- Les transferts de zone.
- Les recherches inverses.
- Les sous-domaines.

```bash
dnsrecon -d <domain>
```

Un autre outil utile est `dnsdumpster`, qui fournit une carte visuelle du DNS.

### Wafw00f
Un WAF (Web Application Firewall) protège les applications web en filtrant le trafic malveillant. `wafw00f` permet d’identifier quel WAF est utilisé par un site web.

```bash
wafw00f <url>
```

### Sublist3r
`Sublist3r` énumère les sous-domaines à l’aide de sources OSINT. N’utilisez pas l’option brute force si vous souhaitez rester en reconnaissance passive.

```bash
sublist3r -d <domain>
```

### Google Dorks
Les Google dorks utilisent des opérateurs de recherche spécifiques pour trouver des informations ciblées.

Exemples :
- `site:<domain>`
- `filetype:pdf`
- `inurl:index.php`
- `intitle:index of`
- `cache:<url>`
- `related:<url>`

Ressources utiles :
- https://www.exploit-db.com/google-hacking-database
- https://archive.org/

### TheHarvester
`theHarvester` est conçu pour l’OSINT. Il collecte des e-mails, noms, sous-domaines, adresses IP, hôtes et URLs à partir de sources publiques.

```bash
theHarvester -d <domain> -b all
```

Outils et services utiles :
- https://phonebook.cz/
- https://search.censys.io/
- https://www.shodan.io/

### Bases de données divulguées
Utilisez https://haveibeenpwned.com/ pour vérifier si une adresse e-mail a été exposée dans une fuite de données.
