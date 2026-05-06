# Spécifications Générales : Grimocette

## 1. Vision du Produit
Une application de gestion de recettes "Offline-First" permettant le versionnage (fork) de recettes. L'utilisateur peut posséder ses recettes, créer des variantes de recettes existantes, et partager le tout au sein de groupes privés ou avec la communauté.

---

## 2. Spécifications Fonctionnelles

### A. Authentification
*   **Provider :** Firebase Authentication — Google Sign-In (Android) et Sign in with Apple (iOS).
*   **Obligation stores :** Sign in with Apple est **obligatoire sur l'App Store** dès lors que l'app propose un login via un provider tiers (règle Apple). Les deux providers doivent donc être supportés pour déployer sur Android et iOS.
*   **Flux :** À la première ouverture, l'utilisateur est invité à se connecter. L'app reste utilisable en mode local (sans compte) pour créer et consulter des recettes, mais le partage et la synchronisation nécessitent un compte.
*   **Côté serveur :** Ktor vérifie les tokens Firebase JWT sur chaque requête authentifiée via le SDK Firebase Admin.

### B. Gestion des Recettes & Versioning
*   **Création de Recette :** Titre, ingrédients (nom, quantité, unité), étapes, temps de préparation, et photo.
*   **Le "Fork" (Variante) :** Possibilité de copier une recette existante. La nouvelle recette conserve un lien (`parentId`) vers l'originale.
*   **Ajustement Dynamique :** Calcul automatique des quantités en fonction du nombre de convives défini par l'utilisateur.

### C. Expérience "Cuisine" (Optimisée Tablette)
*   **Dual-Pane UI :** Ingrédients fixes à gauche (avec cases à cocher), étapes défilantes à droite.
*   **Mode Stepper :** Vue focus étape par étape avec police agrandie.
*   **Mains Sales :** Blocage de la mise en veille de l'écran lorsque le mode cuisine est actif.
*   **Interface Aérée :** Design épuré, typographie sans-serif, boutons larges pour une manipulation facile.
*   **Mode téléphone :** Navigation par BottomBar via Voyager.
    Le mode cuisine doit passer d'une vue liste (verticale) à un Stepper horizontal (paysage) pour maximiser la lisibilité des instructions.
    Les boutons d'action (Valider une étape) doivent être assez grands pour être cliqués avec le pouce.

### D. Social & Groupes
*   **Espaces Privés :** Création de groupes (Famille, Amis) pour partager des recettes hors du flux public.
*   **Flux Communautaire :** Découverte des recettes publiques et système d'abonnement (Follow) aux utilisateurs.
*   **Modération :** Système de notation (1-5 étoiles), commentaires et bouton de signalement d'abus.

### E. Stratégie de Stockage (Offline-First)
*   **Mes Recettes :** Sauvegarde locale permanente de toutes les recettes créées, modifiées ou mises en favoris (images incluses).
*   **Cache de Consultation :** Sauvegarde automatique des **100 dernières recettes consultées**. Au-delà, suppression automatique des plus anciennes (LRU).

---

## 3. Spécifications Techniques

### A. Stack Technologique
*   **Langage :** Kotlin (Multiplatform)
*   **Frontend :** Compose Multiplatform (Android / iOS)
*   **Backend :** Ktor Server (Netty)
*   **Authentification :** Firebase Authentication (client) + Firebase Admin SDK (serveur)
*   **Client HTTP :** Ktor Client (KMP) — moteur OkHttp sur Android, Darwin sur iOS
*   **Base de données Client :** Room ORM (KMP)
*   **Base de données Serveur :** PostgreSQL + Exposed ORM (Style DSL)
*   **Stockage Objet :** API S3-compatible — MinIO (preprod) / Scaleway Object Storage (prod)
*   **Injection de Dépendances :** Koin
*   **Navigation :** Voyager
*   **Images :** Coil3 (KMP)

### B. Environnements

Le projet comporte trois environnements, différenciés par variables d'environnement et build flavors Gradle / schemes Xcode.

| Variable | Preprod | Prod |
|----------|---------|------|
| `BASE_URL` | `https://preprod.kitchengit.fr` (Cloudflare Tunnel → k3s maison) | `https://kitchengit.fr` (Scaleway) |
| `S3_ENDPOINT` | `http://minio:9000` | `https://s3.fr-par.scw.cloud` |
| `S3_BUCKET` | `kitchengit-preprod` | `kitchengit-prod` |
| `FIREBASE_PROJECT` | projet Firebase de dev | projet Firebase de prod |

**Review store :** un troisième flavor `review` pointe vers un backend éphémère (VPS léger) déployé uniquement pendant la fenêtre de validation Apple/Google, sans toucher à la config Cloudflare de preprod.

#### Infrastructure Preprod (k3s maison)
*   Ktor server en pod k3s, exposé via Cloudflare Tunnel
*   PostgreSQL en StatefulSet + PersistentVolumeClaim sur le NAS
*   MinIO en pod k3s pour le stockage objet S3-compatible

#### Infrastructure Prod (Scaleway)
*   Ktor server déployé sur Scaleway (Kubernetes Kapsule ou instance simple)
*   PostgreSQL managé Scaleway
*   Scaleway Object Storage (S3-compatible, région `fr-par`)

### C. Stockage des Images

Une seule implémentation `S3StorageService` dans le serveur Ktor, configurée par variables d'environnement. Le même code fonctionne avec MinIO (preprod) et Scaleway Object Storage (prod) sans modification.

```kotlin
// Injecté via Koin
data class S3Config(
    val endpoint: String,
    val bucket: String,
    val accessKey: String,
    val secretKey: String
)
```

*   **Côté client :** Les images des recettes persistées (Mes Recettes + favoris) sont sauvegardées dans le stockage local de l'appareil. Les images du cache LRU sont supprimées avec les recettes associées.
*   **Côté serveur :** Upload vers le bucket S3-compatible. L'URL publique de l'objet est stockée dans `RecipeDTO.imageUrl`.

### D. Distribution & Validation Stores

| Environnement | Android | iOS | Review requise ? |
|---------------|---------|-----|-----------------|
| Preprod | Firebase App Distribution | TestFlight (testeurs internes) | Non |
| Review store | Google Play — piste interne | TestFlight (testeurs internes) | Non |
| Prod | Google Play — production | App Store | Oui |

Les builds preprod et review sont générés via des flavors Gradle distincts (`preprod`, `review`, `production`) et ne sont jamais soumis en production.

### E. Architecture du Projet

Le projet est divisé en trois modules principaux :
1.  **`:shared`** : Contient les DTOs (Data Transfer Objects) sérialisables et la logique métier commune (calculs de quantités).
2.  **`:composeApp`** : Code UI partagé et base de données locale **Room**. Les DAOs doivent utiliser des `@Query` pour les sélections complexes.
3.  **`:server`** : API Ktor utilisant **Exposed DSL** pour interagir avec PostgreSQL.

### F. Modèle de Données
```kotlin
// Dans le module :shared
@Serializable
data class IngredientDTO(
    val name: String,
    val quantity: Double,
    val unit: String
)

@Serializable
data class RecipeDTO(
    val id: String,
    val parentId: String?,      // ID de la recette source pour le versioning
    val authorId: String,
    val title: String,
    val baseServings: Int,
    val ingredients: List<IngredientDTO>,
    val steps: List<String>,
    val imageUrl: String?,
    val isPublic: Boolean,
    val groupId: String?
)
```

---

## 4. Schéma de Base de Données & Flux

### Système de Versioning
*   **Table `Recipes`** : Contient une colonne `parent_id` (Self-referencing).
*   **Logique de Fork** : Une nouvelle entrée est créée avec un nouvel `id`, mais le `parent_id` pointe vers l'original. Cela permet de reconstruire l'arbre généalogique d'une recette.

### Synchronisation Offline
1.  L'application vérifie la connectivité via le client Ktor.
2.  Si connecté : Envoi/Réception des données et mise à jour de la base **Room**.
3.  Si déconnecté : Lecture seule dans Room. Les créations sont stockées localement avec un flag `isSynced = false` et envoyées au serveur dès le retour du réseau.