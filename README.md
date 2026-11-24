# Not(ion) : ton Notion perso, privé et gratuit

Un clone de l'interface Notion qui tient dans **un seul fichier `index.html`**.
Pas d'IA, pas de collaboration, pas de serveur à gérer. Tes pages sont stockées en ligne sur **ton propre** projet Supabase, gratuit et chiffré si tu le souhaites.

> Chacun installe **sa propre copie** : ton dépôt GitHub et ton projet Supabase. Personne d'autre que toi n'a accès à tes données.

---

## Sommaire

1. [Ce que fait l'outil](#1-ce-que-fait-loutil)
2. [Ce qu'il te faut](#2-ce-quil-te-faut)
3. [Étape 1 : créer ton projet Supabase](#3-étape-1--créer-ton-projet-supabase)
4. [Étape 2 : configurer `index.html`](#4-étape-2--configurer-indexhtml)
5. [Étape 3 : mettre en ligne sur GitHub Pages](#5-étape-3--mettre-en-ligne-sur-github-pages)
6. [Étape 4 : premier lancement](#6-étape-4--premier-lancement)
7. [Importer tes pages Notion](#7-importer-tes-pages-notion)
8. [Vérifier que tout marche](#8-vérifier-que-tout-marche)
9. [Au quotidien](#9-au-quotidien)
10. [Sécurité et confidentialité](#10-sécurité-et-confidentialité)
11. [Raccourcis clavier](#11-raccourcis-clavier)
12. [Dépannage](#12-dépannage)

---

## 1. Ce que fait l'outil

- **Éditeur de blocs façon Notion** : texte, titres, listes, to-do, toggles, citations, encadrés, code, tableaux, images, sous-pages et liens entre pages. Le menu `/` et les raccourcis Markdown marchent comme dans Notion.
- **Bases de données** : vues Tableau, Kanban et Liste, avec filtres, tris, propriétés (statut, sélection, dates, cases à cocher…) et ouverture des fiches en panneau latéral.
- **Organisation** : arborescence de pages, favoris, corbeille, recherche plein texte, mode sombre.
- **Stockage** : en ligne sur Supabase (synchro automatique entre postes), dans un fichier sur ton disque, ou seulement dans le navigateur.
- **Chiffrement optionnel** : AES-256 dans le navigateur. Supabase ne voit alors qu'un bloc illisible.

---

## 2. Ce qu'il te faut

- Un compte **GitHub** (gratuit), pour héberger la page.
- Un compte **Supabase** (gratuit), pour stocker tes données.
- **Chrome** ou **Edge** de préférence. Firefox et Safari fonctionnent aussi, mais sans l'option « fichier sur le disque ».
- Environ **15 minutes**.

---

## 3. Étape 1 : créer ton projet Supabase

### 3.1 Créer le projet
1. Va sur [supabase.com](https://supabase.com) → **Start your project** → connecte-toi, par exemple avec GitHub.
2. **New project** : donne-lui un nom (`notion-perso`), un mot de passe de base de données (garde-le, il ne servira pas au quotidien) et la région **Europe (Paris ou Francfort)**.
3. Attends une à deux minutes que le projet soit prêt.

### 3.2 Créer la table
1. Menu de gauche → **SQL Editor** → **New query**.
2. Colle **tout** le script ci-dessous, puis clique sur **Run**.
3. Résultat attendu : `Success. No rows returned`.

```sql
-- Not(ion) : table de stockage (script relançable sans erreur)
create table if not exists public.notion_state (
  user_id uuid primary key default auth.uid() references auth.users(id) on delete cascade,
  payload jsonb not null,
  updated_at timestamptz not null default now()
);
alter table public.notion_state enable row level security;
drop policy if exists "lire ses donnees" on public.notion_state;
drop policy if exists "creer ses donnees" on public.notion_state;
drop policy if exists "modifier ses donnees" on public.notion_state;
create policy "lire ses donnees" on public.notion_state for select to authenticated using (auth.uid() = user_id);
create policy "creer ses donnees" on public.notion_state for insert to authenticated with check (auth.uid() = user_id);
create policy "modifier ses donnees" on public.notion_state for update to authenticated using (auth.uid() = user_id) with check (auth.uid() = user_id);
revoke all on table public.notion_state from anon;
grant usage on schema public to authenticated;
grant select, insert, update on table public.notion_state to authenticated;
```

> Ce script crée une table avec **une ligne par utilisateur**. Les règles RLS garantissent que chaque compte ne peut lire et modifier que **sa** ligne. Tu peux le relancer autant de fois que tu veux.

### 3.3 Créer ton compte utilisateur
1. **Authentication → Users → Add user → Create new user**.
2. Saisis ton email et un mot de passe : ce sont tes identifiants de connexion à Not(ion).
3. Coche **Auto Confirm User** si l'option est proposée.

### 3.4 Fermer les inscriptions
**Authentication → Sign In / Providers → Email** : désactive **Allow new users to sign up**, puis enregistre.
Personne d'autre ne pourra créer de compte sur ton projet.

### 3.5 Récupérer l'URL et la clé publique
Le plus simple : bouton **Connect** en haut du tableau de bord. Sinon :

| Information | Où la trouver | Forme |
| --- | --- | --- |
| **Project URL** | Project Settings → Data API | `https://abcdefgh.supabase.co` |
| **Clé publique** | Project Settings → API Keys | `sb_publishable_…` ou, sur les anciens projets, `eyJ…` (anon public) |

> ❌ **N'utilise jamais** la clé `service_role` ni la `secret key` : elles donnent tous les droits sur la base et ne doivent jamais apparaître dans une page web.
> ✅ La clé **publishable / anon**, elle, est faite pour être visible : ce sont les règles RLS qui protègent tes données.

---

## 4. Étape 2 : configurer `index.html`

Ouvre `index.html` dans un éditeur de texte (Bloc-notes, VS Code…), ou plus tard directement sur GitHub avec l'icône ✏️.
Cherche `CLOUD_URL` avec **Ctrl+F**, puis remplis les deux lignes :

```js
const CLOUD_URL='https://abcdefgh.supabase.co';   // ta Project URL
const CLOUD_ANON='sb_publishable_xxxxxxxxxxxx';   // ta clé publique
```

Garde bien les apostrophes `'…'` et le `;` en fin de ligne. Enregistre.

> Tu peux aussi laisser ces lignes vides et saisir l'URL et la clé dans **Paramètres → Stockage en ligne**. Mais il faudra les ressaisir à chaque vidage du cache du navigateur : mieux vaut les mettre dans le fichier.

---

## 5. Étape 3 : mettre en ligne sur GitHub Pages

1. Sur [github.com](https://github.com) → **New repository** → nom : `notion-perso` → **Public** → Create.
2. **Add file → Upload files** : glisse `index.html`, `README.md` et `.gitignore` → **Commit changes**.
   Sur Mac, `.gitignore` est un fichier caché. S'il n'apparaît pas, crée-le sur GitHub (Add file → Create new file) avec deux lignes : `*.json` et `*.zip`.
3. **Settings → Pages** → Source : **Deploy from a branch** → branche `main`, dossier `/ (root)` → **Save**.
4. Attends une à deux minutes, avec la coche verte dans l'onglet **Actions**. Ton app est en ligne sur :
   `https://<ton-pseudo>.github.io/notion-perso/`

> ⚠️ Le dépôt est **public** : n'y mets **jamais** de sauvegarde `.json` ni d'export Notion. Le `.gitignore` les bloque si tu passes par Git, mais pas si tu glisses un fichier à la main sur GitHub.

**Astuce** : dans Chrome ou Edge, menu ⋮ → **Caster, enregistrer et partager → Installer la page en tant qu'application**. Tu peux ensuite l'épingler à la barre des tâches.

---

## 6. Étape 4 : premier lancement

1. Ouvre ton adresse GitHub Pages.
2. Connecte-toi avec l'email et le mot de passe créés à l'étape 3.3.
3. L'indicateur en haut à droite affiche **☁ En ligne (non chiffré)** : la synchro fonctionne.
4. **Recommandé** : **Paramètres → Sécurité → Définir un mot de passe**. Tes données sont alors chiffrées avant de partir sur Supabase, et l'indicateur passe à **☁ En ligne**.

> 🔑 **Mot de passe de chiffrement perdu = données irrécupérables.** Personne, pas même toi, ne peut les déchiffrer. Range-le dans un gestionnaire de mots de passe.

Le numéro de version est affiché en haut à gauche, sous « Not(ion) ».

---

## 7. Importer tes pages Notion

### Depuis Notion
1. Dans Notion : **Paramètres → Général** (ou Espace de travail) → **Exporter tout le contenu**.
2. Format : **Markdown & CSV**. Coche **Inclure les sous-pages** et **Créer des dossiers pour les sous-pages**.
3. Tu reçois un fichier `.zip`, par téléchargement ou par email.

### Dans Not(ion)
**Importer**, en bas de la barre latérale, → choisis le `.zip`. Tu peux aussi le glisser sur la fenêtre.
Les pages, sous-pages, bases de données (CSV) et images sont recréées. Tout est traité dans ton navigateur.

> Une base exportée par Notion arrive avec une vue Tableau et, si elle contient un statut ou une sélection, une vue Kanban. Les types de propriétés sont devinés à partir des valeurs : vérifie-les et ajuste-les via l'en-tête de colonne.

---

## 8. Vérifier que tout marche

| Test | Comment | Résultat attendu |
| --- | --- | --- |
| Données en base | Supabase → Table Editor → `notion_state` | 1 ligne, `updated_at` change à chaque modification |
| Chiffrement | Ouvre la cellule `payload` | `"enc": 1` et contenu illisible |
| Vidage du cache | Ctrl+Maj+Suppr → Cookies + Cache → rouvrir l'app | Connexion → mot de passe → tout est là |
| Nouveau poste | Fenêtre InPrivate (Ctrl+Maj+N) ou autre PC | Mêmes données après connexion |
| Synchro | App ouverte à deux endroits, modifier d'un côté | Visible de l'autre côté en 30 s maximum |
| Hors ligne | Couper le Wi-Fi, modifier, rallumer | « ☁ Hors ligne » puis « ☁ En ligne » |

---

## 9. Au quotidien

- **À l'ouverture** : connexion Supabase (seulement après un vidage du cache), puis mot de passe de chiffrement.
- **Indicateur ☁** :
  - **En ligne** : tout est à jour.
  - **Synchro…** : envoi en cours.
  - **Hors ligne** : tes modifications sont gardées en local et renvoyées automatiquement.
- **Sauvegarde** : **Paramètres → Exporter une sauvegarde**, une fois par semaine. Range le `.json` sur ton OneDrive, **pas** sur GitHub.
- **Deux postes** : évite d'éditer en même temps des deux côtés, car c'est la dernière modification enregistrée qui gagne.
- **Mettre à jour l'app** : remplace `index.html` sur GitHub, puis **Ctrl+F5**. Pense à recoller tes deux lignes `CLOUD_URL` et `CLOUD_ANON`.

### Limites du plan gratuit Supabase
- Un projet **inactif pendant environ une semaine est mis en pause**. Rien n'est perdu : bouton **Restore project** dans le tableau de bord. Une utilisation régulière suffit à l'éviter.
- **Pas de sauvegarde automatique** de la base : d'où l'export hebdomadaire.
- 500 Mo de base : très largement suffisant, des milliers de pages tiennent dans quelques Mo.

---

## 10. Sécurité et confidentialité

- **Réseau** : la page ne peut contacter que `*.supabase.co`. Toute autre connexion est bloquée par sa politique de sécurité (CSP).
- **Isolation** : chaque utilisateur ne voit que sa propre ligne (RLS). Le rôle anonyme n'a aucun droit.
- **Chiffrement** : PBKDF2 (310 000 itérations) + AES-256-GCM, fait dans le navigateur. Le mot de passe ne quitte jamais ton poste.
- **Code** : un seul fichier lisible, sans dépendance ni appel externe. Tu peux l'auditer.
- **Données professionnelles** : avant d'y mettre des informations de l'entreprise, vérifie que c'est compatible avec la politique de ta DSI. Le chiffrement limite fortement le risque, mais les données sortent tout de même des outils de l'entreprise.

---

## 11. Raccourcis clavier

Sur Mac, remplace `Ctrl` par `⌘`. La liste complète est accessible avec le bouton **?** en bas à droite ou `Ctrl+/`.

| Action | Raccourci |
| --- | --- |
| Rechercher / aller à une page | `Ctrl+P` ou `Ctrl+K` |
| Nouvelle page | `Ctrl+Alt+N` |
| Barre latérale | `Ctrl+\` (AZERTY : `Ctrl+AltGr+8`) ou bouton ☰ |
| Mode sombre | `Ctrl+Maj+L` |
| Menu des blocs | `/` |
| Lier une page | `[[` ou `@` |
| Gras / italique / souligné / barré | `Ctrl+B` / `I` / `U` / `Ctrl+Maj+S` |
| Code en ligne / lien | `Ctrl+E` / `Ctrl+K` |
| Indenter / désindenter | `Tab` / `Maj+Tab` |
| Cocher la to-do, ouvrir le toggle | `Ctrl+Entrée` |
| Dupliquer / déplacer un bloc | `Ctrl+D` / `Ctrl+Maj+↑↓` |
| Sélectionner le bloc | `Échap` |
| Annuler / rétablir | `Ctrl+Z` / `Ctrl+Maj+Z` |
| Transformer le bloc | `Ctrl+Maj+0…9` (Mac : `⌘+⌥+0…9`) |

**Markdown en début de ligne** : `#` `##` `###` (titres), `-` (puces), `1.` (numéros), `[]` (to-do), `>` (toggle), `"` (citation), ` ``` ` (code), `---` (séparateur).
**En ligne** : `**gras**`, `*italique*`, `` `code` ``, `~barré~`.

**Kanban** : glisse les cartes d'une colonne à l'autre. Déplace les colonnes avec **‹ ›** ou via le menu **•••** de la colonne.

---

## 12. Dépannage

| Message ou symptôme | Cause | Solution |
| --- | --- | --- |
| `permission denied for table notion_state` | Droits manquants sur la table | Relance le script SQL de l'étape 3.2 |
| `policy … already exists` | Ancienne version du script | Utilise le script de ce guide, qui est relançable |
| « Email ou mot de passe incorrect » | Utilisateur inexistant ou non confirmé | Authentication → Users : vérifie le compte et coche Auto Confirm |
| « ☁ Hors ligne » qui persiste | Réseau, ou projet Supabase en pause | Vérifie la connexion ; dans Supabase, clique sur **Restore project** |
| Écran de connexion jamais affiché | `CLOUD_URL` ou `CLOUD_ANON` vide ou mal collé | Vérifie les apostrophes, ou passe par Paramètres → Stockage en ligne |
| Bandeau « page ouverte dans un aperçu » | Fichier ouvert depuis une pièce jointe | Ouvre l'app depuis ton adresse GitHub Pages |
| Une mise à jour n'apparaît pas | Cache du navigateur | Attends la coche verte dans Actions, puis **Ctrl+F5** ; vérifie la version en haut à gauche |
| Barre latérale introuvable | Barre fermée | Bouton ☰ en haut à gauche, ou `Ctrl+AltGr+8` |
| Mot de passe de chiffrement oublié | – | Irrécupérable : restaure une sauvegarde exportée |

**Voir l'erreur exacte** : clique sur l'indicateur ☁ en haut à droite. La dernière erreur est affichée dans **Paramètres → Stockage en ligne**.

---

*Not(ion) · un fichier HTML, zéro dépendance, tes données restent à toi.*
