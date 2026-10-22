# SHIVAM OS setup guide

## 1. Open and build

1. Install Android Studio (Ladybug or newer) with Android SDK 35 and JDK 17.
2. `File → Open` and select this `SHIVAM_OS_Android_Project` folder.
3. Let Gradle sync. If the wrapper jar is missing, run `gradle wrapper --gradle-version 8.9` in a terminal
   from the project folder, or let Android Studio generate it.
4. Run `Build → Make Project`, or `./gradlew assembleDebug`.
5. Run tests with `./gradlew testDebugUnitTest`.

## 2. Connect an AI provider (no rebuild needed)

Open the app → **Optimus** tab → gear icon → **AI provider**.

| Provider | Choose | Base URL | Model examples |
|---|---|---|---|
| OpenAI | OpenAI-compatible | `https://api.openai.com/v1` | `gpt-4o-mini` |
| OpenRouter, Groq, Together, LM Studio, Ollama (`/v1`) | OpenAI-compatible | the provider's `/v1` base | any model the provider lists |
| Anthropic | Anthropic Messages | `https://api.anthropic.com` | `claude-3-5-haiku-latest` |

1. Enter the API key and model name.
2. Tap **Save**, then **Test connection**. The status shows *Connected*, or the reason for failure:
   - *Invalid API key (401)*: re-enter the key.
   - *Access denied (403)*: the key lacks access to that model.
   - *Endpoint or model not found (404)*: check the base URL and model name.
   - *Rate limit or quota exceeded (429)*: check billing or wait.
   - *Network error*: check the connection and base URL.

Local servers (for example Ollama at `http://10.0.2.2:11434/v1` from an emulator) work with `http://`. A key
is still required by the form; use any placeholder string for local servers that ignore it.

## 3. Key security

- The key is typed into the app at runtime and stored in the app's private DataStore. It is not in the source,
  the APK, or the Git history. Do not put real keys in `gradle.properties`, `BuildConfig`, or `strings.xml`.
- **APK-embedded keys can always be extracted.** A key compiled into an app ships to every device that installs
  it. This project never embeds keys for that reason.
- Use a key with a monthly spending limit. Revoke it in the provider console if the device is lost.
- DataStore is not hardware-encrypted in this version. For stronger protection, move the key to the Android
  Keystore with an encrypted wrapper.

## 4. Permissions

| Permission | Why | When asked |
|---|---|---|
| Microphone | Voice commands | First tap on JARVIS speak, or in Settings |
| Notifications | Alarm notifications (Android 13+) | When adding an alarm, or in Settings |
| Exact alarms | Precise alarm timing (Android 12+) | Opened from the Chronexa banner or Settings |
| Internet | AI provider calls | Install time |
| Receive boot completed | Restore alarms after reboot | Install time |

Denying a permission disables only the matching feature. The app shows the reason where it happens.

## 5. Using the features

- **Commands** (typed or spoken): "turn on flashlight", "open YouTube", "alarm 6:30 am", "note: buy milk",
  "remember that my exam is Monday". Flashlight and app-open ask for confirmation: say "yes" or "haan" to run it.
- **Routines** (Control → Command Studio): create a trigger phrase. Saying it runs the action.
- **Voice Lab** (Settings): pick language, speed and pitch, then Preview. If a language is missing, install its
  voice data in Android Settings → Text-to-speech.
- **Study Pilot**: choose a mode, enter a topic or paste text, Run, then save useful output as a note.
- **Privacy** (Settings): clear history, turn memory off, view diagnostics (no key included), or delete all
  local data.

## 6. Troubleshooting

| Symptom | Fix |
|---|---|
| Gradle sync fails on Kotlin or Compose versions | Use Android Studio Ladybug or newer, and keep the versions in `build.gradle.kts` as they are |
| Alarm rings late | Settings → Chronexa banner → allow exact alarms; disable battery optimisation for SHIVAM OS |
| No speech recognition | Install Google app / speech services; Speech recognition needs network on many devices |
| Voice speaks in the wrong language | Check Voice Lab language and install that language's TTS data |
| Flashlight missing | The device has no back flash, or the camera is in use by another app |
