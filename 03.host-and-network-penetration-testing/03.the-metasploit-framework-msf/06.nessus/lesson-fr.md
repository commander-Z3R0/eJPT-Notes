# Nessus

## Vue d’ensemble

**Nessus** est un scanner de vulnérabilités propriétaire développé par Tenable. Il est utilisé pour analyser un système cible à la recherche de vulnérabilités et peut fournir des détails utiles comme l’identifiant CVE, la gravité et d’autres informations liées à chaque faille.

Un scan Nessus est utile en lui-même, mais il devient encore plus puissant lorsque tu importes les résultats dans Metasploit Framework pour l’analyse, la validation et éventuellement l’exploitation.

## Environnement de laboratoire

Pour cette partie, le laboratoire utilise **Metasploitable3**, une machine virtuelle intentionnellement vulnérable basée sur Windows Server 2008. Elle a été créée par Rapid7 pour montrer comment Metasploit peut être utilisé contre un système Windows.

## Installation de Nessus

L’édition gratuite est **Nessus Essentials**, qui permet de scanner jusqu’à 16 IPs. Elle suffit pour les laboratoires et l’entraînement.

### Flux d’installation général

1. Télécharger l’installateur Nessus depuis Tenable.
2. Installer le paquet sur le système.
3. Démarrer le service Nessus.
4. Ouvrir l’interface web sur le port `8834`.
5. Activer Nessus Essentials et créer un compte utilisateur.
6. Attendre la fin du téléchargement et de la mise à jour des plugins.

### Exemple sur Debian ou Kali

```bash
sudo apt install ./Nessus-*.deb
sudo systemctl start nessusd
sudo systemctl enable nessusd
```

Puis ouvrir le navigateur et aller sur :

```bash
https://127.0.0.1:8834
```

Si tu y accèdes depuis une autre machine :

```bash
https://<your_ip>:8834
```

## Créer un scan

Une fois connecté, Nessus permet de créer un nouveau scan et de choisir une politique.

### Flux de travail typique

1. Créer un nouveau scan.
2. Sélectionner le type ou le modèle de scan.
3. Saisir l’IP cible ou la plage réseau.
4. Lancer le scan.
5. Analyser les résultats.

### Exemples de cible

Scanner un hôte :

```bash
<target_ip>
```

Scanner une plage réseau :

```bash
<target_network>/24
```

## Ce que Nessus rapporte

Un rapport Nessus peut inclure :

- Les ports ouverts.
- Les services détectés.
- Les versions de services.
- Les vulnérabilités.
- Les références CVE.
- Les niveaux de gravité.
- Les recommandations de remédiation.

Cela le rend très utile pour passer de la découverte à l’exploitation.

## Importer les résultats Nessus dans Metasploit

Metasploit peut importer la sortie Nessus pour intégrer les données de scan dans sa base de données. Cela aide à centraliser les informations de l’audit et à revoir plus facilement les hôtes, services et vulnérabilités.

### Commande d’importation

Si tu exportes le rapport Nessus dans un format compatible, importe-le dans `msfconsole` avec :

```bash
db_import scan.nessus
```

Après l’importation, tu peux consulter les données avec :

```bash
hosts
services
vulns
```

## Commandes utiles dans Metasploit

Vérifier l’état de la base de données :

```bash
db_status
```

Voir les hôtes importés :

```bash
hosts
```

Voir les services importés :

```bash
services
```

Voir les vulnérabilités :

```bash
vulns
```

Rechercher un module d’exploitation lié à une faille :

```bash
search <keyword>
```

Obtenir des informations détaillées sur un module :

```bash
info <module_name>
```

Utiliser un module :

```bash
use <module_path>
```

Vérifier si une cible est vulnérable :

```bash
check
```

Lancer l’exploitation :

```bash
exploit
```

## Exemple de flux de travail

Un flux simple ressemblerait à ceci :

1. Scanner `<target_ip>` ou `<target_network>/24` avec Nessus.
2. Exporter le rapport.
3. L’importer dans Metasploit avec `db_import`.
4. Examiner `hosts`, `services` et `vulns`.
5. Rechercher un exploit correspondant.
6. Confirmer avec `check`.
7. Exploiter si c’est approprié.

## Pourquoi c’est important

Nessus automatise la détection de vulnérabilités et fournit des résultats structurés qui peuvent être réutilisés dans Metasploit. Cela fait gagner du temps, aide à organiser les résultats et rend la transition entre le scan et l’exploitation beaucoup plus fluide.

## Idée clé

Nessus sert à identifier rapidement et clairement les vulnérabilités, tandis que Metasploit sert à les valider et à les exploiter. Importer les résultats Nessus dans Metasploit donne un flux de travail plus propre et plus efficace pour le test d’intrusion.
