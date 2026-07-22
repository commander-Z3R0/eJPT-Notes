# The Metasploit Framework

## Résumé

Le **Metasploit Framework (MSF)** est un framework de test d’intrusion et d’exploitation open source utilisé par les pentesters et les chercheurs en sécurité. Il fournit la structure nécessaire pour automatiser de nombreuses étapes du cycle d’un test d’intrusion, de la reconnaissance à la post-exploitation.

Il est aussi utilisé pour développer et tester des exploits, et possède une très grande base de données d’exploits publics et testés. Sa conception modulaire facilite l’ajout de nouvelles fonctionnalités.

## Historique

- Développé par **HD Moore** en **2003**.
- Écrit à l’origine en **Perl**.
- Réécrit en **Ruby** en **2007**.
- Racheté par **Rapid7** en **2009**.
- **Metasploit 5.0** a été publié en **2019**.
- **Metasploit 6.0** a été publié en **2020**.

## Éditions

- **Metasploit Pro** — commerciale.
- **Metasploit Express** — commerciale.
- **Metasploit Framework** — édition communautaire.

## Terminologie essentielle

| Terme | Signification |
|---|---|
| **Interface** | Façon d’interagir avec Metasploit. |
| **Module** | Fragment de code qui effectue une tâche précise, comme un exploit. |
| **Vulnérabilité** | Faiblesse ou défaut dans un système ou un réseau pouvant être exploité. |
| **Exploit** | Code ou module utilisé pour tirer parti d’une vulnérabilité. |
| **Payload** | Code livré au système cible par un exploit, généralement pour exécuter des commandes ou fournir un accès distant. |
| **Listener** | Utilitaire qui attend une connexion entrante depuis la cible. |

## Interfaces principales

### MSFconsole

La **Metasploit Framework Console (MSFconsole)** est une interface tout-en-un simple d’utilisation qui donne accès à toutes les fonctionnalités de Metasploit.

### MSFcli

La **Metasploit Framework Command Line Interface (MSFcli)** était un utilitaire en ligne de commande utilisé pour faciliter la création de scripts d’automatisation avec les modules Metasploit. Elle permettait de rediriger la sortie entre d’autres outils et Metasploit. Cette interface a été abandonnée en 2015, mais une fonctionnalité similaire reste disponible via MSFconsole.

### Metasploit Community Edition

**Metasploit Community Edition** est une interface graphique web qui simplifie la découverte du réseau et l’identification des vulnérabilités.

### Armitage

**Armitage** est une interface graphique gratuite basée sur Java pour Metasploit qui simplifie la découverte du réseau, l’exploitation et la post-exploitation.

## Architecture

Metasploit est modulaire, et chaque module effectue une tâche spécifique. Les bibliothèques du framework permettent d’exécuter des modules sans avoir à écrire manuellement le code nécessaire pour les lancer.

## Types de modules

- **Exploit** — utilisé pour tirer parti d’une vulnérabilité, généralement associé à un payload.
- **Payload** — code livré et exécuté sur la cible après une exploitation réussie. Un exemple courant est un reverse shell qui se reconnecte vers l’attaquant.
- **Encoder** — utilisé pour encoder des payloads et aider à éviter la détection par antivirus.
- **NOPs** — utilisés pour garder des tailles de payload cohérentes et améliorer la stabilité à l’exécution.
- **Auxiliary** — utilisé pour des tâches comme le scan de ports et l’énumération.
- **Post** — utilisé après l’accès initial pour des tâches comme l’élévation de privilèges, la récupération d’identifiants et la persistance.

## Types de payloads

Metasploit prend en charge deux grands types de payloads :

- **Non-staged payloads** : envoyés à la cible en une seule fois avec l’exploit.
- **Staged payloads** : livrés en deux parties :
  - Le **stager** établit la connexion inversée et télécharge la seconde partie.
  - La **stage** est le composant du payload téléchargé et exécuté.

## Meterpreter

**Meterpreter** est un payload avancé et multifonction qui s’exécute en mémoire sur le système cible, ce qui le rend plus difficile à détecter. Il communique via un socket de stager et fournit un interpréteur de commandes interactif pour des tâches comme l’exécution de commandes, la navigation dans le système de fichiers, le keylogging, et plus encore.

## Structure du système de fichiers

Le système de fichiers de Metasploit est organisé selon une structure de répertoires simple.

Sous Linux, les modules Metasploit sont stockés dans :

```bash
/usr/share/metasploit-framework/modules
```

Les modules définis par l’utilisateur sont stockés dans :

```bash
~/.ms4/modules
```

## Flux de travail du pentest

Metasploit peut être utilisé pour réaliser et automatiser des tâches tout au long du cycle d’un test d’intrusion. Une bonne façon de comprendre son rôle est d’utiliser la méthodologie PTES.

Les phases principales sont :

- Collecte d’informations et énumération.
- Scan de vulnérabilités.
- Exploitation.
- Post-exploitation.
- Élévation de privilèges.
- Maintien d’un accès persistant.

## Idée clé

Metasploit est un framework modulaire et flexible qui aide à automatiser les tests d’intrusion de bout en bout. Sa force réside dans sa structure, sa bibliothèque de modules et sa capacité à combiner exploitation et post-exploitation sur une seule plateforme.
