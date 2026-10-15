# Cadrage : personas et exigences non fonctionnelles

Premier document de conception de RééducAssist : pour qui on construit, et les contraintes qui ne sont pas des fonctionnalités.

- **Ticket** : RA-010
- **Sources** : pages Notion « User stories (v1) » (validées le 2 octobre 2026), « Contexte et décisions », « Maquettes : prompt Figma et plan d'intégration » et « RGPD et concurrence ». Les seuils absents de ces sources ont été fixés par le développeur le 15 octobre 2026.
- **Statut** : à valider

> Démonstrateur technique. Données entièrement synthétiques et documents publics. Ne constitue pas un dispositif médical.

## 1. Le produit en une phrase

Un assistant IA pour les praticiens en rééducation (kinésithérapeutes, ergothérapeutes) en France : il suit les patients, répond aux questions en citant des recommandations publiques et propose des programmes d'exercices que le praticien valide toujours.

## 2. Personas

Les personas ci-dessous sont construits à partir des user stories et des maquettes. **Ils n'ont pas encore été confrontés à un praticien réel** : ce sont des hypothèses, à valider dès qu'un entretien est possible (voir la question ouverte Q1).

### 2.1 Claire, kinésithérapeute en cabinet de groupe (rôle Praticien)

- **Contexte d'usage** : enchaîne les patients toute la journée, utilise l'application sur un ordinateur de bureau ou une tablette entre deux séances, souvent pendant quelques minutes seulement.
- **Ce qu'elle fait** : crée la fiche d'un patient (motif de prise en charge, objectifs), saisit chaque séance (exercices réalisés, douleur sur 10, notes), suit l'évolution, pose des questions à l'assistant et valide ou refuse les programmes proposés.
- **Besoins** :
  - retrouver le contexte d'un patient en un coup d'œil (US3) ;
  - vérifier rapidement une information avant de l'appliquer, avec sa source (US8, US9) ;
  - gagner du temps de préparation et de rédaction (US10, US12) ;
  - assurer la continuité des soins lors d'un remplacement (US19).
- **Frustrations** :
  - le temps administratif pris sur le temps de soin ;
  - les outils qui demandent trop de clics ou de saisie ;
  - une IA qui affirme sans dire d'où vient l'information, ou qui décide à sa place.
- **Ce qui la fait renoncer** : une réponse sans source, une lenteur visible, une interface peu lisible.

### 2.2 Marc, administrateur de la plateforme (rôle Administrateur)

- **Contexte d'usage** : gère l'outil pour le cabinet ou le réseau, sur ordinateur de bureau, quelques fois par semaine.
- **Ce qu'il fait** : crée et désactive les comptes (US1), alimente la base documentaire (US6, US7), tient la bibliothèque d'exercices (US20), traite les demandes RGPD (US16) et surveille le coût et la qualité de l'IA (US13, US14).
- **Besoins** :
  - contrôler qui accède à l'application, sans jamais connaître les mots de passe ;
  - garantir que l'assistant s'appuie sur des documents à jour ;
  - répondre à une demande d'accès ou d'effacement d'un patient de façon complète et prouvée ;
  - maîtriser le budget des appels au LLM.
- **Frustrations** :
  - ne pas savoir si un document a bien été indexé ;
  - ne pas pouvoir prouver qu'un effacement est complet ;
  - découvrir un dépassement de coût après coup.

## 3. User stories et priorités (MoSCoW)

21 user stories validées le 2 octobre 2026. Le détail et les critères d'acceptation sont dans la page Notion « User stories (v1) ».

| Domaine | Must | Should | Could |
|---|---|---|---|
| Identité et accès | US1 gérer les comptes ; US2 se connecter | US21 réinitialiser son mot de passe | |
| Suivi patient | US3 fiche patient ; US4 enregistrer une séance | US5 visualiser l'évolution | US19 partager un patient |
| Base documentaire | US6 importer des documents | US7 retirer un document | |
| Bibliothèque d'exercices | US20 gérer la bibliothèque | | |
| Assistant IA | US8 question avec sources ; US9 question dans le contexte d'un patient ; US10 proposition de programme ; US11 valider ou refuser | US12 compte rendu de séance ; US17 historique des questions ; US18 export du compte rendu en PDF | |
| Pilotage de l'IA | | US13 coûts et performance | US14 qualité de l'assistant ; US15 avis sur une réponse |
| Droits des patients (RGPD) | US16 exporter et effacer les données d'un patient | | |
| **Total** | **11** | **7** | **3** |

**Hors périmètre de la v1 (Won't)** : application ou espace pour les patients ; prise de rendez-vous et facturation ; vraies données de santé.

## 4. Exigences non fonctionnelles

Chaque exigence est mesurable ou vérifiable, et indique le ticket qui la met en place ou la contrôle.

### 4.1 Sécurité

| ID | Exigence | Vérification |
|---|---|---|
| ENF-SEC-1 | Un praticien ne voit, ne modifie ni ne supprime jamais les patients d'un autre praticien, y compris en cas de bug applicatif : filtrage dans l'API et Row-Level Security PostgreSQL. | Tests croisés API et SQL (RA-050, RA-044, RA-052) |
| ENF-SEC-2 | Jeton d'accès de 15 minutes, jeton de rafraîchissement avec rotation et liste noire ; déconnexion automatique après 30 minutes d'inactivité (US2). | Tests (RA-042, RA-110) |
| ENF-SEC-3 | Le service IA n'accepte que les jetons émis par Django, et le jeton de service uniquement sur ses endpoints internes. | Tests 401 (RA-043) |
| ENF-SEC-4 | Aucun secret dans le dépôt : toute la configuration passe par des variables d'environnement. | Revue de code, `bandit` en CI (RA-025) |
| ENF-SEC-5 | Aucune vulnérabilité de niveau élevé dans les dépendances, aucune alerte de niveau moyen ou élevé non traitée au scan dynamique. | `pip-audit`, `npm audit --audit-level=high` (RA-025) ; OWASP ZAP (RA-132) |
| ENF-SEC-6 | Un document piégé ne détourne pas l'assistant : 0 détournement sur 5 exécutions de chaque scénario d'injection. | RA-141 |

### 4.2 Comptes et mots de passe

- **Création de compte** : pas d'inscription libre. L'administrateur crée le compte ; le praticien reçoit un lien d'invitation à usage unique et choisit lui-même son mot de passe. L'administrateur ne connaît jamais le mot de passe.
- **Mot de passe** : 12 caractères minimum, 64 acceptés, aucune règle de composition imposée (majuscule, chiffre, symbole), refus des mots de passe courants ou trop proches de l'e-mail ou du nom, coller autorisé, pas d'expiration périodique.
- **Stockage** : hachage Argon2.
- **Tentatives** : blocage temporaire après 5 échecs (RA-047).
- **Messages** : neutres, ils ne révèlent jamais si un compte existe (connexion et mot de passe oublié).
- **Références** : recommandation CNIL de 2022 sur les mots de passe et NIST SP 800-63B révision 4.
- **Limite assumée** : pas de double authentification. En santé réelle, les professionnels passent par Pro Santé Connect et la carte e-CPS ; hors périmètre du démonstrateur.

Mise en œuvre et tests : RA-041 (invitation, politique, Argon2), RA-042 (connexion), RA-046 (réinitialisation), RA-047 (blocage).

### 4.3 RGPD et données de santé

| ID | Exigence | Vérification |
|---|---|---|
| ENF-RGPD-1 | Données synthétiques uniquement, jamais de vraie donnée de santé, y compris dans les tests, les captures et les exemples. | Revue de chaque PR |
| ENF-RGPD-2 | Minimisation : patient identifié par un pseudonyme (« Patient P-0142 ») ; ni numéro de sécurité sociale, ni adresse. | Modèle de données (RA-012, RA-050) |
| ENF-RGPD-3 | Le service IA ne reçoit jamais l'identité du patient ; le contexte patient est envoyé au LLM à la volée et n'est jamais indexé dans la base vectorielle. | Modèle de menace (RA-017), revue (RA-082) |
| ENF-RGPD-4 | Journal d'audit en ajout seul : création, modification, export et effacement d'un patient, accès d'un confrère (US19), avec qui, quand et quel patient. | Tests (RA-050, RA-054, RA-100, RA-101) |
| ENF-RGPD-5 | Export des données d'un patient en JSON et PDF : fiche, séances, programmes, comptes rendus. | RA-100 |
| ENF-RGPD-6 | Effacement dans tous les stockages (base, historiques, fichiers, traces d'observabilité), suivi d'un contrôle automatique qui trouve 0 donnée restante. | RA-101 |
| ENF-RGPD-7 | Aucune donnée personnelle ou de santé dans les logs, Sentry ou Langfuse. | Test automatique (RA-028, RA-090) |
| ENF-RGPD-8 | Durées de conservation définies par type de donnée et purge automatique. | RA-102 (valeurs fixées dans ce ticket) |
| ENF-RGPD-9 | Fournisseur de LLM hébergé dans l'Union européenne, sans rétention ni réutilisation des données pour l'entraînement. | ADR du choix du fournisseur |

### 4.4 IA : fiabilité et transparence

| ID | Exigence | Vérification |
|---|---|---|
| ENF-IA-1 | Toute réponse cite au moins une source (document et passage), ou dit qu'aucune source pertinente n'a été trouvée, sans inventer. | Tests et jeu de questions de référence (RA-081, RA-070) |
| ENF-IA-2 | Une proposition de programme naît « en attente de validation » et n'est jamais appliquée sans action du praticien. | Tests (RA-083, RA-084) |
| ENF-IA-3 | L'assistant ne propose que des exercices présents dans la bibliothèque. | Tests (RA-083) |
| ENF-IA-4 | Tout contenu généré par l'IA est marqué comme tel dans l'interface et dans les PDF exportés (transparence, AI Act). | Tests de composants (RA-086, RA-087, RA-112) |
| ENF-IA-5 | Une modification qui fait baisser un score d'évaluation de plus de 5 points par rapport à main bloque le merge. | RA-151 |

### 4.5 Performance

| ID | Exigence | Vérification |
|---|---|---|
| ENF-PERF-1 | API hors IA : latence p95 inférieure à **500 ms** avec **50 praticiens simultanés** (LLM simulé). | Test de charge Locust (RA-142) |
| ENF-PERF-2 | Assistant : premier mot affiché en moins de **2 s** en conditions normales. | RA-115 |
| ENF-PERF-3 | Assistant : réponse complète à une question avec sources en moins de **10 s** au p95. | Latence mesurée dans Langfuse (RA-090, RA-091) |

Le p95 est la valeur sous laquelle se situent 95 % des mesures : il reflète ce que vivent presque tous les utilisateurs, contrairement à la moyenne qui masque les requêtes lentes.

### 4.6 Coût des appels au LLM

| ID | Exigence | Vérification |
|---|---|---|
| ENF-COUT-1 | Budget maximal de **10 € par mois** configuré chez le fournisseur de LLM. | RA-130 |
| ENF-COUT-2 | Plafond de questions par praticien et par jour, et plafond de coût journalier ; au-delà, un message clair indique l'heure de reprise. | RA-089 |
| ENF-COUT-3 | Aucune clé d'API ni appel au vrai modèle dans la CI de chaque PR : un faux LLM aux réponses enregistrées est utilisé ; le vrai modèle n'est appelé que dans les évaluations. | RA-027, RA-092, RA-141, RA-151 |
| ENF-COUT-4 | Le coût de chaque appel est tracé et consultable par jour et par praticien (US13). | RA-090, RA-091 |

### 4.7 Observabilité

| ID | Exigence | Vérification |
|---|---|---|
| ENF-OBS-1 | Une requête se suit dans les logs de Django, FastAPI et Celery grâce au même identifiant de corrélation. | RA-028 |
| ENF-OBS-2 | Chaque erreur remonte dans Sentry avec son identifiant de corrélation. | RA-028 |
| ENF-OBS-3 | Chaque appel au LLM est tracé dans Langfuse avec son coût, sa latence et la version du prompt. | RA-090, RA-094 |
| ENF-OBS-4 | La démo en ligne est surveillée toutes les 5 minutes, avec alerte par e-mail. | RA-131 |

### 4.8 Accessibilité et interface

| ID | Exigence | Vérification |
|---|---|---|
| ENF-ACC-1 | Niveau WCAG AA (RGAA) : contrastes conformes, texte d'au moins 16 px, zones cliquables d'au moins 44 px, focus clavier visible, aucune information portée par la couleur seule. | Contrôle axe automatique sur chaque écran (RA-111), maquettes (RA-016) |
| ENF-ACC-2 | Tous les écrans fonctionnent en 1440 px ; les écrans praticien (E03 à E09) aussi en 1024 px (tablette). | Tests visuels aux deux tailles (RA-144) |
| ENF-ACC-3 | Interface entièrement en français ; mention « Démonstrateur, données synthétiques, pas un dispositif médical » dans le pied de page. | Maquettes (RA-016), tests visuels |

### 4.9 Qualité du code

| ID | Exigence | Vérification |
|---|---|---|
| ENF-QUAL-1 | Couverture de tests d'au moins 80 % sur le back-end (jalon v0.1). | Rapport de couverture en CI (RA-024, RA-025) |
| ENF-QUAL-2 | Aucun merge sur main sans CI verte (lint, typage, tests). | Protection de main (RA-020, RA-025) |
| ENF-QUAL-3 | Chaque ligne d'une PR peut être expliquée avant le merge. | Relecture du développeur, questions de compréhension |

## 5. Questions ouvertes

| # | Question | Proposition en attendant | Où trancher |
|---|---|---|---|
| Q1 | Les personas n'ont pas été confrontés à un praticien réel. | Les garder comme hypothèses ; organiser un entretien dès qu'un contact est possible et mettre à jour ce document. | Développeur |
| Q2 | Le chiffrement applicatif des champs sensibles (notes cliniques), décidé dans la page « RGPD et concurrence », n'est prévu par aucun ticket du board. | Le trancher dans le modèle de menace (RA-017) : soit l'ajouter à RA-052, soit l'écarter dans un ADR. | RA-017 |
| Q3 | Valeur des quotas : nombre de questions par praticien et par jour, plafond de coût journalier. | La fixer pour tenir dans le budget de 10 € par mois. | RA-089 |
| Q4 | Durées de conservation par type de donnée (patients inactifs, historique des questions, traces, logs, sauvegardes). | — | RA-102 |
| Q5 | Aucun objectif de disponibilité n'est fixé pour la démo. | Pas d'engagement de disponibilité, surveillance seulement (ENF-OBS-4). | Développeur |
| Q6 | Navigateurs à prendre en charge non précisés. | Dernières versions de Chrome, Edge, Firefox et Safari. | Développeur |
