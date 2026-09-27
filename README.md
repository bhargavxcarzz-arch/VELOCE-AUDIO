# VELOCE — Premium Automotive-Inspired Music Player

VELOCE is a fully native Android music player (Kotlin + Jetpack Compose +
Media3) with:

- Local-only music library, built entirely from your device's MediaStore
  (no bundled songs, no downloads)
- Background playback via a `MediaSessionService`, with system media
  notification controls (play/pause, next/previous, artwork) and Bluetooth
  media-button support
- Home / Now Playing / Library / Playlists / Equalizer / Themes / Settings
  screens
- System `Equalizer` + `BassBoost` audio effects, with presets and manual bands
- Room-backed playlists and favorites (the song files themselves always stay
  in MediaStore — nothing is duplicated, so moved/deleted files are handled
  by rescanning)
- A dark, glassmorphism "liquid glass" UI with an automotive accent, matching
  the reference designs — no speedometers, steering wheels, or fake driving
  controls, just a premium phone music player

## Phone-only build workflow (no PC required)

You don't need Android Studio or a computer to get an installable APK. This
project builds itself in the cloud via **GitHub Actions**, triggered entirely
from your phone:

### 1. Get this code onto GitHub from your phone
Easiest path — Termux + git:
1. Install **Termux** from F-Droid (not the outdated Play Store version).
2. In Termux:
   ```
   pkg install git
   git config --global user.email "you@example.com"
   git config --global user.name "YourName"
   ```
3. Copy this project folder onto your phone's storage (e.g. via the file you
   downloaded from this chat), then in Termux:
   ```
   cd /sdcard/Download/VELOCE   # wherever you extracted it
   git init
   git add .
   git commit -m "Initial VELOCE project"
   ```
4. Create an empty repo on github.com named `veloce` (the GitHub app or
   mobile site both work — no PC needed).
5. Push it:
   ```
   git remote add origin https://github.com/<your-username>/veloce.git
   git branch -M main
   git push -u origin main
   ```
   (GitHub will prompt for a Personal Access Token instead of a password —
   create one under GitHub → Settings → Developer settings → Personal access
   tokens, entirely from the mobile browser.)

### 2. Let GitHub Actions build the APK
The moment you push, `.github/workflows/build-apk.yml` runs automatically in
GitHub's cloud (no Gradle wrapper jar needed — the workflow installs Gradle
itself). Watch it under the repo's **Actions** tab, from the GitHub app or
mobile browser.

### 3. Download the APK — from your phone
When the build finishes (a few minutes), open that workflow run and download
the `veloce-debug-apk` artifact — it's a zip containing `app-debug.apk`.
Unzip it (any Android file manager / "Files by Google" can unzip), then tap
the APK to install (you may need to allow "install unknown apps" for your
browser or file manager in Android Settings).

### Alternative: AIDE (a phone-native Android IDE)
If you'd rather build directly on-device without GitHub, **AIDE** (on the
Play Store) can open this same project folder and compile it locally on your
phone. Import the project's `app/` folder as an existing Gradle/Android
project. Build times will be slower than a cloud runner, and very old/low-RAM
phones may struggle, but it works entirely offline.

## Project structure
```
app/src/main/java/com/veloce/music/
  MainActivity.kt          entry point, permission request, theme host
  VeloceApp.kt              Application class
  data/                     Song/Album/Artist models, MediaStore scanner,
                             Room database (playlists/favorites), repository
  playback/                 PlaybackService (Media3 MediaSessionService),
                             PlayerViewModel (UI <-> MediaController bridge)
  equalizer/                EqualizerController (system Equalizer/BassBoost)
  ui/                       Navigation + screens (Home, Library, Playlists,
                             NowPlaying, Equalizer, Themes, Settings) +
                             the GlassCard "liquid glass" component + theme
```

## Permissions
Only `READ_MEDIA_AUDIO` (Android 13+) / `READ_EXTERNAL_STORAGE` (older),
plus the foreground-service and notification permissions background
playback requires. Nothing else — no location, camera, mic, contacts, or
network permissions.

## What's implemented vs. what to refine
This is a complete, working first build: scanning, background playback with
notification controls, playlists, favorites, equalizer, themes, and settings
all function. A few things worth polishing as you iterate:
- The Now Playing visualizer is a lightweight animated placeholder, not a
  true FFT analysis of the audio (Android's `Visualizer` API can be wired in
  later if you want a reactive one).
- The equalizer's vertical sliders use a simple rotation trick; on some
  screen sizes you may want to tune their sizing.
- Album art embedded in files not indexed by MediaStore's `albumart`
  provider may need a fallback (e.g. reading ID3/embedded art directly) on
  some devices/OEM skins.
