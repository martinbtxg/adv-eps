# ADV EPS — Webapp Gilac

> Fichier de contexte projet partagé. Mise à jour du journal en fin de session, puis commit.

---

## 1. Identité du projet

- **Nom de code :** ADV EPS
- **Nature :** Webapp interne Gilac pour piloter de bout en bout les **commandes EPS** (Egg Production Systems — alvéoles, intercalaires, palettes destinées aux producteurs d'œufs)
- **Périmètre fonctionnel :** ingestion multi-source (emails, input manuel chat, ERP Silver) → analyse via agents Claude (skills EPS dédiés) → dashboard interactif multi-rôle → actions & notifications
- **Client / utilisateurs :** Gilac (interne) — dirigeant, commerciaux, ADV, supply chain
- **Repo :** https://github.com/martinbtxg/adv-eps
- **Équipe :** Martin Bouthiaux (martin.bouthiaux@gilac.com) + martin-acco33
- **Démarrage :** 2026-05-19

---

## 2. Objectif fonctionnel

Centraliser le pilotage des commandes EPS, aujourd'hui éclaté entre :

- **Notion** — base "Suivi des Commandes EPS" (source actuelle, à migrer)
- **Outlook / Teams** — échanges client et internes
- **SharePoint** — fichiers `COMMANDES SPID - EPS.xlsx`, `Planning de prod simplifié.xlsx`
- **Silver ERP** — commandes sous-traitants, N° CDE, factures
- **BigQuery `datalake_marts`** — entrepôt Gilac (clients, factures, legal_entity)

…vers une **webapp unique** avec table + kanban, gestion fine des droits, notifications centralisées.

---

## 3. Stack & architecture (actée)

| Couche | Technologie |
|---|---|
| Backend API | **Python FastAPI** + SQLAlchemy + Alembic |
| Worker async | Celery ou RQ (à trancher Phase 0) |
| Agent layer | **Claude Agent SDK** (Python) invoquant les skills EPS comme microservice |
| Base de données | **Postgres** |
| Frontend | **React** + Vite + TanStack Table |
| Styling | Tailwind paramétré sur les tokens du style guide |
| Auth | Entra ID SSO (Gilac M365) + JWT côté API + RBAC métier |
| Notifications | Microsoft Graph (Teams + Outlook) |
| Infrastructure | **VM non choisie** — à cadrer Phase 0 avec IT Gilac |

**Source de vérité :** la webapp devient maître. Migration progressive depuis Notion.

---

## 4. Phasage

| Phase | Périmètre | Statut |
|---|---|---|
| 0. Specs & infra | Cadrage produit, choix VM, AD Gilac, structuration repo | En cours |
| 1. MVP "viewer" | Webapp lecture seule : tableau + kanban, SSO, déployée | À faire |
| 2. Agent EPS server-side | Service Claude Agent SDK exécutant les skills EPS sur ingestion mail/input | À faire |
| 3. Actions & rôles | Input manuel webapp, permissions fines, audit log | À faire |
| 4. Notifications | Teams/email sur événements clés | À faire |

---

## 5. Skills Claude EPS à utiliser

Ces skills sont chargés dans l'environnement Gilac et doivent piloter la couche d'analyse server-side via Claude Agent SDK :

- **`adv-eps`** — agent principal : suivi de bout en bout des commandes (Notion + Outlook + Teams + Silver)
- **`planning-production-eps`** — lit/met à jour le planning des 3 sites EPS (`Planning de prod simplifié.xlsx` sur SharePoint)
- **`controle-silver-eps`** — vérifie la saisie Silver d'une commande sous-traitant, récupère le N° CDE
- **`export-commandes-eps`** — génère l'Excel synthèse "Commandes EPS en cours — Production par site" (déclenchement explicite uniquement)
- **`bigquery-gilac`** — accès `datalake_marts` (clients, factures, legal_entity)

---

## 6. Sites de production EPS

| Site | Produit | Régime |
|---|---|---|
| **Alençon Plastique** | Alvéoles | Sous-traitance |
| **Polysemble (IP3)** | Intercalaires | Sous-traitance |
| **Cabka** | Palettes recyclées | Sous-traitance |
| **MIAM Izernore** | Alvéoles (production interne) | Interne Gilac |

---

## 7. Rôles utilisateurs cibles

- **Dirigeant** — vue COMEX, KPI consolidés, lecture seule
- **Commercial** — ses commandes, actions à mener côté client
- **ADV** — toutes les commandes, saisie Silver, suivi délais
- **Supply chain** — planning production, expéditions, transport

Permissions fines à définir en Phase 3.

---

## 8. Style guide GILAC

**Règle absolue :** toute UI/dashboard/rapport respecte rigoureusement le style guide GILAC.

Essentiels :

- **Palette** — fond `#fafaf9`, surface blanc, encre `#0f1722`, **un seul accent émeraude `#047857`**
- **Typo** — Inter (400/500/600/700), `tnum` partout, locale FR (`Intl.NumberFormat('fr-FR')`, € après le nombre, virgule décimale, espaces insécables)
- **Forme** — radius max 10px, pas d'ombres, pas de gradients (sauf hero optionnel), pas d'emoji, pas d'icônes décoratives
- **Composants clés** — segmented control actif = fond ink (PAS l'accent), pills actif = accent, badges discrets, tableaux sans bordures verticales
- **Anti-patterns interdits** — gradients, ombres marquées, emoji, plusieurs accents, coins >10px, polices au choix

Style guide complet en mémoire Claude : voir `~/.claude/projects/*/memory/gilac_style_guide.md`.

---

## 9. Conventions de travail

### Git
- **Identité commit :** Martin Bouthiaux <martin.bouthiaux@gilac.com> (config globale)
- **Branche principale :** `main`
- **Branches feature :** `feat/<sujet>`, `fix/<sujet>`, `docs/<sujet>`
- **PR systématique** vers `main` — pas de push direct sur `main` en pratique
- **Push autorisé sans reconfirmation** pour ce repo uniquement (autres repos = confirmation requise)
- **PAT GitHub** dans Keychain macOS, scopes : Contents Read/Write + Pull requests Read/Write

### Fichiers à NE PAS committer
- `COMMANDES SPID - EPS.xlsx` — données commerciales internes Gilac
- Tout `.xlsx` / `.docx` / `.pptx` métier sans validation préalable
- Tokens, secrets, `.env`, credentials

→ Un `.gitignore` adapté reste à créer.

### Format de commit
Messages en français, premier verbe descriptif (Ajout, Mise à jour, Correction, Refactor, Doc), corps explicatif "le pourquoi". Co-author Claude conservé.

---

## 10. État courant

- **Repo GitHub** créé (privé) et identité Gilac configurée
- **Prototype HTML** dashboard livré dans `prototype/index.html` (PR #1 ouverte) — validation UX en cours
- **Phase 0** en cours : cadrage VM, AD Gilac, modèle de données

---

## 11. Journal de bord

> Mise à jour fin de session : owner, ce qui a été fait, ce qui est bloqué, next step.

### 2026-05-19 — Martin
- Configuration repo GitHub `martinbtxg/adv-eps`, identité Gilac, push HTTPS via PAT Keychain
- Choix techniques actés (FastAPI + React + Postgres, webapp source de vérité)
- Style guide GILAC sauvegardé en mémoire durable
- Livraison du prototype HTML statique (table + kanban, 14 commandes mockées)
- PR #1 ouverte → en attente de validation UX

### 2026-05-20 — Martin
- Création de ce `CLAUDE.md` (corrigé du précédent fichier basé sur une mauvaise interprétation M&A)
- **Next step :** retour UX sur le prototype, puis spec Phase 0 (infra VM, modèle de données, plan de migration Notion)
