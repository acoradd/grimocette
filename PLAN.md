# Plan de développement — Grimocette

Approche **mobile-first**, par lots progressifs. Le serveur et le social arrivent après.
Le code est structuré dès le départ pour accueillir les lots futurs (champs `isSynced`, `parentId`, routes déclarées en WIP, TODOs explicites).

---

## Lot 1 — Fondations ⬜
- [ ] Nettoyage du boilerplate KMP (Greeting, Platform)
- [ ] DTOs & modèles dans `:shared` (`RecipeDTO`, `IngredientDTO`, `UserDTO`, `GroupDTO`)
- [ ] Build flavors `preprod` / `production` dans `:composeApp`
- [ ] Setup Koin DI (modules skeleton)
- [ ] Navigation Voyager — toutes les routes déclarées, écrans futurs en WIP

## Lot 2 — Authentification ⬜
- [ ] Dépendances Firebase Auth
- [ ] Google Sign-In (Android)
- [ ] Sign in with Apple (iOS)
- [ ] Profil utilisateur stocké en local (Room)
- [ ] Écran onboarding / login
- [ ] Mode local sans compte (accès limité)

## Lot 3 — Recettes CRUD ⬜
- [ ] Room DB + DAOs (schéma complet : `isSynced`, `parentId` présents dès maintenant)
- [ ] Création de recette (titre, ingrédients, étapes, photo locale)
- [ ] Liste des recettes (Mes Recettes)
- [ ] Écran détail recette
- [ ] Fork / variante (local uniquement — `parentId` stocké, TODO sync serveur)
- [ ] Ajustement dynamique des quantités selon le nombre de convives

## Lot 4 — Expérience Cuisine ⬜
- [ ] Mode cuisine téléphone : liste verticale → stepper horizontal (paysage)
- [ ] Mode cuisine tablette : dual-pane (ingrédients gauche + étapes droite)
- [ ] Wake lock (blocage mise en veille)
- [ ] Cases à cocher ingrédients
- [ ] Boutons d'action larges (navigation étape par étape)

## Lot 5 — Offline & Cache ⬜
- [ ] Cache LRU des 100 dernières recettes consultées (images incluses)
- [ ] Favoris persistants
- [ ] Détection connectivité (préparée pour la sync — TODO Lot 6)

---

## Lots futurs

### Lot 6 — Serveur & Sync ⬜
- [ ] Setup Ktor server + Exposed DSL + PostgreSQL
- [ ] Firebase Admin JWT middleware
- [ ] S3StorageService (MinIO preprod / Scaleway prod)
- [ ] Routes API recettes, images, utilisateurs
- [ ] Sync offline (`isSynced = false` → envoi au retour réseau)

### Lot 7 — Social & Groupes ⬜
- [ ] Groupes privés (Famille, Amis)
- [ ] Flux communautaire + recettes publiques
- [ ] Follow utilisateurs
- [ ] Notation (1-5 étoiles), commentaires
- [ ] Signalement d'abus

---

## Décisions techniques structurantes

| Sujet | Décision |
|-------|----------|
| Auth | Firebase Auth — Google (Android) + Apple (iOS) |
| DB locale | Room ORM (KMP) |
| DB serveur | PostgreSQL + Exposed DSL |
| Stockage images | S3-compatible : MinIO (preprod) / Scaleway Object Storage (prod) |
| DI | Koin |
| Navigation | Voyager |
| Images | Coil3 |
| HTTP Client | Ktor Client (OkHttp / Darwin) |
| Environments | `preprod` (k3s maison) · `review` (VPS éphémère) · `production` (Scaleway) |
| Distribution | Firebase App Distribution + TestFlight interne (sans review store) |
