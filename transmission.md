# Transmission — Projet Blendora

## 1. Objectif du projet

Application de matchmaking relationnel (rencontres compatibles) pour PME/indépendants.
Single-file HTML (`blendora.html`, ~7 100 lignes), tout en vanilla JS inline, pas de build tool.
L'admin importe des participants depuis Airtable, lance un matching par score de compatibilité,
puis envoie des notifications WhatsApp personnalisées à chaque participant.

---

## 2. État actuel du code

**Fichier principal :** `blendora.html` (7 168 lignes, 233 fonctions JS)

**Fonctions clés à connaître :**

| Fonction | Rôle |
|---|---|
| `buildMsgNotifWA(personne, matchPers, score, resumeIA)` | Construit le message WA standard via `WA_NOTIF_TEMPLATE_DEF` |
| `WA_NOTIF_TEMPLATE_DEF` | Template compact avec `{{placeholders}}` dont `{{resume_ia_block}}` |
| `genSummaryMatch(personne, matchPers, score)` | Résumé IA contextualisé au duo (2 phrases, LLM) |
| `genSummaryProfil(p)` | Portrait factuel d'un profil seul (2 phrases, LLM) |
| `envoyerFicheWAAvecResume(evt, matchId, who)` | Boutons "Fiche de X → Y" en Résultats — async, appelle `genSummaryMatch` |
| `notifierMatchDepuisFiche(id, evt)` | Bouton "💑 Notif match" dans fiche participant — async, appelle `genSummaryMatch` |
| `buildMsgNotifWAPremium(personne, matchPers, score, accrochePerso, dateSuggestion, resumeIA)` | Message Premium autonome (langage amour + accroche + RDV) |
| `envoyerNotifPremiumWA(id, evt)` | Bouton Premium — async, appelle `genAccroche` + `genAccrocheIA` + `genSummaryMatch` |
| `genAccroche(h, f)` | Génère `{msgH, msgF, dateProp}` synchrone |
| `genAccrocheIA(h, f, score)` | Version IA async des messages d'accroche |
| `loadSettings()` | Inclut migration : reset `wa_notif_template` si format obsolète (sans `{{resume_ia_block}}` ou sans `4 €`) |
| `purgeMockData()` | IIFE dans `window.onload` — purge les IDs H1-H5/F1-F5 du localStorage |

**Template WA actuel (`WA_NOTIF_TEMPLATE_DEF`) :**
```
Bonjour {{prenom}},
Nous avons le plaisir de vous presenter votre match du mois Blendora.
━━━━━━━━━━━━━━━━━━━
  Fiche profil Blendora
━━━━━━━━━━━━━━━━━━━
{{prenom_match}} {{nom_match}} — {{age_match}} ans
📍 Ville : {{ville_match}}
💼 Profession : {{profession_match}}
{{bio_match}}{{photo_match}}📱 WhatsApp : {{whatsapp_match}}
📧 Email : {{email_match}}
Compatibilite : {{score}}%
━━━━━━━━━━━━━━━━━━━{{resume_ia_block}}
Pour recevoir l'analyse complète du langage de l'amour, la phrase d'accroche personnalisée et le meilleur lieu de rendez-vous, un supplément de 4 € est demandé.
Belle journee,
Blendora
```

Quand `resumeIA` est fourni, `{{resume_ia_block}}` devient :
```
\n🧠 Résumé du profil :\n[texte]\n━━━━━━━━━━━━━━━━━━━
```

**Message Premium autonome :**
```
⭐ Premium Blendora
━━━━━━━━━━━━━━━━━━━
Bonjour [prenom],
Voici votre analyse complète pour votre match avec [match] (score%).
━━━━━━━━━━━━━━━━━━━
--- Infos complémentaires ---
━━━━━━━━━━━━━━━━━━━

💝 Langage d'amour de [match] : [langage]
💬 Message suggéré à envoyer à [match] : [msgH ou msgF selon le profil]
📍 Suggestion de 1er rendez-vous : [dateProp nettoyé du HTML]
━━━━━━━━━━━━━━━━━━━
🧠 Analyse IA : [resumeIA si dispo]
━━━━━━━━━━━━━━━━━━━
Belle journée,
Blendora
```

**localStorage keys :**
- `blendora_settings` — tous les paramètres (clé LLM, Airtable, template WA custom, etc.)
- `blendora_inscriptions` — participants
- `blendora_premium` — flags premium par ID
- `blendora_blocked` — paires blacklistées
- `blendora_historique` — historique des sessions

---

## 3. Fichiers en cours de modification

- `blendora.html` — seul fichier modifié, toutes les fonctionnalités dedans

Pas de `blendora-register.html` modifié dans cette session.

---

## 4. Ce qui a changé depuis le début de cette session

1. **MOCK data supprimés** — `MOCK_H`, `MOCK_F`, `MOCK_MATCHS` vidés ; `purgeMockData()` ajouté dans `window.onload` pour nettoyer les IDs H1-H5/F1-F5 du localStorage.

2. **`genSummaryProfil`** — prompt réécrit en français, concis et factuel (2 phrases max).

3. **`genSummaryMatch`** — nouvelle fonction : résumé IA contextualisé au duo (prend les deux profils + score, met en valeur la compatibilité).

4. **`WA_NOTIF_TEMPLATE_DEF`** — format compact avec séparateurs `━━━`, placeholder `{{resume_ia_block}}`, footer upsell 4 €.

5. **`buildMsgNotifWA`** — 4e param `resumeIA` optionnel, gère `resumeBlock` avec son propre séparateur trailing.

6. **`envoyerFicheWAAvecResume`** — passe de `genSummaryProfil` à `genSummaryMatch`.

7. **`notifierMatchDepuisFiche`** — rendu `async`, appelle `genSummaryMatch`, loader sur le bouton, passe `event` depuis le `onclick`.

8. **`buildMsgNotifWAPremium`** — message autonome (ne concatène plus avec le message standard), accepte `accrochePerso`, `dateSuggestion`, `resumeIA`.

9. **`envoyerNotifPremiumWA`** — rendu `async`, appelle `genAccroche` (sync) puis `genAccrocheIA` (async IA) pour récupérer `msgH`/`msgF`/`dateProp`, choisit le bon message selon `isH`, nettoie le HTML de `dateProp`, appelle `genSummaryMatch`.

10. **Migration `loadSettings`** — reset automatique de `settings.wa_notif_template` si format obsolète (détection : absence de `{{resume_ia_block}}` ou de `4 €`).

---

## 5. Ce qui a été testé et n'a pas marché

- **Patch Python avec strings UTF-8 exactes** — les caractères accentués (é, à, â) dans `old_string` ne matchent pas toujours le fichier à cause de l'encodage. Solution : utiliser les numéros de ligne (`lines[start:end]`) ou `re.subn` avec regex.

- **Remplacement en une seule passe des deux fonctions premium** — le bloc exact incluait des chars UTF-8 (`café`, `Accepté`, `—`) qui faisaient échouer `assert old_block in content`. Solution : remplacement par ligne avec `f.readlines()` + slicing.

---

## 6. Ce qui était prévu ensuite

Rien de demandé explicitement. Pistes potentielles selon contexte :

- Tester en production les 3 boutons WA (Fiche, Notif match, Premium) avec de vrais profils Airtable.
- Vérifier que `genAccrocheIA` retourne bien `dateProp` en texte propre (certaines versions IA omettent ce champ).
- Ajouter un bouton "Aperçu du message Premium" dans la modal participant avant envoi.
- Pousser sur GitHub (`git add blendora.html && git commit -m "..." && git push origin main`).
