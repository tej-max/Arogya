# MyHealth (com.myhealth.personalassistant) - Phase 1 + 2

Offline-first Android health app. Kotlin, Jetpack Compose, Material 3, Room, DataStore, AlarmManager.
English + Telugu, light/dark/system theme, 5 accent colours.

## STATUS: WRITTEN, NOT YET COMPILED

These files were written in an environment with no Android SDK, no Gradle and no network, so
**nothing here has been compiled or run on a device.** Expect a few compile errors on the first
build; send them back and they get fixed. Do not rely on the reminders for real medication until
you have tested them on your own phone (checklist below).

## Build the debug APK

1. Install Android Studio (Koala 2024.1 or newer; a JDK 17+ is bundled).
2. File > Open > select this `MyHealth` folder. Let Gradle sync (needs internet once, to download
   Gradle 8.9, Android SDK platform 35 and the libraries). Accept any prompt to install missing SDK parts.
3. Build > Build Bundle(s) / APK(s) > Build APK(s).
   Output: `app/build/outputs/apk/debug/app-debug.apk`
   Terminal alternative: `./gradlew assembleDebug`. If there is no `gradlew` yet, Android Studio
   creates it on sync, or run `gradle wrapper --gradle-version 8.9` once.
4. Install: copy the APK to the phone, open it, allow "install from this source".
   Or with USB debugging on: `adb install -r app/build/outputs/apk/debug/app-debug.apk`

A release APK needs your own signing key: Build > Generate Signed Bundle / APK.

## What exists (Phase 1 + 2)

- Foundation: Room DB v1, DataStore settings, theme + accents, EN/TE resources, bottom navigation,
  first-run setup (name + language).
- Medicines: add / edit / detail / discontinue / resume / delete (with confirmation; history is kept).
  Fields: name, category (8 defaults + your own), dosage, tablets per dose, before/after/with food,
  instructions, notes, prescribed by, photo, start/end date, once/twice/three/custom times,
  every day / every N days / chosen weekdays.
- Reminders: one exact alarm per medicine pointing at its next dose; notification with
  Taken / Snooze / Skip; every action logged with a timestamp; restored after reboot, app update,
  time or time-zone change; vibration on/off; works with the app closed.
- Home: greeting, date, today overview, timeline, mark Taken/Skip, permission warnings.
- History: 30 days back + 3 days ahead, filters, adherence %.
- Nothing is ever marked Taken automatically. A past dose with no action shows as Missed.

## NOT built yet

Checkups + calendar + measurements (Phase 3), food planner + Sunday reminder (Phase 4),
fitness / water / walking (Phase 5), web search button + history (Phase 6), backup / restore /
export, custom notification sounds, per-channel settings, app lock, records vault, family profiles,
widget, splash screen, PDF/CSV export (Phase 7+).
The Food and Fitness tabs show a "not built yet" notice instead of fake screens.

## Design notes

- Schedule fields live on the Medicine row (simplified from a separate MedicationSchedule table).
- Missed = past dose, no recorded action, more than 60 minutes ago (`DoseCalc.GRACE_MS`).
- Resume sets the start date to today so the paused gap is not counted as missed.
- If the phone was off at dose time, no late notification is posted after boot; that dose shows
  as Missed in Home and History.
- Exact alarms: Android 12+ needs the "Alarms and reminders" permission. The app shows a warning
  card with a Settings button. Without it, alarms fall back to inexact and may be delayed.
- Some manufacturers (Xiaomi, Oppo, Vivo, Samsung and others) kill background apps. Exclude MyHealth
  from battery optimisation (Profile > Permissions). Delivery cannot be guaranteed on every device.
- The database never uses destructive migration. When the schema changes, bump the version in
  `AppDatabase` and add a Migration in `AppContainer`.

## Phase 2 test checklist (do this on a real phone)

1. Add a medicine for 2 minutes from now. Close the app. A notification arrives with 3 buttons.
2. Tap Snooze: it returns after the snooze time. Tap Taken: it shows as Taken in History.
3. Add two medicines at the same time: two separate notifications, neither replaces the other.
4. Reboot with a reminder pending: it still fires. Change the time zone: the next dose follows local time.
5. Edit a medicine time: the old alarm no longer fires. Discontinue: no more notifications.
6. Delete a medicine: a confirmation is shown and its history rows remain.
7. Switch to Telugu in Profile: all screens and the next notification are in Telugu.
8. Deny notification permission: Home shows the warning card.
