# StylePalette — Android App

<p align="center">
  <img alt="Kotlin" src="https://img.shields.io/badge/Kotlin-2.0.21-7F52FF?logo=kotlin&logoColor=white">
  <img alt="Android" src="https://img.shields.io/badge/Android-minSdk%2026%20%7C%20target%2036-3DDC84?logo=android&logoColor=white">
  <img alt="Firebase" src="https://img.shields.io/badge/Firebase-Auth%20%7C%20Firestore%20%7C%20Storage-FFCA28?logo=firebase&logoColor=black">
</p>

Native Kotlin client for **StylePalette**. Users sign up, take a selfie for personal color analysis, browse a community outfit feed, upload their own outfits with per-garment colors, favorite outfits, and filter the feed to items that match their personal color palette.

For the overall project (including the FastAPI backend), see the [root README](../README.md).

## Features

- Email/password authentication (Firebase Auth)
- Selfie-based personal color analysis with an editable trait-confirmation step (calls the StylePalette backend)
- Personal seasonal palette (Winter / Summer / Autumn / Spring) with power & neutral color swatches, stored per user
- Community outfit feed with search-by-vibe, search-by-store/garment, and "Match My Palette" filtering
- Feed de-clustering/interleaving so one user's posts don't dominate consecutive slots
- Outfit upload with a per-garment (top, bottom, jacket, shoes, jewelry, sunglasses, bag) color picker (common swatches, extended HSV grid, or color wheel)
- Favorites/wishlist (like an outfit to save it)
- Profile screen with palette summary, sampled trait colors, and a grid of the user's own outfits
- Firebase Storage-backed images for outfits and profile pictures
- Firebase App Check (Play Integrity in release, Debug provider in development)

## Tech Stack

- Kotlin 2.0.21, XML layouts (AndroidX AppCompat, Material Components, ConstraintLayout) — no Jetpack Compose
- Firebase BoM 34.8.0: Auth, Firestore, Storage, Analytics, App Check (Debug + Play Integrity)
- Glide 4.16.0 (image loading), Lottie 6.7.1 (animations)
- [Dhaval2404 ColorPicker](https://github.com/Dhaval2404/ColorPicker) 2.3 for garment color selection
- Plain `HttpURLConnection` for calling the FastAPI backend (no Retrofit/OkHttp dependency)
- JUnit for unit tests

## Project Structure

```
app/
├── app/
│   ├── build.gradle.kts              # applicationId, minSdk 26 / targetSdk 36, dependencies
│   ├── google-services.json          # Firebase project config
│   └── src/
│       ├── main/
│       │   ├── AndroidManifest.xml
│       │   ├── java/com/example/myapplication/
│       │   │   ├── App.kt                    # Application class, Firestore/App Check setup
│       │   │   ├── BaseActivity.kt           # Shared bottom-nav wiring
│       │   │   ├── LoginActivity.kt / RegisterActivity.kt
│       │   │   ├── MainActivity.kt           # Outfit feed, search, palette filter
│       │   │   ├── UploadOutfitActivity.kt   # Create outfit + per-garment color picker
│       │   │   ├── OutfitDetailActivity.kt   # View/delete a single outfit
│       │   │   ├── FavoritesActivity.kt      # Liked outfits
│       │   │   ├── ProfileActivity.kt        # Palette summary + user's outfits
│       │   │   ├── BackendApi.kt             # HTTP client for the FastAPI backend
│       │   │   ├── FeedOrderMixer.kt         # Feed interleaving algorithm
│       │   │   ├── FeedPaletteMatcher.kt     # Palette-based outfit filtering
│       │   │   ├── adapters/OutfitAdapter.kt
│       │   │   ├── ui/OutfitColorPicker.kt
│       │   │   ├── repository/               # AuthRepository, OutfitRepository (Firestore/Storage)
│       │   │   └── models/                   # Outfit, OutfitRgb, PersonalPalette, PaletteSwatch, User, ...
│       │   └── res/                          # Layouts, drawables, strings, themes
│       ├── androidTest/                      # Instrumented test scaffold
│       └── test/                             # JUnit unit tests (e.g. FeedOrderMixerTest)
├── gradle/libs.versions.toml
├── firestore.rules                           # Firestore security rules
└── build.gradle.kts / settings.gradle.kts
```

## Architecture

- **UI**: Activity-per-screen with XML layouts; `BaseActivity` wires up the shared bottom navigation (Home / Add / Favorites / Profile).
- **Firebase**: `AuthRepository` and `OutfitRepository` wrap Firebase Auth, Firestore (`users`, `outfits`, per-user `favorites` subcollection), and Storage (`outfits/{id}.jpg`, `profile_images/{uid}.jpg`).
- **Backend integration**: `BackendApi.kt` calls the StylePalette FastAPI backend over `HttpURLConnection`:
  - `POST /api/v1/analysis/selfie` — multipart selfie upload during registration, returns estimated skin/eye/hair traits and a seasonal palette
  - `POST /api/v1/analysis/palette-from-traits` — sends user-confirmed trait labels, returns the final seasonal palette and RGB swatches
  - The base URL is read from the `backend_analysis_base_url` string resource.
- **Palette matching**: `FeedPaletteMatcher` filters outfits whose garment RGB values fall inside the user's power/neutral swatch ranges; `FeedOrderMixer` reorders the feed so consecutive posts from the same author are avoided where possible.
- **Color picking**: `OutfitColorPicker` wraps the Dhaval2404 ColorPicker library to assign a color per garment when uploading an outfit.

## Getting Started

**Requirements:** Android Studio, JDK 11, Android SDK (minSdk 26 / targetSdk 36)

1. Open this `app/` directory in Android Studio.
2. The project includes a Firebase `google-services.json` — replace it with your own Firebase project's config for Auth/Firestore/Storage if you're not using the bundled one.
3. Point the app at your backend by editing the `backend_analysis_base_url` string resource in `app/src/main/res/values/strings.xml` — e.g. `http://10.0.2.2:8000` for the Android emulator, your machine's LAN IP for a physical device, or an `ngrok` tunnel URL (ngrok URLs are auto-detected and sent with a bypass header).
4. Sync Gradle and run the `app` module on an emulator or device. The backend (see [`/backend`](../backend)) must be running and reachable at the configured URL for selfie analysis to work.

## Testing

```bash
./gradlew test                 # JVM unit tests, e.g. FeedOrderMixerTest
./gradlew connectedAndroidTest # instrumented tests (requires a connected device/emulator)
```
