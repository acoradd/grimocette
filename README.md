# Grimocette

Application de gestion de recettes **Offline-First**. Testez, variez, et partagez vos recettes au sein de groupes privés ou avec la communauté. Accompagnée de **Grimo**, la mascotte grimoire.

Projet **Kotlin Multiplatform** ciblant Android, iOS et un backend serveur.

---

## Structure du projet

| Module | Rôle |
|--------|------|
| `:shared` | DTOs sérialisables et logique métier commune (calcul de quantités) |
| `:composeApp` | UI Compose Multiplatform + base de données Room (Android / iOS) |
| `:server` | API Ktor (Netty) + Exposed DSL + PostgreSQL |

---

## Build & Run

### Android

```shell
# macOS / Linux
./gradlew :composeApp:assembleDebug

# Windows
.\gradlew.bat :composeApp:assembleDebug
```

### Serveur

```shell
# macOS / Linux
./gradlew :server:run

# Windows
.\gradlew.bat :server:run
```

### iOS

Ouvrir le dossier [`/iosApp`](./iosApp) dans Xcode et lancer depuis là, ou utiliser la configuration de run de l'IDE.

---

## Stack technique

- **Langage :** Kotlin Multiplatform
- **UI :** Compose Multiplatform
- **Navigation :** Voyager
- **HTTP Client :** Ktor Client (OkHttp / Darwin)
- **Base de données locale :** Room ORM (KMP)
- **Base de données serveur :** PostgreSQL + Exposed DSL
- **DI :** Koin
- **Images :** Coil3

---

Voir [SPEC.md](./SPEC.md) pour les spécifications complètes.
