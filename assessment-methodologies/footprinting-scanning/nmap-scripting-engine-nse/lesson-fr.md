# Nmap Scripting Engine (NSE)

Le **Nmap Scripting Engine (NSE)** est l'une des fonctionnalités les plus puissantes de Nmap. Il te permet d'exécuter des **scripts** contre des hôtes cibles pour effectuer une **enumération avancée de services**, une **détection de vulnérabilités** et une **collecte d'informations** de manière automatisée et personnalisable.

## Qu'est-ce que NSE ?

NSE utilise des **scripts Lua** pour étendre les capacités de Nmap au-delà du simple scan de ports. Chaque script est conçu pour effectuer une tâche spécifique, comme :

- Énumérer des **utilisateurs** (FTP, SMTP, SMB, SSH).
- Détecter les **versions de services** et configurations.
- Trouver des **vulnérabilités** (CVEs, malconfigurations).
- Collecter des structures de **dossiers web**.
- Extraire des **enregistrements DNS** et informations réseau.

Les scripts NSE sont stockés dans `/usr/share/nmap/scripts/` sur la plupart des systèmes Linux et sont catégorisés par fonction (default, discovery, vuln, brute, auth, etc.).

---

## Exécuter des Scripts NSE

### Scripts par Défaut (`-sC`)
L'option `-sC` exécute la **catégorie par défaut de scripts NSE**, qui inclut des scripts sûrs et couramment utiles pour l'énumération de base :

```bash
nmap -sS -sC 192.168.1.10
```

### Scripts Spécifiques (`--script`)
Tu peux exécuter des scripts ou catégories de scripts spécifiques par nom :

```bash
nmap -sS --script http-enum 192.168.1.10
nmap -sS --script vuln 192.168.1.10
nmap -sS --script ftp-anon,ftp-bounce 192.168.1.10
nmap -sS --script smb-enum-shares,smb-enum-users 192.168.1.10
```

### Catégories de Scripts
Les scripts NSE sont organisés en catégories :

| Catégorie | Objectif |
|-----------|----------|
| **default** | Scripts sûrs activés avec `-sC` |
| **discovery** | Découverte de services et hôtes |
| **vuln** | Détection de vulnérabilités |
| **auth** | Tests d'authentification |
| **brute** | Attaques par force brute |
| **http** | Énumération 
