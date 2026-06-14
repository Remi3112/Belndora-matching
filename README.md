# Blendora — Application de Matching IA

Application web de matching pour événements de rencontres. Single-file HTML, zéro installation, connectée à Airtable.

---

## Sommaire

- [Présentation](#présentation)
- [Fonctionnalités](#fonctionnalités)
- [Architecture technique](#architecture-technique)
- [Installation et lancement](#installation-et-lancement)
- [Configuration Airtable](#configuration-airtable)
- [Configuration LLM (IA)](#configuration-llm-ia)
- [Structure Airtable attendue](#structure-airtable-attendue)
- [Algorithme de matching](#algorithme-de-matching)
- [Module Événements et Inscriptions](#module-événements-et-inscriptions)
- [Module Groupes](#module-groupes)
- [Notifications WhatsApp](#notifications-whatsapp)
- [Mode Premium](#mode-premium)
- [Page d'inscription publique](#page-dinscription-publique)
- [Stockage local (localStorage)](#stockage-local-localstorage)
- [Mise à jour et déploiement GitHub](#mise-à-jour-et-déploiement-github)

---

## Présentation

Blendora est un outil de gestion de matching conçu pour les organisateurs de soirées rencontres. Il permet de :

- Importer les profils participants depuis Airtable
- Lancer un algorithme de matching multicritère (score de compatibilité)
- Gérer les événements et les inscriptions
- Constituer des groupes de rencontre
- Envoyer les notifications de match par WhatsApp
- Proposer une expérience enrichie aux participants Premium

**Deux fichiers :**
- `blendora.html` — Interface d'administration (organisateur)
- `blendora-register.html` — Page publique d'inscription pour les participants

---

## Fonctionnalités

### Dashboard
Vue d'ensemble en temps réel : nombre de participants actifs, matchs du jour, meilleur score de compatibilité, sessions de matching. Widget de relance rapide vers le moteur de matching.

### Participants
- Synchronisation depuis Airtable (bouton "Sync Airtable")
- Filtrage par sexe, tranche d'âge, ville, statut actif
- Fiche détaillée par participant : valeurs, langage d'amour, projet de vie, activités, tempérament, rythme, bio
- Envoi du profil par WhatsApp directement depuis la fiche
- Toggle Premium par participant (active les notifications enrichies)

### Matching
- Algorithme multicritère avec score de compatibilité 0–100%
- Mode IA : génération d'une phrase d'accroche personnalisée via GPT/Mistral/Groq
- Paramètres ajustables : écart d'âge max, score minimum, nombre de matchs max par personne
- Contexte libre pour orienter l'IA (ville, thème de soirée, contraintes)
- Exclusion automatique des paires déjà matchées
- Blacklist de paires définitives
- Matching manuel : sélection libre de deux participants + calcul de score en temps réel

### Résultats
- Liste de tous les matchs avec score, statut, phrase d'accroche
- Fiche détaillée de chaque match (langages d'amour, points communs, accroche)
- Statuts : En attente / Envoyé / Accepté / Refusé
- Envoi de la fiche de match par WhatsApp (un bouton par personne)
- Export CSV de tous les matchs
- Recherche et tri par score, prénom, statut

### Événements
- Création d'événements (nom, date, lieu, capacité, description)
- Génération d'un lien d'inscription unique par événement
- Validation manuelle des inscriptions (accepter / refuser / mettre en attente)
- Badge de notification en temps réel pour les nouvelles inscriptions

### Groupes
- Création manuelle : ajout/retrait de participants par clic
- Constitution automatique depuis un événement ("Groupes IA") : répartition H/F équilibrée, score moyen calculé
- Modification d'un groupe existant
- Taille de groupe adaptée automatiquement au nombre d'inscrits disponibles

### Analytics
- Distribution des scores de compatibilité
- Top matchs de la session
- Historique des sessions de matching
- Statistiques globales (taux d'acceptation, scores moyens)

### Paramètres
- Connexion Airtable (clé API, Base ID, noms de tables)
- Configuration LLM (provider, modèle, clé API)
- Template de message WhatsApp personnalisable avec variables
- Gestion des données locales (export JSON, import, remise à zéro)

---

## Architecture technique

```
blendora.html          (~6800 lignes)
├── <style>            CSS inline, layout flex sidebar+main
├── <body>             HTML de toutes les pages (display:none/block)
└── <script>           Tout le JavaScript inline

blendora-register.html Page publique d'inscription
```

Pas de build, pas de npm, pas de framework. Vanilla JS + HTML + CSS dans un seul fichier.

### Navigation
`goTo('section')` bascule l'affichage entre les sections via `display:none/block` avec animation CSS. Sidebar fixe desktop + barre mobile en bas de l'écran.

---

## Installation et lancement

### Option 1 — Ouvrir directement (mode lecture seule)
Double-cliquer sur `blendora.html`. Fonctionne pour la navigation mais bloque les appels Airtable (restriction CORS du navigateur en `file://`).

### Option 2 — Serveur local (recommandé)
```bash
# Python
python -m http.server 8080

# Node.js
npx serve .
```
Puis ouvrir `http://localhost:8080/blendora.html`

### Option 3 — GitHub Pages (accès en ligne)
Activer GitHub Pages sur ce repo : Settings → Pages → Source: main → Save.
L'app sera accessible sur : `https://remi3112.github.io/Belndora-matching/blendora.html`

---

## Configuration Airtable

Aller dans **Paramètres → Connexion Airtable** :

| Champ | Description |
|---|---|
| Clé API | Personal Access Token Airtable (commence par `pat...`) |
| Base ID | Identifiant de la base (commence par `app...`) |
| Table participants | Nom ou ID de la table principale |
| Table compatibilité | Table secondaire (optionnelle) |
| Table inscriptions | Nom exact de la table des inscriptions |

Cliquer **Enregistrer** puis **Tester la connexion** pour vérifier.

> En cas d'erreur CORS : utiliser un serveur local (Option 2 ci-dessus).

---

## Configuration LLM (IA)

Aller dans **Paramètres → Modèle IA** :

| Provider | Modèles supportés |
|---|---|
| OpenAI | gpt-4o, gpt-4o-mini, gpt-4-turbo |
| Mistral AI | mistral-large, mistral-medium, mistral-small |
| Groq | llama-3, mixtral-8x7b |

L'IA est utilisée pour générer la phrase d'accroche personnalisée entre deux matchs et enrichir le contexte de matching. Sans clé LLM, l'algorithme algorithmique seul fonctionne (score calculé, accroche générée localement).

---

## Structure Airtable attendue

### Table Participants

| Champ Airtable | Type | Description |
|---|---|---|
| `Prénom` | Texte | Prénom du participant |
| `Nom` | Texte | Nom de famille |
| `Sexe` | Select | `Homme` ou `Femme` (valeurs exactes requises) |
| `Age` | Nombre | Âge en années |
| `Ville` | Texte | Ville de résidence |
| `Email` | Email | Adresse email |
| `WhatsApp` | Texte | Numéro au format `+336XXXXXXXX` |
| `Profession` | Texte | Métier |
| `Bio` | Texte long | Présentation libre |
| `Photo` | Pièce jointe ou URL | Photo de profil |
| `Valeurs` | Multi-select | Ex : Famille, Liberté, Authenticité |
| `Langage` | Select | Langage d'amour (5 types) |
| `Tempérament` | Select | Ex : Dynamique, Posé, Créatif |
| `Rythme` | Select | Ex : Actif, Équilibré, Flexible |
| `Role` | Select | Ex : Protecteur, Complémentaire, Leader |
| `Projet` | Select | Projet de vie |
| `Activités` | Multi-select | Loisirs et activités |
| `Accroche` | Texte long | Phrase d'accroche personnalisée (optionnel) |
| `Activite_suggest` | Texte | Suggestion de premier rendez-vous (optionnel) |
| `Actif` | Checkbox | Participant disponible pour matching |

### Table Inscriptions
Créée automatiquement via le bouton **"Créer la table Inscriptions"** dans Paramètres. Champs : Prénom, Nom, Email, WhatsApp, Événement ID, Date, Statut.

---

## Algorithme de matching

### Critères et poids

| Critère | Poids | Logique |
|---|---|---|
| Valeurs communes | 35% | Intersection des valeurs, plafonné à 3 communes max |
| Projet de vie | 20% | Correspondance exacte = 100%, compatible = 60% |
| Langage d'amour | 15% | Identique = 100%, compatible = 50% |
| Tempérament | 10% | Matrice de compatibilité prédéfinie |
| Activités communes | 10% | Nombre d'activités en commun (bonus si 2+) |
| Rythme de vie | 5% | Identique = 100%, proche = 60% |
| Écart d'âge | 5% | Bonus si < 5 ans, malus si > 10 ans |

**Score final : 0–100%**
- >= 88% : Excellente compatibilité (vert)
- >= 75% : Bonne compatibilité (orange)
- < 75% : Correcte (rouge)

### Processus d'attribution

1. Calcul des scores pour toutes les paires H/F possibles
2. Exclusion des paires déjà matchées (historique localStorage)
3. Exclusion des paires blacklistées
4. Application des filtres (écart d'âge max, score minimum)
5. Attribution optimale : chaque personne reçoit au maximum N matchs (paramétrable)
6. Génération de la phrase d'accroche (IA si clé configurée, sinon locale)

### Matching Manuel
Sélectionner deux participants depuis l'onglet Manuel. Score calculé en temps réel. La fiche de match peut être envoyée immédiatement par WhatsApp.

---

## Module Événements et Inscriptions

### Côté organisateur (`blendora.html`)
1. Créer un événement (nom, date, lieu, capacité max, description)
2. Copier le lien d'inscription unique (vers `blendora-register.html?event=ID`)
3. Valider les inscriptions : tableau avec actions Accepter / Refuser / En attente
4. Constituer les groupes IA depuis les inscrits validés

### Côté participant (`blendora-register.html`)
Formulaire public : prénom, nom, email, WhatsApp, ville, âge, sexe. Soumission enregistrée dans Airtable. Page accessible sans compte, fonctionne sur mobile.

---

## Module Groupes

### Création manuelle
Bouton "Nouveau groupe" → sélectionner les participants un par un. Le compteur H/F se met à jour en temps réel. Score moyen du groupe calculé automatiquement.

### Constitution automatique (Groupes IA)
Depuis la section Événements → bouton **"Groupes IA"** :
1. Récupère les inscrits validés à l'événement
2. Adapte la taille des groupes au nombre de participants disponibles
3. Répartit H/F de manière équilibrée (shuffle aléatoire)
4. Calcule le score moyen inter-groupes
5. Sauvegarde et affiche un message de confirmation

---

## Notifications WhatsApp

Les messages s'ouvrent dans WhatsApp Web (`wa.me/...`) via le navigateur.

### Variables disponibles dans les templates

| Variable | Contenu |
|---|---|
| `{{prenom}}` | Prénom du destinataire |
| `{{prenom_match}}` / `{{nom_match}}` | Identité du match |
| `{{age_match}}` | Âge du match |
| `{{ville_match}}` | Ville du match |
| `{{score}}` | Score de compatibilité en % |
| `{{bio_match}}` | Bio du match |
| `{{message_accroche}}` | Phrase d'accroche générée |
| `{{whatsapp_match}}` | Numéro WhatsApp du match |
| `{{photo_match}}` | URL de la photo du match |

Le template est entièrement personnalisable dans Paramètres → Template WhatsApp.

---

## Mode Premium

Activé participant par participant depuis la **fiche participant → bloc Premium**.

Ce que ça change : un bouton "Envoyer notification Premium WA" apparaît dans la fiche. La notification envoyée inclut en plus du message standard :
- Le langage d'amour du match
- Une phrase d'accroche personnalisée à envoyer
- Une suggestion de premier rendez-vous adaptée aux profils des deux personnes

Les flags Premium sont sauvegardés dans `localStorage` sous la clé `blendora_premium`, indépendamment des données Airtable.

---

## Page d'inscription publique

`blendora-register.html` est une page standalone qui :
- Détecte l'événement depuis le paramètre URL `?event=ID`
- Affiche le nom et la date de l'événement
- Enregistre l'inscription directement dans Airtable (table Inscriptions)
- Fonctionne sur mobile

Pour partager : copier le lien depuis la section Événements de l'interface admin.

---

## Stockage local (localStorage)

Toutes les données sont stockées côté navigateur. Aucune base de données serveur.

| Clé | Contenu |
|---|---|
| `blendora_settings` | Configuration Airtable, LLM, préférences matching |
| `blendora_historique` | Historique des paires déjà matchées |
| `blendora_paires_matchees` | Blacklist de paires |
| `blendora_events` | Événements créés |
| `blendora_inscriptions` | Inscriptions reçues |
| `blendora_groupes` | Groupes sauvegardés |
| `blendora_premium` | Flags Premium par participant ID |

Export / Import : Paramètres → Données locales → boutons Export JSON / Import JSON / Réinitialiser.

---

## Données de démonstration

Sans connexion Airtable, l'app charge automatiquement 10 profils fictifs (5H + 5F) pour tester le matching. Ces données sont remplacées dès qu'une synchronisation Airtable réussit.

---

## Sécurité

- La clé API Airtable est stockée uniquement dans le `localStorage` du navigateur
- Ne pas partager `blendora.html` avec la clé déjà enregistrée dedans
- La page publique `blendora-register.html` ne contient aucune clé API

---

## Mise à jour et déploiement GitHub

### Premier push (une seule fois)
```powershell
cd "C:\Users\albar\OneDrive\Documents\Claude\Projects\Blendora"
git init
git add blendora.html blendora-register.html README.md
git commit -m "init: Blendora v2.0"
git remote add origin https://github.com/Remi3112/Belndora-matching.git
git branch -M main
git push -u origin main
```

### Après chaque modification
```powershell
cd "C:\Users\albar\OneDrive\Documents\Claude\Projects\Blendora"
git add blendora.html
git commit -m "update: description du changement"
git push
```

---

*Blendora v2.0 — Single-file HTML app, Vanilla JS, Airtable-powered*
