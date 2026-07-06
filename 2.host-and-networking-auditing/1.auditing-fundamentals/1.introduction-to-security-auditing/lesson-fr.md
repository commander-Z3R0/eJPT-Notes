# Introduction to Security Auditing

L'**Audit de Sécurité** est un processus systématique d'évaluation et de vérification des mesures et contrôles de sécurité au sein d'une organisation pour s'assurer qu'ils sont **efficaces**, **appropriés** et **conformes** aux normes, politiques et réglementations pertinentes.

Cette leçon couvre les **fondamentaux de l'audit de sécurité**, notamment son importance, la terminologie clé, le processus d'audit, les types d'audits et la relation entre audit et test d'intrusion — des connaissances essentielles pour la certification eJPT.

---

## Pourquoi l'Audit de Sécurité est Important

L'audit de sécurité aide les organisations à :

- **Identifier les vulnérabilités et faiblesses** dans les systèmes, processus et infrastructures avant que les attaquants ne le fassent.
- **Garantir la conformité** avec les normes réglementaires (RGPD, HIPAA, PCI DSS, ISO 27001).
- **Améliorer la gestion des risques** en priorisant les menaces selon leur impact potentiel.
- **Améliorer les politiques de sécurité** en identifiant les lacunes et les axes d'amélioration.
- **Soutenir les objectifs métier** en protégeant les opérations et en renforçant la confiance des clients.
- **Permettre l'amélioration continue** en maintenant les mesures de sécurité à jour face aux menaces émergentes.

Contrairement aux tests d'intrusion (qui se concentrent sur l'exploitation des vulnérabilités techniques), l'audit de sécurité commence au **niveau fondamental** — en évaluant les processus organisationnels, le respect des politiques par les employés et les procédures opérationnelles.

---

## Terminologie Essentielle

| Terme | Définition |
|-------|-----------|
| **Politique de Sécurité** | Document formel définissant les objectifs, directives et procédures de sécurité d'une organisation pour protéger ses actifs informationnels |
| **Conformité** | Adhésion aux exigences réglementaires, normes sectorielles et politiques internes liées à la sécurité et à la protection des données |
| **Vulnérabilité** | Faiblesse dans un système ou processus pouvant être exploitée pour obtenir un accès non autorisé ou causer des dommages |
| **Contrôle** | Mesure de protection mise en œuvre pour atténuer les risques et protéger les actifs informationnels |
| **Évaluation des Risques** | Processus d'identification, d'analyse et d'évaluation des risques pour les actifs informationnels d'une organisation |
| **Piste d'Audit** | Enregistrement chronologique des événements fournissant la preuve des actions effectuées dans un système |
| **Audit de Conformité** | Examen de l'adhésion d'une organisation aux exigences réglementaires et normes sectorielles |
| **Contrôle d'Accès** | Mesures régulant qui peut accéder à des systèmes ou données spécifiques et quelles actions il peut effectuer |
| **Rapport d'Audit** | Document formel présentant les conclusions, résultats et recommandations issus d'un audit de sécurité |

---

## Normes de Conformité à Connaître

| Norme | Nom Complet | À Qui Elle S'applique |
|-------|------------|----------------------|
| **RGPD** | Règlement Général sur la Protection des Données | Toute org traitant des données de citoyens de l'UE |
| **HIPAA** | Health Insurance Portability and Accountability Act | Organisations de santé (États-Unis) |
| **PCI DSS** | Payment Card Industry Data Security Standard | Orgs traitant des paiements par carte |
| **ISO 27001** | Système de Management de la Sécurité de l'Information | Gestion générale de la sécurité de l'information |

Le non-respect peut entraîner de **graves sanctions juridiques et financières**, notamment à la suite de violations de données.

---

## Le Processus d'Audit

### 1. Planification et Préparation
- Définir le **périmètre et les objectifs** de l'audit.
- Rassembler la documentation pertinente : politiques, schémas réseau, rapports d'audits précédents.
- Constituer l'équipe d'audit et établir un calendrier.

### 2. Collecte d'Informations
- Examiner les **politiques, procédures et normes** de sécurité existantes.
- Mener des **entretiens** avec le personnel clé pour identifier les lacunes potentielles.
- Collecter des données techniques sur les configurations système et l'architecture réseau.

### 3. Évaluation des Risques
- **Identifier les actifs critiques** et les menaces potentielles qui les ciblent.
- **Évaluer les vulnérabilités** dans les systèmes et processus existants.
- **Attribuer des niveaux de risque** selon la probabilité et l'impact potentiel.

### 4. Exécution de l'Audit
- Effectuer des **tests techniques** : scans de vulnérabilités, revues de configuration, tests d'intrusion.
- **Vérifier la conformité** avec les réglementations et normes pertinentes.
- **Évaluer les contrôles** — mesurer l'efficacité des mesures de sécurité existantes.

### 5. Analyse et Évaluation
- **Analyser les résultats** pour identifier les faiblesses de sécurité et les axes d'amélioration.
- **Comparer aux normes** et aux meilleures pratiques du secteur.
- **Prioriser les problèmes** par gravité et impact potentiel sur l'activité.

### 6. Rapport
- **Documenter les résultats** : vulnérabilités, non-conformités, contrôles inefficaces.
- **Fournir des recommandations actionnables** pour traiter chaque problème identifié.
- **Présenter les résultats** aux parties prenantes et discuter des conclusions clés.

### 7. Remédiation et Suivi
- **Élaborer un plan de remédiation** basé sur les résultats de l'audit.
- **Mettre en œuvre les changements** et améliorations.
- **Planifier des audits de suivi** pour vérifier l'efficacité de la remédiation.
- **Surveiller en continu** et mettre à jour les mesures de sécurité à mesure que les menaces évoluent.

---

## Types d'Audits

| Type d'Audit | Qui le Réalise | Périmètre | Pertinence pour le Pentest |
|--------------|---------------|-----------|---------------------------|
| **Interne** | Équipe d'audit interne | Contrôles internes, respect des politiques | Référence d'auto-évaluation ; met en évidence les zones nécessitant des tests approfondis |
| **Externe** | Auditeurs tiers indépendants | Évaluation impartiale, benchmarks de conformité | Les résultats guident le périmètre du pentest |
| **Conformité** | Interne ou externe | Adhésion réglementaire (RGPD, HIPAA, PCI DSS) | Identifie les lacunes réglementaires pour des tests ciblés |
| **Technique** | Spécialistes en sécurité | Infrastructure IT, matériel, logiciels, configs | Révèle les contrôles techniques où le pentest peut découvrir des vulns |
| **Réseau** | Équipe sécurité réseau | Routeurs, switches, firewalls, équipements réseau | Découvre les failles de conception réseau pour les tests d'exploitation |
| **Application** | Équipe AppSec | Qualité du code, validation des entrées, authentification, gestion des données | Met en évidence les failles applicatives (SQLi, XSS) pour les scénarios de pentest |

---

## Audit de Sécurité vs. Test d'Intrusion

Ce sont deux **types différents** d'évaluations de sécurité. Comprendre la distinction est essentiel pour l'eJPT.

| Aspect | Audit de Sécurité | Test d'Intrusion |
|--------|------------------|-----------------|
| **Objectif** | Évaluer la posture de sécurité globale et la conformité | Simuler des attaques réelles et exploiter des vulnérabilités |
| **Périmètre** | Global : politiques, procédures, contrôles, conformité | Spécifique : systèmes, réseaux ou applications définis |
| **Méthodologie** | Revue documentaire, entretiens, évaluation technique | Exploitation active avec outils et techniques |
| **Résultat** | Lacunes dans les politiques et contrôles ; statut de conformité | Rapport détaillé de vulnérabilités avec vecteurs d'attaque |
| **Fréquence** | Calendrier régulier (annuel/semestriel) ou selon exigences | Selon les besoins : après modifications système, selon un calendrier, ou post-audit |

### Approche Séquentielle (La Plus Courante)

Le flux typique est :

```
Audit de Sécurité → Résultats → Test d'Intrusion → Remédiation → Audit de Suivi
```

1. L'organisation réalise un **audit de sécurité** pour évaluer la posture globale et la conformité.
2. Les résultats de l'audit définissent le **périmètre et la focale** du test d'intrusion.
3. Le test d'intrusion **valide et confirme** les vulnérabilités techniques identifiées lors de l'audit.
4. La remédiation est appliquée et un **audit de suivi** vérifie les corrections.

**Avantages** :
- Vision globale depuis des perspectives politiques et techniques.
- Traite les lacunes dans les contrôles procéduraux et techniques.
- Aide à prioriser les efforts de remédiation sur la base du risque réel.

### Approche Combinée

Certaines organisations exécutent les deux simultanément dans une **évaluation de sécurité holistique**.

**Avantages** :
- Rationalise le processus en combinant les évaluations politiques, procédurales et techniques.
- Offre une image plus complète de la posture de sécurité en un seul engagement.
- Plus efficace et rentable.

---

## Points Clés

- L'**audit de sécurité** évalue la posture de sécurité globale d'une organisation — politiques, procédures, contrôles et conformité.
- Ce n'est **pas la même chose qu'un test d'intrusion** : l'audit se concentre sur les processus et la conformité ; le pentest sur l'exploitation technique.
- Le **processus d'audit en 7 étapes** : Planification → Collecte d'Informations → Évaluation des Risques → Exécution → Analyse → Rapport → Remédiation.
- **Normes de conformité** à connaître : RGPD, HIPAA, PCI DSS, ISO 27001.
- **Types d'audits** : Interne, Externe, Conformité, Technique, Réseau, Application.
- Les audits précèdent généralement les tests d'intrusion — les résultats de l'audit définissent le périmètre du pentest.
- L'audit de sécurité est un **processus continu et permanent** — pas un événement ponctuel.
- Une posture de sécurité solide construite grâce à des audits réguliers **soutient à la fois la conformité et les objectifs métier**.
