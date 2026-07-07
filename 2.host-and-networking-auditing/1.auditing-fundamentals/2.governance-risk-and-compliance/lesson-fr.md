# Governance, Risk & Compliance (GRC)

**GRC** est l'acronyme de **Governance, Risk, and Compliance**. C'est un cadre global utilisé par les organisations pour gérer et aligner leurs pratiques de gouvernance, leurs stratégies de gestion des risques et leur conformité aux exigences réglementaires — contribuant à maintenir la **transparence**, la **responsabilité** et la **résilience**.

Cette leçon couvre les **trois piliers du GRC**, pourquoi il est pertinent pour les pentesters, et les **frameworks, normes et directives** clés à connaître pour la certification eJPT.

---

## Les Trois Piliers du GRC

| Pilier | Définition |
|--------|-----------|
| **Governance** | Le cadre de politiques, procédures et pratiques qui garantit qu'une organisation atteint ses objectifs, gère ses risques et respecte les exigences légales. Inclut le développement des politiques, la définition des rôles et responsabilités, et l'établissement de mécanismes de responsabilisation. |
| **Risk** | Le processus d'identification, d'évaluation et d'atténuation des risques pouvant impacter négativement les actifs et opérations d'une organisation. |
| **Compliance** | S'assurer qu'une organisation respecte les lois, réglementations et normes sectorielles pertinentes — tant externes (RGPD, HIPAA, PCI DSS) qu'internes (politiques de sécurité). Inclut la réalisation de revues régulières pour vérifier la conformité continue. |

---

## Pourquoi le GRC est Important pour les Pentesters

Le GRC est directement pertinent pour les pentesters car il permet de :

- Réaliser des **évaluations plus approfondies et pertinentes** en comprenant le contexte de gouvernance de l'organisation.
- **Contextualiser les résultats** dans le cadre des politiques organisationnelles, de la gestion des risques et des exigences de conformité.
- Fournir des **recommandations plus stratégiques** alignées sur le framework GRC de l'organisation.
- Aider à renforcer la **posture de sécurité globale** au-delà des seules vulnérabilités techniques.

Lorsque vous comprenez le framework GRC d'une entreprise, votre rapport de pentest devient bien plus actionnable et aligné avec ce qui importe réellement aux parties prenantes.

---

## Frameworks vs. Normes vs. Directives

Avant d'examiner les frameworks spécifiques, il est important de comprendre la distinction :

| Type | Description | Obligatoire ? |
|------|-------------|--------------|
| **Framework** | Approche structurée pour mettre en œuvre des pratiques de sécurité ; flexible et adaptable à diverses organisations et secteurs | Généralement Non |
| **Norme** | Exigences et critères spécifiques devant être respectés pour atteindre la conformité | Souvent Oui (secteurs réglementés) |
| **Directive** | Pratiques recommandées et conseils ; non contraignants | Non |

---

## Frameworks, Normes et Directives Clés

### NIST CSF (Cybersecurity Framework)

Développé par le **National Institute of Standards and Technology**, le NIST CSF fournit des lignes directrices et des bonnes pratiques pour aider les organisations à gérer et réduire les risques de cybersécurité.

**Fonctions Principales** :

| Fonction | Objectif |
|----------|---------|
| **Identify** | Comprendre les actifs, risques et vulnérabilités |
| **Protect** | Mettre en œuvre des protections pour assurer la continuité des services |
| **Detect** | Identifier la survenance d'événements de cybersécurité |
| **Respond** | Agir face aux incidents de cybersécurité détectés |
| **Recover** | Restaurer les capacités après un incident de cybersécurité |

---

### COBIT (Control Objectives for Information & Related Technologies)

Un framework pour **développer, implémenter, surveiller et améliorer** les pratiques de gouvernance et de gestion IT.

**Axes principaux** :
- Aligner les objectifs IT sur les **objectifs métier**.
- Gérer efficacement les **risques IT**.
- Garantir la **conformité** aux réglementations.

---

### ISO/IEC 27001

Une **norme internationale** pour les Systèmes de Management de la Sécurité de l'Information (SMSI/ISMS). Elle définit les bonnes pratiques pour gérer et protéger les informations sensibles.

**Axes principaux** :
- Établir, implémenter, maintenir et **améliorer continuellement** un ISMS.
- Applicable à **toute organisation**, quelle que soit sa taille ou son secteur.
- La certification démontre un engagement formel envers la sécurité de l'information.

---

### PCI DSS (Payment Card Industry Data Security Standard)

Un ensemble de normes de sécurité conçues pour **protéger les données de paiement par carte** et garantir le traitement sécurisé des transactions par carte de crédit.

**Axes principaux** :
- Protéger les **données du titulaire de la carte**.
- Maintenir un **réseau sécurisé**.
- Mettre en œuvre des mesures robustes de **contrôle d'accès**.

**Obligatoire pour** : Toute organisation traitant des transactions par carte de crédit.

---

### HIPAA (Health Insurance Portability and Accountability Act)

Une **loi américaine** établissant des normes pour protéger les informations sensibles des patients et garantir la confidentialité et la sécurité des données de santé.

**Axes principaux** :
- Protéger les **Informations de Santé Protégées (PHI)**.
- Normes de sécurité et de confidentialité pour le traitement des données de santé.

**Obligatoire pour** : Prestataires de santé américains, régimes d'assurance maladie et toute entité gérant des PHI.

---

### GDPR — General Data Protection Regulation (RGPD)

Un **règlement européen** régissant la protection des données et la vie privée des personnes au sein de l'UE et de l'EEE (Espace Économique Européen).

**Axes principaux** :
- **Principes de protection des données** et droits des personnes concernées.
- Obligations pour les **responsables du traitement** et les **sous-traitants**.
- Consentement de l'utilisateur, droit d'accès, droit à l'effacement ("droit à l'oubli").

**Obligatoire pour** : Toute organisation traitant des données personnelles de personnes dans l'UE/EEE — quelle que soit la localisation de l'organisation.

---

### CIS Controls (Center for Internet Security Controls)

Un ensemble de **bonnes pratiques prioritaires et actionnables** pour aider les organisations à améliorer leur posture de cybersécurité. Organisées en Groupes d'Implémentation (IG1, IG2, IG3) selon la maturité et les ressources de l'organisation.

**Axes principaux** :
- Améliorations de sécurité pratiques et progressives.
- Largement utilisées comme **référence pour le durcissement de sécurité**.

---

### NIST SP 800-53

Une publication du NIST fournissant un **catalogue exhaustif de contrôles de sécurité et de confidentialité** pour les systèmes d'information fédéraux et les organisations.

**Axes principaux** :
- Contrôles de sécurité pour les **systèmes d'information fédéraux**.
- Couvre la gestion des risques, le contrôle d'accès, la réponse aux incidents et plus encore.

**Obligatoire pour** : Agences fédérales américaines et organisations gérant des données fédérales.

---

## Comparatif des Frameworks et Normes

| Framework/Norme | Type | Obligatoire ? | Axe Principal | À Qui S'applique |
|----------------|------|--------------|---------------|-----------------|
| **NIST CSF** | Framework | Non | Identify, Protect, Detect, Respond, Recover | Toute organisation |
| **COBIT** | Framework | Non | Gouvernance IT alignée sur les objectifs métier | Organisations IT |
| **ISO/IEC 27001** | Norme | Non (certification volontaire) | Système de Management de la Sécurité de l'Information | Toute organisation |
| **PCI DSS** | Norme | Oui | Protéger les données de paiement par carte | Orgs traitant des paiements par carte |
| **HIPAA** | Norme (Loi) | Oui | Protéger les informations de santé des patients | Entités de santé américaines |
| **GDPR/RGPD** | Norme (Règlement) | Oui | Confidentialité et protection des données | Orgs traitant des données personnelles UE/EEE |
| **CIS Controls** | Directives | Non | Bonnes pratiques de sécurité actionnables | Toute organisation |
| **NIST SP 800-53** | Norme | Oui (fédéral) | Catalogue de contrôles de sécurité | Agences fédérales américaines |

---

## Points Clés

- **GRC** est l'acronyme de Governance, Risk, and Compliance — un cadre pour aligner les pratiques de sécurité sur les objectifs organisationnels et les exigences réglementaires.
- **Governance** établit les politiques et la responsabilisation ; **Risk** identifie et atténue les menaces ; **Compliance** garantit l'adhésion aux lois et normes.
- Pour les pentesters, comprendre le GRC aide à contextualiser les résultats stratégiquement et à fournir des recommandations pertinentes pour les parties prenantes.
- Les **frameworks** (NIST CSF, COBIT) sont flexibles et adaptables — non obligatoires.
- Les **normes** (PCI DSS, HIPAA, GDPR, ISO 27001) définissent des exigences spécifiques — souvent légalement obligatoires.
- Fonctions principales du **NIST CSF** : Identify → Protect → Detect → Respond → Recover.
- **PCI DSS** — obligatoire pour les organisations traitant des paiements par carte.
- **HIPAA** — obligatoire pour les entités de santé américaines gérant des PHI.
- **GDPR/RGPD** — obligatoire pour toute organisation traitant des données personnelles UE/EEE, quelle que soit sa localisation.
- **ISO 27001** — norme internationale pour ISMS ; volontaire mais largement reconnue.
- **CIS Controls** — étapes pratiques et prioritaires pour le durcissement de sécurité ; excellent point de départ pour toute organisation.
- **NIST SP 800-53** — obligatoire pour les agences fédérales américaines ; catalogue exhaustif de contrôles de sécurité.
