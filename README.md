# Namma-Kathey — Project guide for everyone

This document explains what the app is, which technologies it uses, how authentication and data work, where the important files live, and how to open and run the project on a **new computer**. You do **not** need prior Android experience to follow the “how to run” section step by step.



## 1. What is this app?

**Namma-Kathey** (see `app_name` in `app/src/main/res/values/strings.xml`) is an **Android storytelling app** focused on **Karnataka**: users explore a **district map**, read **hero stories** (English / Kannada in the UI), take **quizzes**, and collect **badges** for perfect quiz scores. Progress is stored **on the device** and tied to either a **signed-in account** or **guest mode**.

There is **no separate website or custom server** that this repo talks to for stories; story content is **bundled** with the app (see [Section 5 — Backend and data](#5-backend-and-data-what-is-not-in-the-cloud)).

---

## 2. Glossary (quick definitions)

| Term | Meaning |
|------|--------|
| **Android** | Google’s mobile operating system. This project builds an app that runs on Android phones and tablets. |
| **Kotlin** | The programming language used for almost all app logic (`*.kt` files). |
| **Jetpack Compose** | Google’s modern toolkit for building **UI in code** (not XML layouts for screens). |
| **Material 3** | Google’s design system; the app uses **Material 3** components (buttons, bars, themes). |
| **Gradle** | The **build system**: it downloads libraries, compiles code, and produces the installable **APK**. |
| **Firebase** | Google’s **Backend-as-a-Service**: here it is used mainly for **Authentication** (who is logged in) and **Analytics**. |
| **Room** | A **local database** library: data is stored in a SQLite file **on the device**. |
| **ViewModel** | Holds screen-related state and survives configuration changes (e.g. rotation). |

---

## 3. UI / UX — what is used for design?

### How the “design” is implemented

- **There is no separate Figma or design file in this repository.** The visual design is expressed **in code** using:
  - **Jetpack Compose** — every main screen is a `@Composable` function.
  - **Material 3** (`androidx.compose.material3`) — `Scaffold`, `TopAppBar`, `NavigationBar`, `Button`, `TextField`, etc.
  - **App theme** under `app/src/main/java/com/example/myapplication/ui/theme/` — colors, typography, shapes (`Color.kt`, `Type.kt`, `Theme.kt`, `Shape.kt`).
- **Icons**: Material Icons Extended (`Icons.Default.Map`, `Icons.Default.List`, etc.).
- **Images**: **Coil** loads remote images for hero pictures (`imageUrl` in data). Some assets are in `app/src/main/res/drawable/` (e.g. splash / auth art).
- **Fonts**: Google Fonts integration (`ui-text-google-fonts`) where used in theme.
- **Strings for localization**: English defaults in `app/src/main/res/values/strings.xml`, Kannada overrides in `app/src/main/res/values-kn/strings.xml`.

### Screen map (user journey)

1. **Splash** → `SplashScreenActivity.kt` (short branded intro, then routes to auth or main).
2. **Sign-in / register** → `AuthActivity.kt` (email, Google, phone OTP, or guest).
3. **Main app** → `MainActivity.kt` with bottom navigation:
   - **Map** → `DistrictMapScreen.kt`
   - **Heroes** → `HeroListScreen.kt`
   - **Badges** → `BadgeRoomScreen.kt`
   - **Story** → `StoryViewerScreen.kt`
   - **Quiz** → `QuizScreen.kt`

---

## 4. Authentication — how does “logging in” work?

### Providers (ways to sign in)

Implemented in **`auth/AuthRepository.kt`** and **`auth/AuthViewModel.kt`**, with the UI in **`AuthActivity.kt`**:

| Method | Technology | Notes |
|--------|------------|--------|
| **Email / password** | Firebase Authentication | Register and sign-in; forgot-password uses Firebase’s reset email. |
| **Google** | Google Sign-In + Firebase | Needs **OAuth Web Client ID** in `default_web_client_id` (see `res/values/strings.xml`) and correct **SHA fingerprints** in the Firebase project. |
| **Phone (SMS OTP)** | Firebase Phone Auth | Often requires **Blaze (billing)** on Firebase for production-scale SMS; errors may mention billing. |
| **Guest** | Local preference only | **`UserSessionStore`** sets guest mode; **no** Firebase user. Progress uses a fixed guest id. |

### Session rules (who can open the main app)

- **`UserSessionStore.kt`** decides if the user may open **`MainActivity`**: either **Firebase user is signed in** **or** **guest mode** is enabled.
- **`MainActivity`** checks `canAccessMainContent()` on start; if false, it sends the user to **`AuthActivity`**.
- **Sign out** clears guest flag, signs out Firebase, signs out Google client, and returns to **`AuthActivity`** (see `StoryViewModel.signOutToAuth` and `UserSessionStore.signOutReturningToAuth`).

### Launcher icon vs language

- **`AndroidManifest.xml`** defines two **activity-alias** entries for English vs Kannada launcher icons (`LauncherIconEn` / `LauncherIconKn`).
- **`auth/AppIconAliasManager.kt`** enables the correct alias (synced on cold start / splash) so the home-screen icon can match language choice.

---

## 5. Backend and data — what is **not** in the cloud

### “Backend” in this project means:

1. **Firebase (remote)**  
   - **Authentication** — identity.  
   - **Analytics** — usage metrics (no custom API in this repo).

2. **On-device “backend”**  
   - **Room database** — `AppDatabase.kt`, file name `namma_kathey.db`.  
   - **Tables**: heroes (`HeroEntity`), per-user progress (`UserProgressEntity` + `UserProgressDao`).  
   - **First launch**: if the DB is empty, **`DatabaseInitializer.kt`** reads **`app/src/main/assets/stories.json`** and seeds heroes (`StoriesSeedParser.kt`).

3. **Static assets** (under `app/src/main/assets/`)  
   - **`stories.json`** — main story content source.  
   - **`karnataka_districts.geojson`** — district boundaries for the map (`KarnatakaGeoJsonLoader.kt` loads this file by name). If you copy the project without this file, the map can fail or appear empty until you restore it.

There is **no** Node/Java/Python server in this repository that you deploy separately.

---

## 6. How to run on a new system (from zero)

### 6.1 What to install

1. **Android Studio** (recent version that supports **compileSdk 35** and **AGP 9.x** — check [Android Studio release notes](https://developer.android.com/studio) if unsure).  
2. During setup, install **Android SDK**, **SDK Platform** for API **35**, and a **device image** (emulator) **or** prepare a **physical phone** with **USB debugging** enabled.

### 6.2 Java version

The app module is configured for **Java / JVM 21** (`jvmToolchain` in `app/build.gradle.kts`). Android Studio usually bundles a compatible JDK; if Gradle errors mention Java version, set the **Gradle JDK** to **21** in Android Studio settings.

### 6.3 Open the project

1. Clone or copy the folder **`storytellingApp`** to the new machine.  
2. Open Android Studio → **Open** → select the folder that contains **`settings.gradle.kts`** (project root).  
3. Wait for **Gradle Sync** to finish (first time downloads dependencies).

### 6.4 Firebase configuration (required for real sign-in)

1. In [Firebase Console](https://console.firebase.google.com/), create or select a project and add an **Android app** whose **package name** matches **`applicationId`** in `app/build.gradle.kts` (currently **`com.nammakathey.app`**).  
2. Download **`google-services.json`** and place it at **`app/google-services.json`** (this path is required by the Google Services Gradle plugin).  
3. In Firebase **Authentication**, enable **Email**, **Google**, and/or **Phone** as you need.  
4. For **Google Sign-In**: add your app’s **SHA-1** and **SHA-256** (debug and release keystores) in Firebase project settings, and put the **Web client ID** into **`default_web_client_id`** in `app/src/main/res/values/strings.xml` (see comment near `google_needs_web_client` in the same file).  
5. Rebuild the app after changing Firebase or strings.

### 6.5 Run

- Click the **Run** ▶ button in Android Studio, or from the project root on Windows:

```text
gradlew.bat installDebug
```

(On Linux/macOS: `./gradlew installDebug`.)

---

## 7. Repository layout (folders)

| Path | Role |
|------|------|
| **`app/`** | The Android application module (code, resources, assets). |
| **`app/src/main/java/.../`** | Kotlin source: UI, ViewModels, data, auth. |
| **`app/src/main/res/`** | XML resources: strings, colors, themes, mipmaps (launcher), drawables. |
| **`app/src/main/assets/`** | Raw files shipped in the APK (`stories.json`, optional `karnataka_districts.geojson`). |
| **`app/build.gradle.kts`** | App dependencies, SDK versions, `applicationId`. |
| **`gradle/libs.versions.toml`** | Central place for **library and plugin versions**. |
| **`settings.gradle.kts`** | Project name and which modules exist (`:app`). |
| **`build.gradle.kts`** (root) | Top-level Gradle plugins. |

---

## 8. Important files and what they do

### 8.1 Entry point and application class

| File | Purpose |
|------|--------|
| **`AndroidManifest.xml`** | Declares app name, theme, permissions (`INTERNET`), activities, launcher **aliases**, and `NammaKatheyApplication`. |
| **`NammaKatheyApplication.kt`** | Runs on app start: creates **Room** database, **`UserSessionStore`**, seeds DB if empty. |
| **`SplashScreenActivity.kt`** | First screen after tap on launcher; splash animation; routes to **Auth** or **Main**. |
| **`MainActivity.kt`** | Main **Compose** UI: `NavHost`, bottom bar, language toggle, sign-out menu. |
| **`AuthActivity.kt`** | Full auth UI (login, register, phone, Google, guest). |

### 8.2 Authentication

| File | Purpose |
|------|--------|
| **`auth/AuthRepository.kt`** | Talks to **FirebaseAuth**, **GoogleSignIn**, **PhoneAuthProvider**; builds Google sign-in intent. |
| **`auth/AuthViewModel.kt`** | UI state for auth flows (routes, loading, errors). |
| **`auth/UserSessionStore.kt`** | Guest vs logged-in, **`ProgressOwner`** uid for Room progress, sign-out behavior. |
| **`auth/AppIconAliasManager.kt`** | Switches launcher icon alias by language. |

### 8.3 Data and database

| File | Purpose |
|------|--------|
| **`data/Hero.kt`** | Data classes for **Hero**, localized text, quiz models (Kotlinx Serialization). |
| **`data/db/AppDatabase.kt`** | Room database definition and version. |
| **`data/db/HeroEntity.kt` / `HeroDao.kt`** | Hero table and queries. |
| **`data/db/UserProgressEntity.kt` / `UserProgressDao.kt`** | Progress per **owner uid** (Firebase uid or guest). |
| **`data/HeroRepository.kt` / `HeroRepositoryRoom.kt`** | Abstraction over Room for the ViewModel. |
| **`data/HeroMappers.kt`** | Maps entities ↔ domain `Hero`. |
| **`data/db/DatabaseInitializer.kt`** | Seeds from **`assets/stories.json`** when DB empty. |
| **`data/db/StoriesSeedParser.kt`** | Parses JSON into `Hero` list. |

### 8.4 Karnataka map

| File | Purpose |
|------|--------|
| **`data/karnataka/KarnatakaGeoJsonLoader.kt`** | Loads GeoJSON from assets, maps legacy district names. |
| **`data/karnataka/KarnatakaDistrictLabels.kt`** | Labels / metadata for map UI. |
| **`data/karnataka/DistrictNameMatch.kt`** | Matching district names from taps / data. |
| **`ui/screens/DistrictMapScreen.kt`** | Compose map screen. |

### 8.5 UI screens and shared UI

| File | Purpose |
|------|--------|
| **`ui/screens/HeroListScreen.kt`** | List of heroes (optionally filtered by district). |
| **`ui/screens/StoryViewerScreen.kt`** | Story reading, narration hooks, navigate to quiz. |
| **`ui/screens/QuizScreen.kt`** | Quiz UI. |
| **`ui/screens/BadgeRoomScreen.kt`** | Badges unlocked from perfect quizzes. |
| **`ui/theme/*`** | Colors, typography, theme wrapper `MyApplicationTheme`. |

### 8.6 State and narration

| File | Purpose |
|------|--------|
| **`viewmodel/StoryViewModel.kt`** | Heroes list, language, progress, quiz state, narration, sign-out. |
| **`narration/NarrationManager.kt`** | Text-to-speech–style narration for stories. |

### 8.7 Preferences

| File | Purpose |
|------|--------|
| **`prefs/AppPrefs.kt`** | Keys for SharedPreferences (e.g. UI language `en` / `kn`). |

---

## 9. “I want to add something” — which file to change?

Use this as a **starting map** (you may touch more than one file for a polished feature).

| Goal | Start here |
|------|------------|
| **Change app display name** | `app/src/main/res/values/strings.xml` (`app_name`). |
| **Add or change English / Kannada UI text** | `values/strings.xml` and `values-kn/strings.xml`. |
| **Change colors / fonts / dark-light theme** | `ui/theme/Color.kt`, `Type.kt`, `Theme.kt`. |
| **Add a new hero or edit stories** | `app/src/main/assets/stories.json` (then bump Room **`version`** in `AppDatabase.kt` if entity shape changes, or uninstall app to reset DB — destructive migration is enabled). |
| **New screen in the main app** | Add composable under `ui/screens/`, register route in **`MainActivity.kt`** `NavHost`. |
| **Change bottom navigation tabs** | **`MainActivity.kt`** (`NavigationBar` + `NavHost` routes). |
| **Change auth UI or add a button** | **`AuthActivity.kt`** (+ possibly **`AuthViewModel.kt`**). |
| **Change how Google / email / phone sign-in works** | **`auth/AuthRepository.kt`** / **`AuthViewModel.kt`**. |
| **Change guest or “who owns progress” rules** | **`UserSessionStore.kt`** + any code reading `owner` in **`StoryViewModel.kt`**. |
| **New database field for heroes** | `HeroEntity.kt`, `HeroDao.kt`, mappers, `StoriesSeedParser.kt`, `DatabaseInitializer.kt`, and **`AppDatabase` version**. |
| **Add a library** | `app/build.gradle.kts` `dependencies { }` and often **`gradle/libs.versions.toml`**. |
| **Change minimum Android version** | `minSdk` in **`app/build.gradle.kts`**. |
| **Change package / Play Store id** | `applicationId` and **`namespace`** in **`app/build.gradle.kts`** (advanced: also Firebase app id and manifest refactors). |

---

## 10. Tests and quality

| Path | Role |
|------|------|
| **`app/src/test/`** | Unit tests (JVM). |
| **`app/src/androidTest/`** | Instrumented tests (run on device/emulator). |

---

## 11. Troubleshooting (common)

| Symptom | Likely cause |
|---------|----------------|
| Gradle sync fails | Missing SDK components or wrong JDK; check Android Studio SDK Manager and Gradle JDK **21**. |
| Google Sign-In fails | SHA keys not in Firebase, or **`default_web_client_id`** empty / wrong. |
| Phone OTP fails | Billing not enabled, wrong phone format, or Firebase phone auth not set up. |
| Map empty / crash on map | Missing or malformed **`assets/karnataka_districts.geojson`**. |
| No heroes after install | **`stories.json`** missing or invalid; check Logcat for `DatabaseInitializer`. |

---

## 12. Summary

**Namma-Kathey** is a **native Android** app built with **Kotlin**, **Jetpack Compose**, and **Material 3**. **Firebase** handles **authentication** (and analytics); **stories and progress** live in a **local Room database** seeded from **`stories.json`**. Use **`MainActivity.kt`** for navigation and main UX, **`AuthActivity.kt`** for login flows, **`assets/stories.json`** for content, and **`app/build.gradle.kts`** + **`google-services.json`** when setting up a **new machine** or **new Firebase project**.

If you maintain this doc when the app grows, add new screens and dependencies under [Section 8](#8-important-files-and-what-they-do) and [Section 9](#9-i-want-to-add-something--which-file-to-change) so the next reader stays oriented.
