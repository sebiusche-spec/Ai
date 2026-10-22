# SHIVAM OS

An Android AI assistant built with Kotlin, Jetpack Compose, Material 3, MVVM, Room, DataStore and OkHttp.
It is an app you run on your own device, with your own AI provider key.

> **Build status:** the source was written and statically reviewed in a sandbox without the Android SDK or
> network access, so it has **not been compiled or run**. See *Verification status* below. Open it in Android
> Studio, sync Gradle, and run `./gradlew testDebugUnitTest assembleDebug` first.

## Modules and status

| Module | What works in this build | Known limits |
|---|---|---|
| **Optimus Core** (chat) | Chat with an OpenAI-compatible or Anthropic provider, conversation memory (last 20 turns), stored "remember" facts, English/Hindi/Hinglish replies via system prompt, command routing for device actions | The model does not call tools itself. Device actions use a local, deterministic command router. Streaming responses are not implemented |
| **Universal API settings** | Provider choice, base URL, model name, API key, Test connection, status and error messages, no key in the APK | Only two wire formats: OpenAI-compatible chat completions and Anthropic Messages. Other formats need a new adapter in `ai/AiClient.kt` |
| **JARVIS voice** | Speech-to-text (platform recognizer), TTS replies, animated waveform driven by microphone level, Stop and Interrupt controls, typed fallback | Works only while the app is open. No wake word and no background microphone (see Limits) |
| **Chronexa alarms** | One-time and repeating alarms (weekday mask), labels, Snooze 10 min and Dismiss from the notification, reboot restore via `BootReceiver`, exact alarms when allowed | No full-screen alarm UI. Exact timing needs the user to allow exact alarms on Android 12+. Alarms on Android 14+ may be dismissible by the user from the notification |
| **Morning companion** | Spoken greeting with the date and the next alarm (Morning brief button) | No weather. Real weather needs a weather API key and is not wired up. No scheduled automatic morning speech |
| **Phone control** | Flashlight on/off (back torch, with confirmation), open an installed app by name (with confirmation), open Settings panels, in-app countdown timers | Cannot toggle Wi-Fi, Bluetooth or data silently (Android forbids this for ordinary apps). Opens the panel instead |
| **Command Studio** | Routines with a trigger phrase and one action: flashlight on/off, open app, speak text, ask Optimus. Test-run button | One action per routine. No time or location triggers |
| **Voice Lab** | Language (en-IN, en-US, en-GB, hi-IN), speech speed, pitch, preview, engine status | Voice list depends on the device's TTS engine. Missing language data must be installed in Android settings |
| **Study Pilot** | Explain, summarize, quiz and multi-day study plan using the configured AI; save outputs as notes; delete notes | Notes are stored locally only |
| **Futuristic UI** | Dark theme with cyan/violet neon, glass cards, animated AI core, bottom navigation | Phone-first layouts. Tablet layout is not optimized |
| **Background system** | Notification channels, AlarmManager, BroadcastReceivers | No foreground service for voice. No WorkManager jobs (none needed by the current features) |
| **Settings & privacy** | Permission status and requests, memory toggle, clear conversations, delete all local data, diagnostics report (no key included) | Error reporting is local only; nothing is uploaded |

## Architecture

```
app/src/main/java/com/shivam/os/
  ShivamApp.kt            application container (lazy singletons) + notification channels
  MainActivity.kt         single activity, Compose host
  ai/                     provider types, adapters (AiClient), URL helpers, command router, chat repository
  data/                   Room entities/DAOs/database, DataStore settings
  alarm/                  AlarmScheduler, AlarmReceiver (fire/snooze/dismiss), BootReceiver, next-trigger maths
  control/                PhoneActions (flashlight, app launch, settings panels) returning explicit results
  voice/                  Speaker (TTS), VoiceListener (STT + RMS level)
  viewmodel/              AssistantCore (shared input pipeline) and one ViewModel per feature
  ui/                     Compose screens, components, theme, navigation
app/src/test/             JVM unit tests (alarm timing, URL building, command routing, error mapping)
```

Every input (typed chat, voice, routine trigger) goes through `AssistantCore.handle()`. Actions that change
device state wait for an explicit "yes" before running.

## Build

Requirements: Android Studio Ladybug (2024.2) or newer, JDK 17, Android SDK 35.

```bash
# first time only, if the gradle wrapper jar is missing:
gradle wrapper --gradle-version 8.9
./gradlew testDebugUnitTest
./gradlew assembleDebug        # app/build/outputs/apk/debug/
```

See **SETUP.md** for API key configuration, permissions and troubleshooting.

## Verification status

| Check | Result |
|---|---|
| Source files written and reviewed (imports, braces, resource references, package/directory match) | Done |
| Gradle build (`assembleDebug`) | **Not run.** No Android SDK or Gradle in the authoring sandbox |
| Unit tests (`testDebugUnitTest`) | **Not run.** Tests are written for alarm timing, URL building, command routing and error mapping |
| Instrumented / device tests | Not written |
| APK | **Not produced.** Build it locally with the steps above |

Expect a few compile fixes on the first Android Studio build (for example import tweaks after dependency
resolution). The architecture and API usage follow the stable public APIs of the pinned versions.

## Limits you should know about

- No always-on background voice listening. Android restricts background microphone use, and a foreground
  service with a microphone type needs a visible notification and a user-started session.
- No universal device control. Apps cannot silently change Wi-Fi, Bluetooth, mobile data, volume profiles,
  or install/uninstall apps.
- Exact alarms are subject to user permission and battery optimisation. Alarms are not guaranteed to be
  precise on every device.
- The API key is stored on the device in DataStore. It is not hardware-encrypted in this version. Anyone who
  can extract the app's data from a rooted device, or a backup if backups were enabled, could read it (backups
  are disabled in the manifest). Never embed a key in the APK, because APK contents can be extracted.
