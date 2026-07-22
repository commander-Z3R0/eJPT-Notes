# The Metasploit Framework

## Installation et configuration

Avant d’utiliser Metasploit, il faut d’abord l’installer et vérifier que la base de données fonctionne correctement. Metasploit est distribué par Rapid7 et peut être installé sur Windows et Linux, mais dans ce cours nous utilisons Kali Linux, car Metasploit et ses dépendances sont déjà inclus.

Metasploit utilise une base de données pour stocker les hôtes, les scans, les services et les autres données d’évaluation. PostgreSQL est la base de données principale, donc le service PostgreSQL doit être démarré avant d’initialiser la base de données de Metasploit.

### Étapes d’installation

```bash
sudo apt update && sudo apt upgrade -y
sudo systemctl start postgresql
sudo systemctl enable postgresql
msfdb init
msfconsole
```

Pour vérifier la connexion à la base de données dans `msfconsole`, utilise :

```bash
db_status
```

Si tout est correctement configuré, Metasploit indiquera que PostgreSQL est connecté. La base de données permet aussi d’importer des résultats d’outils comme Nmap et Nessus.

## Notions de base de MSFconsole

`msfconsole` est l’interface principale tout-en-un de Metasploit. C’est l’outil que tu utiliseras le plus pendant le cours, car il donne accès à toutes les fonctionnalités du framework depuis une seule console.

### Ce qu’il faut savoir

- Comment chercher des modules.
- Comment sélectionner des modules.
- Comment configurer les options et variables des modules.
- Comment rechercher des payloads.
- Comment gérer les sessions.
- Comment utiliser des fonctions supplémentaires.
- Comment enregistrer la configuration.

### Commandes utiles

Rechercher un module :

```bash
search <keyword>
```

Sélectionner un module :

```bash
use <module_path>
```

Afficher les options du module :

```bash
show options
```

Afficher les payloads :

```bash
show payloads
```

Lancer un module :

```bash
run
```

ou

```bash
exploit
```

### Variables de module

De nombreux modules nécessitent des valeurs comme l’IP cible, les ports et les adresses de callback. Ces valeurs sont configurées à l’aide de variables dans `msfconsole`.

| Variable | Rôle |
|---|---|
| `LHOST` | Adresse IP de l’attaquant. |
| `LPORT` | Port sur la machine attaquante utilisé pour recevoir la connexion inversée. |
| `RHOST` | Adresse IP de la cible. |
| `RHOSTS` | Plusieurs IPs cibles ou une plage réseau. |
| `RPORT` | Port de la cible. |

Exemple :

```bash
set RHOSTS 10.10.10.10
set LHOST 10.10.14.2
set LPORT 4444
```

Tu peux définir ces valeurs localement pour un seul module ou globalement pour toute la session.

## Créer et gérer les workspaces

Les workspaces permettent d’organiser les hôtes, les scans et les activités pendant une évaluation. Ils sont utiles pour séparer les données par cible ou par client afin de garder un travail propre et structuré.

### Commandes de workspace

Lister les workspaces :

```bash
workspace
```

Créer un nouveau workspace :

```bash
workspace -a client1
```

Basculer vers un workspace :

```bash
workspace client1
```

Supprimer un workspace :

```bash
workspace -d client1
```

### Pourquoi les workspaces sont importants

- Ils séparent les évaluations.
- Ils facilitent le suivi des hôtes et des résultats de scan.
- Ils aident à organiser le travail lors d’engagements plus longs.

## Idée clé

La première étape avec Metasploit consiste à préparer l’environnement : PostgreSQL doit être en cours d’exécution, `msfdb` doit être initialisé, et `msfconsole` est l’endroit où tu gères les modules, les variables, les sessions et les workspaces. Une fois cela en place, tu peux utiliser Metasploit efficacement pour le reste du cours.
