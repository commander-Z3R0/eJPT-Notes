# From Auditing to Penetration Testing

Cette leçon démontre comment les **audits de sécurité et les tests d'intrusion fonctionnent ensemble** en pratique. À travers un scénario réel, nous parcourons le développement d'une politique de sécurité, la réalisation d'un audit système avec Lynis et la validation des remédiations par un test d'intrusion — montrant comment chaque phase informe et s'appuie sur la précédente.

---

## Scénario Pratique : SecureTech Solutions

**Entreprise** : SecureTech Solutions
**Spécialisation** : Sécurisation de l'infrastructure IT pour divers clients.

**Objectif** : Développer une politique de sécurité pour les serveurs Linux, réaliser une évaluation des risques avec le framework **NIST SP 800-53**, effectuer un audit de sécurité et valider les remédiations avec un test d'intrusion.

---

> **Note** : Toutes les commandes de cette leçon supposent que vous opérez en tant que **root** directement sur le serveur Linux cible. Si vous utilisez un utilisateur non-root, ajoutez `sudo` devant chaque commande. Dans les environnements de laboratoire (INE/eLabs), l'accès root est fourni par défaut.

---

## Le Flux de Travail en Trois Phases

```
Phase 1: Develop Security Policy → Phase 2: Audit with Lynis → Phase 3: Penetration Test
```

Chaque phase alimente directement la suivante — la politique définit ce qu'il faut auditer, les résultats de l'audit définissent ce qu'il faut remédier, et le test d'intrusion valide que les remédiations fonctionnent réellement.

---

## Phase 1 — Develop a Security Policy

### Objectif

Établir une **politique de sécurité de référence pour les serveurs Linux** alignée sur les directives du NIST SP 800-53, garantissant que les serveurs sont configurés et gérés de manière sécurisée — protégés contre les accès non autorisés, les vulnérabilités et autres menaces de sécurité.

### Structure de la Politique (Alignée sur NIST SP 800-53)

| Section | Objectif |
|---------|---------|
| **1. Purpose & Scope** | Définir les objectifs de la politique et les systèmes/actifs couverts |
| **2. Access Control** | Règles sur qui peut accéder au serveur et dans quelles conditions |
| **3. Audit & Accountability** | Exigences de journalisation, gestion des pistes d'audit et procédures de révision |
| **4. Configuration Management** | Configurations de référence, contrôle des changements et standards de durcissement |
| **5. Identification & Authentication** | Politiques de mots de passe, exigences MFA et gestion des comptes utilisateurs |
| **6. System & Information Integrity** | Gestion des correctifs, protection contre les malwares et surveillance de l'intégrité |
| **7. Maintenance** | Procédures de maintenance planifiée et personnel de maintenance autorisé |

> Documentation complète NIST SP 800-53 : https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final

### Pourquoi Cette Phase est Importante

Sans politique de sécurité définie, il n'y a **pas de référence à auditer**. La politique établit ce que signifie "sécurisé" pour cette organisation — tout ce qui est fait dans les Phases 2 et 3 est mesuré par rapport à elle.

---

## Phase 2 — Auditing with Lynis

### Objectif

Réaliser un audit de sécurité sur le serveur Linux avec **Lynis**, identifier les vulnérabilités et les remédier — comme mettre à jour les logiciels et renforcer les politiques de mots de passe — pour mettre le système en conformité avec la politique de la Phase 1.

### Qu'est-ce que Lynis ?

**Lynis** est un outil d'audit de sécurité open-source pour les systèmes Unix/Linux. Il effectue un **scan de santé** du système pour soutenir :
- Le **durcissement du système**
- Les **tests de conformité**
- L'**identification des vulnérabilités**

> Documentation Lynis : https://cisofy.com/lynis/

**Important** : Exécutez toutes les commandes Lynis directement sur le **serveur Linux audité** — pas depuis votre machine Kali ni depuis une VM.

### Installation

```bash
# Télécharger Lynis
wget https://downloads.cisofy.com/lynis/lynis-3.1.4.tar.gz

# Décompresser
gzip -d lynis-3.1.4.tar.gz

# Extraire
tar -xf lynis-3.1.4.tar

# Entrer dans le répertoire et définir les permissions
cd lynis/
chmod +x lynis
```

### Exécuter l'Audit

```bash
./lynis audit system
```

Lynis analysera le système et produira un rapport détaillé couvrant :
- **Suggestions de durcissement du système**
- **Vulnérabilités et mauvaises configurations détectées**
- **Statut de conformité par rapport aux benchmarks de sécurité**
- **Avertissements et recommandations** priorisés par gravité

### Interprétation des Résultats de l'Audit

Après le scan, contextualisez les résultats par rapport à la politique de sécurité de l'organisation :

| Sortie Lynis | Action |
|-------------|--------|
| **Warnings** | Problèmes haute priorité — remédier immédiatement |
| **Suggestions** | Améliorations basse priorité — planifier pour remédiation |
| **Hardening Index** | Score global ; plus il est bas, plus le durcissement est nécessaire |

### Exemples de Remédiation

Basés sur des résultats Lynis typiques face à une politique NIST SP 800-53 :

```bash
# Mettre à jour tous les paquets (System & Information Integrity)
apt update && apt upgrade -y

# Renforcer les politiques de mots de passe (Identification & Authentication)
# Éditer /etc/login.defs pour définir :
PASS_MAX_DAYS   90
PASS_MIN_DAYS   7
PASS_WARN_AGE   14

# Désactiver les services inutilisés (Configuration Management)
systemctl disable <service_inutilise>
systemctl stop <service_inutilise>

# Activer et configurer auditd (Audit & Accountability)
apt install auditd -y
systemctl enable auditd
systemctl start auditd
```

### Pourquoi Cette Phase est Importante

L'audit établit **ce qui ne va pas réellement** sur le système par rapport à la politique de référence. Les remédiations sont appliquées ici — mais l'audit seul ne peut pas confirmer qu'elles fonctionnent. C'est le rôle de la Phase 3.

---

## Phase 3 — Penetration Test

### Objectif

**Valider l'efficacité des remédiations** de la Phase 2 en testant activement le serveur Linux — confirmant que les vulnérabilités ont été corrigées et identifiant toute faiblesse résiduelle ou précédemment inconnue.

### Ce que Fait le Penetration Test

| Tâche | Objectif |
|-------|---------|
| **Comparer les résultats de l'audit initial vs. les résultats du pentest** | Vérifier que les vulnérabilités remédiées ne sont plus exploitables |
| **Tester les vulnérabilités résiduelles** | Vérifier si des problèmes persistent après remédiation |
| **Découvrir de nouvelles vulnérabilités** | Identifier des faiblesses non détectées par l'audit Lynis |
| **Valider la conformité** | Confirmer que le serveur respecte désormais les exigences de la politique de sécurité |

### Flux de Travail du Pentest pour ce Scénario

```
Reconnaissance → Scanning → Enumeration → Exploitation Attempts → Post-Exploitation → Reporting
```

1. **Reconnaissance** : Collecter des informations sur le serveur Linux (version OS, ports ouverts, services en cours d'exécution).
2. **Scanning** : Utiliser Nmap pour identifier les ports ouverts et les versions de services.

```bash
nmap -sV -sC -p- <target_ip>
```

3. **Enumeration** : Énumérer les services à la recherche de mauvaises configurations, d'identifiants faibles et de CVEs connus.
4. **Exploitation Attempts** : Tenter d'exploiter les vulnérabilités signalées par Lynis — confirmant si les remédiations ont tenu.
5. **Post-Exploitation** : Si un accès est obtenu, évaluer ce qu'un attaquant pourrait faire (élévation de privilèges, mouvement latéral, accès aux données).
6. **Reporting** : Documenter tous les résultats avec des niveaux de gravité et des recommandations de remédiation.

### Le Rapport de Pentest

Le rapport final est le livrable qui unit les trois phases :

| Section du Rapport | Contenu |
|-------------------|---------|
| **Executive Summary** | Vue d'ensemble de haut niveau des résultats pour la direction |
| **Audit vs. Pentest Comparison** | Comparaison côte à côte des résultats Lynis vs. résultats du pentest |
| **Confirmed Remediations** | Vulnérabilités de la Phase 2 désormais corrigées |
| **Residual Vulnerabilities** | Problèmes qui n'ont pas été entièrement remédiés |
| **New Findings** | Vulnérabilités découvertes uniquement lors du pentesting |
| **Severity Ratings** | Critical / High / Medium / Low par résultat |
| **Recommendations** | Étapes actionnables pour traiter chaque problème restant |

### Pourquoi Cette Phase est Importante

La remédiation sans validation est une **sécurité incomplète**. Un test d'intrusion fournit une confirmation active et adversariale que les corrections fonctionnent réellement — et fait remonter tout ce qui a échappé à la phase d'audit.

---

## Résumé du Flux de Travail Complet

| Phase | Outil/Méthode | Résultat |
|-------|--------------|---------|
| **1. Security Policy** | Framework NIST SP 800-53 | Politique de référence documentée pour les serveurs Linux |
| **2. Security Audit** | Lynis | Liste de vulnérabilités et mauvaises configurations ; remédiations appliquées |
| **3. Penetration Test** | Nmap, Metasploit, tests manuels | Rapport de validation confirmant l'efficacité des remédiations et les nouveaux résultats |

---

## Points Clés

- L'audit de sécurité et le test d'intrusion sont **complémentaires, non interchangeables** — chacun a un objectif distinct.
- La **politique de sécurité** (Phase 1) est le fondement — sans elle, il n'y a pas de référence à auditer ou à tester.
- **Lynis** est un puissant outil open-source pour l'audit et le durcissement des systèmes Linux — exécutez-le toujours directement sur le serveur cible, pas depuis Kali.
- La commande `./lynis audit system` effectue un scan de santé complet produisant des warnings, suggestions et un hardening index.
- Les **remédiations** doivent être appliquées après l'audit avant que le test d'intrusion ne commence.
- Le **test d'intrusion** (Phase 3) valide que les remédiations fonctionnent réellement et découvre les vulnérabilités résiduelles ou nouvelles.
- Le **rapport final** compare les résultats de l'audit aux résultats du pentest — montrant clairement ce qui a été corrigé, ce qui reste et ce qui a été nouvellement découvert.
- Les résultats du rapport sont **classés par gravité** (Critical / High / Medium / Low) pour aider à prioriser les efforts de remédiation.
- Ce flux de travail en trois phases correspond directement aux engagements réels et constitue le fondement du **penetration testing orienté conformité**.
