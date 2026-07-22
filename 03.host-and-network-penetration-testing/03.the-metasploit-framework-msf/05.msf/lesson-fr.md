# MSF

## Vue d’ensemble

La détection de vulnérabilités consiste à vérifier si une cible présente des vulnérabilités connues et à confirmer si elles peuvent être exploitées. Dans cette phase, Metasploit est utilisé avec des modules auxiliaires et des modules d’exploitation pour identifier des faiblesses dans les services, les systèmes d’exploitation et les applications web.

Cette phase est importante parce qu’elle permet de réduire les vulnérabilités à tester pendant l’exploitation. Elle permet aussi d’organiser les résultats dans Metasploit afin de les réutiliser plus tard.

## Environnement de laboratoire

Pour cette partie, le laboratoire utilise **Metasploitable3**, une machine virtuelle intentionnellement vulnérable basée sur Windows Server 2008. Elle a été créée par Rapid7 pour montrer comment Metasploit peut être utilisé contre un système Windows.

## Gestion des workspaces

Les workspaces sont utiles pour séparer les données de scan par cible ou par réseau.

### Commandes utiles

Lister les workspaces :

```bash
workspace
```

Créer un workspace :

```bash
workspace -a lab1
```

Changer de workspace :

```bash
workspace lab1
```

Supprimer un workspace :

```bash
workspace -d lab1
```

## Scan et analyse

Avant de scanner, assure-toi que la base de données est active et que le bon workspace est sélectionné.

### Vérifier l’état de la base de données

```bash
db_status
```

### Voir les hôtes

```bash
hosts
```

### Voir les services

```bash
services
```

### Voir les vulnérabilités

```bash
vulns
```

## Commandes de scan de vulnérabilités

Metasploit peut scanner un hôte unique ou un sous-réseau entier.

Scanner un hôte :

```bash
set RHOSTS <target_ip>
```

Scanner un sous-réseau :

```bash
set RHOSTS <target_network>/24
```

### Rechercher des modules pertinents

Rechercher par service ou par nom de vulnérabilité :

```bash
search <keyword>
```

Exemples :

```bash
search smb
search http
search mysql
search ftp
```

### Vérifier un module

Certains modules d’exploitation permettent d’utiliser une vérification sûre pour confirmer si la cible est probablement vulnérable :

```bash
check
```

### Lancer un module

```bash
run
```

ou

```bash
exploit
```

## Importer les résultats de scan

Metasploit peut aussi importer les résultats d’outils externes comme Nmap et Nessus.

Importer un scan Nmap au format XML :

```bash
db_import scan.xml
```

Après l’importation, vérifie les résultats :

```bash
hosts
services
vulns
```

## Searchsploit

`searchsploit` est utile pour chercher des exploits publics après avoir identifié un service ou une version vulnérable.

Rechercher par nom de logiciel :

```bash
searchsploit <software_name>
```

Rechercher par version :

```bash
searchsploit <software_name> <version>
```

Copier un exploit en local si nécessaire :

```bash
searchsploit -m <exploit_id>
```

## Metasploit Autopwn

`metasploit-autopwn` était une ancienne méthode d’automatisation utilisée pour associer des services à d’éventuels exploits. Ce n’est plus considéré comme un flux de travail standard, mais il apparaît encore dans certains supports de formation comme référence historique.

## Intégration avec Nessus

Les résultats de Nessus peuvent aussi être importés dans Metasploit si tu disposes d’une exportation compatible. Cela permet de centraliser les données de vulnérabilités dans la base de données de Metasploit.

## Flux de travail utile

1. Sélectionner ou créer un workspace.
2. Vérifier la base de données avec `db_status`.
3. Importer des résultats de scan ou scanner directement la cible.
4. Examiner `hosts`, `services` et `vulns`.
5. Rechercher des modules avec `search`.
6. Confirmer la vulnérabilité avec `check`.
7. Utiliser `searchsploit` pour comparer les résultats avec des exploits publics.

## Idée clé

La détection de vulnérabilités avec Metasploit consiste à identifier des faiblesses potentielles et à bien organiser les résultats. Les workspaces, la base de données, les scans importés et les vérifications de modules facilitent le passage de la découverte à l’exploitation.
