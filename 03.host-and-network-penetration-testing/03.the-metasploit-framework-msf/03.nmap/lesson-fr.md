# Nmap

## Résumé

**Nmap** est un scanner réseau libre et open source utilisé pour découvrir les hôtes sur un réseau, analyser les cibles à la recherche de ports ouverts et identifier les services et les systèmes d’exploitation présents sur ces cibles.

Il peut aussi exporter les résultats d’un scan dans un format qui peut être importé dans Metasploit pour la détection de vulnérabilités et l’exploitation.

## Scan de ports et énumération

Nmap est couramment utilisé pour :

- Découvrir les hôtes actifs sur un réseau.
- Rechercher les ports ouverts.
- Identifier les services exécutés sur les ports ouverts.
- Détecter le système d’exploitation de la cible.

### Syntaxe de base de Nmap

```bash
nmap <target_ip>
```

### Exemples courants

Scanner un hôte unique :

```bash
nmap <target_ip>
```

Scanner une plage réseau :

```bash
nmap <target_network>/24
```

Scanner des ports spécifiques :

```bash
nmap -p 22,80,443 <target_ip>
```

Identifier les versions des services :

```bash
nmap -sV <target_ip>
```

Détecter le système d’exploitation :

```bash
nmap -O <target_ip>
```

## Importer les résultats Nmap dans MSF

Metasploit peut importer les résultats d’un scan Nmap et les utiliser pour remplir sa base de données avec les hôtes et les services.

### Exporter les résultats Nmap

Pour importer les résultats dans Metasploit, la sortie de Nmap doit être enregistrée au format XML :

```bash
nmap -sV -O -oX scan.xml <target_ip>
```

ou

```bash
nmap -sV -O -oX scan.xml <target_network>/24
```

### Importer dans Metasploit

Dans `msfconsole`, utilise :

```bash
db_import scan.xml
```

Après l’importation, tu peux afficher les hôtes et les services détectés avec :

```bash
hosts
services
```

## Pourquoi c’est important

Importer les résultats de Nmap dans Metasploit permet de gagner du temps et d’organiser les données de l’évaluation. Cela permet de réutiliser les résultats pour la suite de l’exploitation et de centraliser les informations sur les hôtes et les services.

## Idée clé

Nmap est l’un des premiers outils utilisés dans un test d’intrusion, car il donne une vue rapide des hôtes actifs, des ports ouverts, des services et des systèmes d’exploitation. Sa sortie XML peut être importée directement dans Metasploit pour poursuivre efficacement l’évaluation.
