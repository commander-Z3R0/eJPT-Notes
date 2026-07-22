# Web Apps

## Vue d’ensemble

**WMAP** est un scanner de vulnérabilités pour applications web intégré à Metasploit. Il sert à automatiser l’énumération des serveurs web et à scanner les applications web à la recherche de vulnérabilités depuis `msfconsole`.

WMAP est utile car il garde tout le flux de travail dans Metasploit, ce qui permet d’enregistrer les sites, les chemins et les vulnérabilités dans la base de données du framework.

## Environnement de laboratoire

Pour pratiquer, le cours utilise une cible intentionnellement vulnérable comme **Metasploitable3**. Dans le laboratoire, tu scannera généralement un service web sur `<target_ip>` ou une application web accessible sur `<target_network>/24`.

## Préparation

Avant d’utiliser WMAP, assure-toi que Metasploit et la base de données fonctionnent.

```bash
sudo systemctl start postgresql
msfdb init
msfconsole
```

Vérifie l’état de la base de données :

```bash
db_status
```

Crée ou change de workspace :

```bash
workspace -a webapp
workspace webapp
```

## Charger WMAP

Dans `msfconsole`, charge le plugin avec :

```bash
load wmap
```

S’il se charge correctement, les commandes WMAP deviennent disponibles dans la console.

## Ajouter un site

Avant de scanner, ajoute l’application web cible comme site.

Ajouter une cible unique :

```bash
wmap_sites -a <target_ip>
```

Ou ajouter une cible via une URL :

```bash
wmap_targets -t http://<target_ip>
```

Pour une application web hébergée sur un réseau, utilise :

```bash
wmap_targets -t http://<target_network>/24
```

Lister les sites configurés :

```bash
wmap_sites -l
```

Lister les cibles configurées :

```bash
wmap_targets -l
```

## Lancer les scans WMAP

WMAP peut cartographier le site puis exécuter les scanners activés contre lui.

Lister les modules de scan disponibles :

```bash
wmap_run -t
```

Exécuter tous les modules activés :

```bash
wmap_run -e
```

Selon la taille de l’application, cela peut prendre un certain temps.

## Consulter les résultats

Une fois le scan terminé, consulte les vulnérabilités et chemins découverts.

```bash
wmap_vulns -l
```

Tu peux aussi consulter les données générales de la base Metasploit avec :

```bash
hosts
services
vulns
```

## Flux de travail utile

1. Démarrer PostgreSQL et Metasploit.
2. Vérifier `db_status`.
3. Créer un workspace.
4. Charger WMAP avec `load wmap`.
5. Ajouter le site avec `wmap_sites -a <target_ip>`.
6. Définir la cible avec `wmap_targets -t http://<target_ip>`.
7. Lister les cibles et les sites pour confirmer.
8. Lancer `wmap_run -t` puis `wmap_run -e`.
9. Vérifier les problèmes détectés avec `wmap_vulns -l`.
10. Consulter `hosts`, `services` et `vulns` dans la base de données.

## Pourquoi c’est important

WMAP est utile car il automatise la reconnaissance et le scan de vulnérabilités web depuis Metasploit. Cela facilite le lien entre la découverte et l’exploitation ultérieure, tout en gardant tout organisé au même endroit.

## Idée clé

WMAP est le scanner d’applications web intégré à Metasploit. Il aide à cartographier un site, à le scanner à la recherche de vulnérabilités et à enregistrer les résultats dans la base de données de Metasploit pour les utiliser plus tard.
