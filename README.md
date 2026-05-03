# Frog-Generator
Frog Image app for Lucas
# Random Frog Generator

A tap-the-button Android app for kids: each tap fetches a real, research-grade frog photograph from the [iNaturalist](https://www.inaturalist.org/) API, complete with the frog's species name.

- **Min Android version:** 10 (API 29)
- **UI:** Jetpack Compose, kid-friendly pond theme
- **Image source:** iNaturalist v1 public API — no API key, no cost
- **Permissions:** `INTERNET` only (no location, no storage, no analytics)

---

## Build & install (step by step)

You said you've already installed Android Studio and created a GitHub repo for the assistant-dashboard project, so I'll skip the basics and call out only the things specific to this project.

### 1. Open the project in Android Studio

1. Launch Android Studio.
2. **File → Open** and select the `Random Frog Generator` folder (the one containing `settings.gradle.kts`, not its parent).
3. Click **OK / Trust Project** when prompted.

Android Studio will start a **Gradle Sync**. First sync downloads ~250 MB of dependencies and the Gradle 8.9 distribution. Sit tight — 3-10 minutes on a fresh machine.

> **If Sync complains that `gradle-wrapper.jar` is missing:** click the link in the error to "Generate Gradle Wrapper", or run `gradle wrapper` once from a terminal in the project folder. Modern Android Studio (Koala / Ladybug / Meerkat) usually regenerates it automatically.

> **If Sync complains about Android SDK location:** go to **File → Project Structure → SDK Location** and point it at your installed Android SDK. Android Studio then writes a `local.properties` file (gitignored).

You may also be prompted to install **SDK Platform 35** (Android 15) — accept; that's our `compileSdk`.

### 2. Connect your son's phone

1. On the phone: **Settings → About phone**, tap **Build number** 7 times. This unlocks Developer Options.
2. **Settings → System → Developer options**, enable:
   - **USB debugging**
   - **Install via USB** (some OEMs)
3. Plug the phone into your computer. The phone will pop up an **"Allow USB debugging?"** dialog — accept (and check "Always allow" so you don't have to do it every time).

The device should now show in Android Studio's device dropdown (top toolbar, looks like a phone icon).

### 3. Run on the phone

With the phone selected in the device dropdown, hit the green **▶ Run 'app'** button.

Android Studio compiles, installs, and launches. You should see the **🐸 Random Frog** title, a loading spinner for ~1 second, then a frog photo appears with the species name. Tap **New Frog!** for another.

### 4. Build a standalone APK (optional — to keep on the phone without your computer)

1. **Build → Build Bundle(s) / APK(s) → Build APK(s)**
2. When done, click the **locate** link in the notification — it opens `app/build/outputs/apk/debug/app-debug.apk`.
3. Email/AirDrop/USB-copy that APK to the phone, then tap it to install. The phone may need **"Install unknown apps"** allowed for whichever app you used to transfer it.

> The debug APK is signed with your machine's debug keystore — fine for personal use. For Play Store, you'd build a signed release AAB; out of scope here.

---

## How it works (one paragraph)

`MainActivity` hosts a single Compose screen (`FrogScreen`) bound to `FrogViewModel`. The ViewModel calls `FrogRepository.fetchRandomFrog()`, which hits iNaturalist's `/v1/observations` endpoint with `taxon_id=20979` (Anura — the order containing all frogs), filtered to research-grade observations with photos. To add randomness, the repository picks a random page within iNat's pagination cap (max page = 300) and then a random observation from that page's results. The square-thumbnail URL is rewritten to the `/large.` variant for a phone-sized image. Coil loads the image; `AnimatedContent` cross-fades it in. The button is debounced (button-mashing during a fetch is ignored).

## Project layout

```
Random Frog Generator/
├── app/
│   ├── build.gradle.kts            # app module deps + Android config
│   └── src/main/
│       ├── AndroidManifest.xml     # INTERNET permission, MainActivity
│       ├── java/com/randomfrog/app/
│       │   ├── MainActivity.kt     # Compose UI: frog card, button, animations
│       │   ├── FrogViewModel.kt    # StateFlow<FrogUiState>
│       │   └── data/
│       │       ├── INaturalistApi.kt    # Retrofit interface
│       │       ├── Models.kt            # Serializable DTOs + UI model
│       │       ├── FrogRepository.kt    # Random-page fetch logic
│       │       └── NetworkModule.kt     # OkHttp + Retrofit wiring
│       └── res/
│           ├── values/              # strings, colors, themes
│           ├── drawable/            # cartoon frog launcher icon
│           └── mipmap-anydpi-v26/   # adaptive launcher icons
├── build.gradle.kts                 # root
├── settings.gradle.kts
├── gradle/
│   ├── libs.versions.toml           # version catalog
│   └── wrapper/gradle-wrapper.properties
└── README.md
```

## Customising

A few easy tweaks if your son wants something different:

- **Different animal.** Change `TAXON_ID_ANURA = 20979` in `INaturalistApi.kt`. iNat taxon IDs: turtles 39532, lizards 26036, salamanders 26718, butterflies 47224, sharks 47273.
- **Bigger button text or different colours.** All in `MainActivity.kt`'s `BouncyButton` and `PondGradient` definitions.
- **Add a frog fact.** Hit `/v1/taxa/{id}` after fetching the observation — that endpoint returns a `wikipedia_summary` field. Easy follow-up.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Sync fails: "Could not resolve `androidx.compose.bom`" | Check `gradle.properties` has `android.useAndroidX=true` (it does) and that you're online. Re-sync. |
| Sync fails on `kotlin.plugin.compose` | You're on Kotlin <2.0. The version catalog pins Kotlin 2.0.21 — re-sync to download. |
| App installs but shows "Oops! Couldn't catch a frog." every time | The phone has no internet, or iNaturalist is rate-limiting (very unusual at 1 req per tap). Check Wi-Fi. |
| App crashes immediately | Run `adb logcat \| grep -i randomfrog` (or just look at Android Studio's **Logcat** tab while it's connected) and send me the stacktrace. |

## Credits

Frog photos and species data are courtesy of contributors to [iNaturalist](https://www.inaturalist.org/), licensed under various Creative Commons licenses (each photo's attribution is displayed under the image in the app).

---

*Built for your son. Have fun spotting frogs.* 🐸
