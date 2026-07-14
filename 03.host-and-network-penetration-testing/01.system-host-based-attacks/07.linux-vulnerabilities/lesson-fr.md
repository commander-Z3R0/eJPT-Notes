# Linux Vulnerabilities

Linux est un système d'exploitation libre et open source composé du noyau Linux et de la suite d'outils GNU. Il est largement utilisé dans de nombreux environnements, mais il est principalement déployé comme système d'exploitation pour serveurs.

Pour cette raison, les serveurs Linux exécutent généralement des services et protocoles spécifiques. Ces services peuvent exposer des surfaces d'attaque que les attaquants peuvent exploiter pour accéder à un système cible.

## Pourquoi Linux est une cible

Les systèmes Linux sont ciblés par les attaquants pour plusieurs raisons :

1. **Domination des serveurs :** Une grande partie des serveurs web et des systèmes d'infrastructure fonctionne sous Linux.
2. **Exposition des services :** Les serveurs Linux exposent souvent des services réseau comme SSH, FTP et les serveurs web.
3. **Mauvaises configurations :** Des permissions incorrectes, des configurations faibles ou des services exposés peuvent introduire des vulnérabilités.
4. **Logiciels obsolètes :** Des paquets non mis à jour ou des noyaux anciens peuvent contenir des failles connues.
5. **Nature open source :** Bien que transparent, les vulnérabilités sont aussi étudiées publiquement et peuvent être exploitées rapidement.
6. **Faiblesse des identifiants :** Des mots de passe faibles ou la réutilisation de clés SSH peuvent permettre des accès non autorisés.

## Types de vulnérabilités Linux

Les vulnérabilités Linux peuvent provenir du système lui-même, des services installés ou de problèmes de configuration. Exemples courants :

- **Escalade de privilèges :** Exploitation du noyau ou de binaires SUID pour obtenir des privilèges élevés.
- **Exécution de code à distance (RCE) :** Failles dans des services permettant d’exécuter des commandes à distance.
- **Permissions mal configurées :** Permissions incorrectes sur des fichiers ou dossiers exposant des données sensibles.
- **Authentification faible :** Mauvaise configuration SSH, mots de passe faibles ou absence d’authentification par clé.
- **Logiciels non corrigés :** Vulnérabilités dans des paquets ou services non mis à jour.
- **Services non sécurisés :** Utilisation de services inutiles ou non sécurisés comme Telnet ou des serveurs FTP obsolètes.
- **Exploits du noyau :** Bugs dans le noyau Linux permettant de compromettre le système.
- **Abus des tâches cron :** Exploitation de tâches planifiées avec des permissions faibles.
- **Abus des variables d’environnement :** Manipulation de variables comme PATH ou LD_PRELOAD pour exploiter le système.

## Services Linux fréquemment exploités

| Service | Ports | Objectif |
|---|---|---|
| Apache | TCP ports 80/443 | Serveur web libre et open source multiplateforme, largement utilisé pour héberger des sites et applications web. |
| SSH | TCP port 22 | Protocole sécurisé permettant l’accès et l’administration à distance des systèmes. |
| FTP | TCP port 21 | Protocole utilisé pour le transfert de fichiers entre systèmes, souvent non sécurisé s’il est mal configuré. |
| SAMBA | TCP port 445 | Implémentation de SMB sous Linux, utilisée pour le partage de fichiers et d’imprimantes sur un réseau. |

## Idée clé

Linux est un élément essentiel de l’infrastructure moderne, en particulier dans les environnements serveurs. Son utilisation massive, combinée à des services exposés et à des mauvaises configurations potentielles, en fait une cible importante pour les attaquants. Comprendre les services courants et leurs vecteurs d’exploitation est essentiel pour les défenseurs comme pour les pentesters.
