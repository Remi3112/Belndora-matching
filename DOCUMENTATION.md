# Documentation Blendora — Guide complet

## Vue d'ensemble

Blendora est une application de matchmaking relationnel single-file. Tout tient dans **`blendora.html`** : HTML, CSS et JavaScript inline, sans framework ni build tool. L'admin ouvre le fichier dans un navigateur et tout fonctionne localement. Les données sont stockées dans le localStorage du navigateur.

---

## Flux de travail principal

```
Airtable → Import → Participants → Matching → Notifications WhatsApp
```

**1. Import Airtable** — onglet Participants → bouton "Sync Airtable". L'app lit les tables via l'API Airtable et fusionne les nouvelles inscriptions avec les données existantes.

**2. Matching** — onglet Résultats → bouton "Lancer le matching". L'algorithme calcule un score de compatibilité (valeurs communes, activités, rythme de vie, langage de l'amour, etc.) pour chaque paire Homme/Femme, puis génère les matchs.

**3. Notification WhatsApp** — pour chaque match, deux boutons envoient un message pré-rempli via `wa.me` :
- **Fiche de X → Y** : envoie la fiche profil du match avec (si IA activée) un résumé de compatibilité
- **💑 Notif match** (depuis la fiche participant) : même chose mais déclenché depuis le profil

**4. Option Premium** — bouton ⭐ dans la fiche participant. Envoie un message enrichi avec le langage d'amour du match, un message d'accroche personnalisé et une suggestion de rendez-vous.

---

## Configurer la clé API pour l'IA (ChatGPT / Claude / autre)

L'IA est **facultative**. Sans clé, tous les boutons fonctionnent normalement sans résumé IA.

### Étape 1 — Obtenir une clé API

**OpenAI (ChatGPT / GPT-4)**
1. Aller sur [platform.openai.com](https://platform.openai.com)
2. Se connecter ou créer un compte
3. Menu en haut à droite → **API keys** → **Create new secret key**
4. Copier la clé (commence par `sk-...`)

> ⚠️ La clé n'est visible qu'une seule fois. Copiez-la immédiatement.

**Anthropic (Claude)**
1. Aller sur [console.anthropic.com](https://console.anthropic.com)
2. Se connecter → **API Keys** → **Create Key**
3. Copier la clé (commence par `sk-ant-...`)

### Étape 2 — Configurer dans Blendora

1. Ouvrir `blendora.html` dans le navigateur
2. Aller dans l'onglet **⚙️ Paramètres**
3. Section **🤖 Intelligence Artificielle (LLM)** :

| Champ | Valeur |
|---|---|
| Fournisseur | `openai` pour ChatGPT, `anthropic` pour Claude |
| Modèle | `gpt-4o` ou `gpt-4o-mini` (OpenAI) / `claude-sonnet-4-6` (Anthropic) |
| Clé API | Coller votre clé ici |

4. Cliquer **🔍 Tester l'IA** pour vérifier que la connexion fonctionne
5. Cliquer **💾 Enregistrer**

> La clé est stockée dans le localStorage du navigateur, uniquement sur votre machine. Elle n'est jamais envoyée ailleurs que vers l'API du fournisseur choisi.

### Coût estimé

- **GPT-4o-mini** : ~0,001 € par message généré (très économique)
- **GPT-4o** : ~0,01 € par message
- **Claude Sonnet** : ~0,003 € par message

Chaque notification génère 1 à 2 appels IA selon le mode (standard ou premium).

---

## Les deux prompts système (Paramètres → IA)

La section IA des Paramètres contient deux éditeurs de prompt que vous pouvez personnaliser :

### 🎯 Prompt — Notification standard

Contrôle le résumé IA injecté dans le message WhatsApp standard.

**Quand il est utilisé :** bouton "Envoyer fiche" dans Résultats + bouton "💑 Notif match" dans la fiche participant.

**Ce que l'IA génère :** 2 phrases décrivant le profil du match et mettant en avant un point fort de compatibilité avec le destinataire.

**Prompt par défaut intégré (si vide) :**
```
Tu es un assistant Blendora. A partir des profils ci-dessous, redige 2 phrases maximum 
en francais. Decris brievement le profil du match (age, ville, profession, 
caracteristiques cles), puis mets en valeur un ou deux points forts de compatibilite 
avec le destinataire. Sois factuel, concis, positif. Ne repete pas les noms des champs. 
Ne mets pas de titre.
```

**Exemple de personnalisation :**
```
Tu es un assistant Blendora bienveillant. En 2 phrases max, décris le profil du match 
de façon chaleureuse et mets en avant ce qui fait que cette rencontre a du sens. 
Ton positif, jamais générique.
```

---

### ⭐ Prompt — Analyse Premium

Contrôle le ton et le style des messages générés pour la notification Premium.

**Quand il est utilisé :** bouton ⭐ Premium dans la fiche participant.

**Ce que l'IA génère :**
- `msgH` — message d'accroche que l'homme peut copier-coller et envoyer à la femme
- `msgF` — même chose du côté de la femme
- `dateProp` — suggestion de lieu/activité pour le premier rendez-vous
- `langageAnalyse` — analyse de la compatibilité des langages de l'amour

**Prompt par défaut intégré (si vide) :**
```
Tu es conseiller en rencontres pour Blendora (service de matching amoureux).
[...profils fournis dynamiquement...]
Génère exactement un JSON avec les clés msgH, msgF, dateProp, langageAnalyse.
Messages naturels, personnels, 2-3 phrases, avec emoji. Commence par bonjour + 
quelque chose de personnel. Pas de "je suis tombé(e) sur ton profil".
```

**Exemple de personnalisation :**
```
Tu es un coach en relations pour Blendora. Tes messages sont chaleureux, directs 
et inspirants. Propose une activité originale adaptée aux goûts communs. 
Évite les formules convenues.
```

> **Si le champ est laissé vide**, le prompt interne par défaut est utilisé — il est déjà optimisé pour produire de bons résultats.

---

## Architecture technique

### Stockage (localStorage)

| Clé | Contenu |
|---|---|
| `blendora_settings` | Tous les paramètres (Airtable, LLM, template WA, prompts...) |
| `blendora_inscriptions` | Liste des participants |
| `blendora_premium` | Flags premium par ID participant |
| `blendora_blocked` | Paires blacklistées (ne seront plus matchées) |
| `blendora_historique` | Historique des sessions de matching |
| `blendora_paires_matchees` | Paires déjà matchées (évite les doublons) |

### Fonctions principales

| Fonction | Rôle |
|---|---|
| `calcScore(h, f)` | Score de compatibilité 0-100% entre un homme et une femme |
| `buildMsgNotifWA(personne, matchPers, score, resumeIA)` | Construit le message WA standard via template |
| `buildMsgNotifWAPremium(...)` | Message WA premium (autonome) |
| `genSummaryMatch(personne, matchPers, score)` | Résumé IA du duo (utilise le Prompt standard) |
| `genAccrocheIA(h, f, score)` | Génère msgH/msgF/dateProp via IA (utilise le Prompt premium) |
| `envoyerNotifPremiumWA(id, evt)` | Orchestre l'envoi premium (sync + IA + résumé) |
| `callLLM(prompt, key, provider, model, ...)` | Appel générique vers n'importe quel LLM |

### Template WhatsApp (personnalisable dans Paramètres → WhatsApp)

Le template standard utilise des `{{placeholders}}` :

| Placeholder | Valeur |
|---|---|
| `{{prenom}}` | Prénom du destinataire |
| `{{prenom_match}}` | Prénom du match |
| `{{nom_match}}` | Nom du match |
| `{{age_match}}` | Âge du match |
| `{{ville_match}}` | Ville du match |
| `{{profession_match}}` | Profession du match |
| `{{whatsapp_match}}` | Numéro WhatsApp du match |
| `{{email_match}}` | Email du match |
| `{{score}}` | Score de compatibilité (%) |
| `{{bio_match}}` | Biographie du match (avec saut de ligne si non vide) |
| `{{photo_match}}` | URL photo du match (avec saut de ligne si non vide) |
| `{{resume_ia_block}}` | Bloc résumé IA (vide si IA non configurée ou sans résultat) |

> **Migration automatique** : si un ancien template sans `{{resume_ia_block}}` ou sans le footer 4€ est détecté au chargement, il est automatiquement réinitialisé vers le template par défaut.

---

## Providers LLM supportés

| Provider | Valeur | Modèles recommandés |
|---|---|---|
| OpenAI | `openai` | `gpt-4o-mini`, `gpt-4o` |
| Anthropic | `anthropic` | `claude-sonnet-4-6`, `claude-haiku-4-5-20251001` |
| Mistral | `mistral` | `mistral-small`, `mistral-medium` |
| Groq | `groq` | `llama-3.1-8b-instant`, `mixtral-8x7b-32768` |
| Custom | `custom` | Endpoint OpenAI-compatible |

Pour un provider **custom** (Ollama, LM Studio, etc.), renseigner l'endpoint complet dans le champ dédié, ex : `http://localhost:11434/v1/chat/completions`.

---

## FAQ

**Q : Les données sont-elles sécurisées ?**  
Toutes les données restent dans votre navigateur (localStorage). Aucun serveur Blendora ne reçoit vos données. Seuls les appels IA envoient les profils (anonymisés : prénom, âge, valeurs) au fournisseur LLM choisi.

**Q : Comment réinitialiser toutes les données ?**  
Paramètres → bas de page → "Effacer toutes les données". Ou manuellement : F12 → Application → localStorage → Tout supprimer.

**Q : Le bouton IA tourne indéfiniment ?**  
Vérifier que la clé API est valide (bouton "Tester l'IA") et que le fournisseur/modèle est correctement renseigné.

**Q : Comment partager les paramètres entre navigateurs ?**  
Paramètres → "Exporter les paramètres" génère un fichier JSON. L'importer sur l'autre machine avec "Importer les paramètres".
