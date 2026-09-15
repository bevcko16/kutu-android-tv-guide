# Android TV Optimization + Kutu Mirror + Kutu Home
## Master execution guide for Claude Code

> **Read this entire file before doing anything.**
>
> This guide describes a workflow developed and fully tested on a **Xiaomi Mi Box 3 running Android TV 9**. Do not assume the connected device is identical. Inspect the actual Android TV / Google TV / Android box first and adapt package names, ABIs, Android-version requirements and HOME-launcher behaviour.
>
> The human has already completed the prerequisite:
> - Developer Options / debugging is enabled.
> - ADB is installed.
> - `adb devices` shows the TV/box as `device`.
>
> The human may also provide a background image. **If an image is supplied with this guide, use that exact image as the Kutu Home background. Do not redraw or replace it.**

---

# 0. Non-negotiable safety rules

1. Start read-only. Do not change the TV before the baseline and restore infrastructure exist.
2. Never use root, bootloader unlocking, fastboot flashing, Magisk or custom firmware.
3. Never modify `/system`.
4. Never uninstall a system package.
5. System debloat must use only `pm disable-user --user 0 <package>`.
6. Every package changed by this project must have an exact `pm enable --user 0 <package>` undo.
7. Preserve packages already disabled before this project. Restore only what this project changed.
8. Do not clear app data or caches as an optimization technique.
9. Do not clear or modify `com.android.providers.tv`.
10. Do not blindly copy Xiaomi package names to another manufacturer.
11. Preserve unless the human explicitly says otherwise:
   - Wi-Fi/networking
   - Bluetooth and physical remote
   - Android audio
   - on-screen keyboard/input
   - DRM/Widevine
   - Google Play Services
   - Play Store
   - Chromecast / Google Cast
   - Google Assistant / microphone voice search
   - installed streaming apps
12. Do not change HDMI-CEC as part of this project.
13. Before disabling a stock HOME launcher, prove the replacement works and create a tested rollback path.
14. When ADB drives remote key events, tell the human **HANDS OFF** and do not interleave physical-remote input.
15. Do not turn ADB/debugging off until all work, restore scripts, final tests and the final report are complete.
16. Never expose signing keys, passwords or keystores in git.
17. Prefer the smallest reversible change. If package purpose or dependency risk is unclear, leave it enabled.

---

# 1. Establish host/device variables and project directory

Detect the host OS.

Create a working directory:
- macOS/Linux: `~/tv-debloat`
- Windows: `%USERPROFILE%\tv-debloat` or equivalent

Identify the connected ADB target with:

```bash
adb devices
```

If exactly one target is `device`, use it. If multiple devices exist, ask the human which one to use.

Use explicit targeting for consequential commands:

```bash
adb -s <DEVICE_SERIAL> shell ...
```

Do not embed a private LAN IP into scripts.

Create:
- `BASELINE-SUMMARY.md`
- `DEBLOAT-LOG.md`
- `RESTORE-ALL.sh` on Unix-like systems, or Windows equivalent
- evidence folders for each phase

The restore script must exist before the first system change.

---

# 2. Phase 1: complete READ-ONLY baseline

Do not reboot, kill processes or change settings.

Collect and save raw outputs for:

## Device identity
- `getprop`
- manufacturer/model
- Android version/API level
- build number/date
- security patch
- CPU/SoC
- ABI(s)
- 32/64-bit userland
- GPU/OpenGL
- display resolution/density

## Memory
- `/proc/meminfo`
- `dumpsys meminfo`
- zRAM/swap
- top memory users
- PSS of major processes

## Storage
- `df -h`
- `/data`
- largest app-data consumers where readable

## Packages
- all packages
- system packages
- third-party packages
- currently disabled packages
- package paths
- installer/source where available

## Running state
- processes
- services
- foreground services
- wakelocks
- CPU snapshot
- unusually active idle processes

## Launcher
- currently resolved HOME
- all HOME candidates
- intent priorities where observable
- Leanback launchers
- already-installed third-party launchers

## Important TV subsystems
Identify packages/components for:
- Bluetooth remote
- Android TV Remote
- TV input
- HDMI/CEC
- Chromecast
- Google Play Services
- Play Store
- keyboard/input
- audio
- Wi-Fi/networking
- DRM/Widevine
- Assistant/voice
- updater
- manufacturer analytics
- manufacturer home/content services
- manufacturer phone-remote/discovery
- recommendations/content feeds

## Network exposure
Record listening TCP/UDP ports and map them to processes where possible.

## Existing settings
Record, but do not change:
- debugging/ADB state
- animation scales
- CEC state
- unusual prior changes

Write `BASELINE-SUMMARY.md` with:
1. device spec
2. RAM/storage
3. current HOME
4. package counts
5. major resident processes with PSS/CPU
6. potential optimization candidates:
   - A: likely low-risk
   - B: usage-dependent
   - C: critical / do not touch
7. uncertainties needing investigation

Be conservative.

---

# 3. One usage checkpoint before debloat

Show the human:
- installed user-facing media/streaming apps
- conditional features such as YouTube Music, TalkBack, screensaver/Ambient mode, calendar sync, manufacturer phone remote, manufacturer content feeds and analytics

Ask:

> Which installed apps do you actively use, and do you use voice search, TalkBack, YouTube Music, screensavers/Ambient mode, calendar sync or the manufacturer's phone-remote feature?

Default to **preserve** when unsure.

Regardless of the answer, preserve Assistant, Chromecast, Bluetooth, networking, DRM, keyboard and Play Store unless explicitly requested otherwise.

---

# 4. Restore infrastructure BEFORE the first modification

Snapshot:
- every package enabled/disabled state
- current HOME resolution
- current preferred HOME where observable

Build `RESTORE-ALL` so every future project disable is added automatically.

It must:
1. re-enable only packages disabled by this project
2. later restore the original stock HOME if changed
3. be idempotent where practical

Maintain `DEBLOAT-LOG.md` with:
- timestamp
- package
- reason
- exact command
- exact undo
- test result

---

# 5. Phase 2A: conservative first debloat

Use the actual baseline and usage answers.

## Xiaomi Mi Box 3 reference only

The original first batch was:

```text
com.miui.tv.analytics
tv.alphonso.alphonso_eula
com.google.android.feedback
com.google.android.tv.bugreportsender
com.android.printspooler
com.android.dreams.basic
com.google.android.youtube.tvmusic
com.google.android.marvin.talkback
com.google.android.syncadapters.calendar
com.google.android.backdrop
```

**Do not apply this list blindly to another device.**

Even on the same Mi Box model, verify:
- package exists
- label/purpose matches
- package is currently enabled
- the human does not use the feature

Do not touch yet:
- stock launcher
- recommendations
- PatchWall/manufacturer feeds
- MiLink/manufacturer remote service
- `com.android.providers.tv`
- Assistant
- Chromecast
- Play Services/Store
- updater/download service unless separately justified

For each approved candidate:

```bash
adb -s <DEVICE> shell pm disable-user --user 0 <PACKAGE>
```

Verify package state immediately.

After the batch:
1. verify state
2. reboot
3. wait for Android boot completion
4. verify state again
5. diff against pre-change snapshot

If firmware unexpectedly re-enables something, diagnose instead of changing unrelated components.

Ask the human to test:
- physical remote
- HOME
- voice search
- streaming apps they use
- audio
- Wi-Fi
- Chromecast

Stop if anything breaks.

---

# 6. Check whether native AirPlay already exists

Before building, keep manufacturer discovery services enabled and check whether the TV already advertises:

```text
_airplay._tcp
_raop._tcp
```

If native Apple Screen Mirroring already works reliably, report it before building Kutu Mirror.

If not, continue.

---

# 7. Kutu Mirror: audit upstream

Use:

```text
https://github.com/jqssun/android-airplay-server
```

Record the exact upstream commit.

Audit:
- licence/attribution
- Gradle/manifest
- minSdk/targetSdk
- ABI support
- native libraries
- OpenSSL download/source
- FFmpeg
- Media3/ExoPlayer
- Compose/Hilt
- analytics/telemetry/ads/billing/accounts/cloud
- remote endpoints
- service lifecycle
- boot behaviour
- AirPlay/RAOP discovery
- H.264 path
- mirrored audio path
- HEVC/HLS/PiP/music features
- wakelocks/multicast locks

Do not trust it only because it is open source.

If Android build tools are missing, ask permission before downloading JDK/SDK/NDK/CMake/Gradle.

If approved:
- prefer official/trusted sources
- keep toolchain in project dir where practical
- pin Gradle SHA-256
- verify OpenSSL against trusted/official SHA-256

Do not install upstream APK binaries.

---

# 8. Build final Kutu Mirror directly

Create:

```text
Package: local.kutu.mirror
Application name: Kutu Mirror
AirPlay receiver name: Kutu
```

Adapt ABI/minSdk/targetSdk to the actual device.

Original tested target:
- Android 9 / API 28
- armeabi-v7a
- H.264 only
- max 60 fps

Do not force those values on incompatible hardware.

For newer Android, comply with foreground-service and notification requirements rather than copying Android 9 manifest assumptions.

## Keep
- AirPlay screen mirroring
- `_airplay._tcp`
- `_raop._tcp`
- H.264
- AAC-LC/AAC-ELD as required
- hardware MediaCodec H.264 decode where supported
- efficient Android audio output
- iPhone/iPad native Screen Mirroring
- macOS native Screen Mirroring

## Remove if not needed by that path
- HEVC on old/unsupported devices
- HLS player
- FFmpeg
- ALAC
- ExoPlayer/Media3
- Compose
- Hilt
- PiP
- music/artwork/DACP UI
- debug overlays
- analytics/telemetry/ads/billing/accounts/cloud

Do not sacrifice mirroring audio.

---

# 9. Kutu Mirror lifecycle/power

Requirements:

1. Start after boot once activated.
2. Advertise `Kutu` while awake/on LAN.
3. Multicast lock only as needed for discovery.
4. No permanent partial CPU wakelock while idle.
5. Acquire mirroring wakelock only for real session.
6. Release on every session end.
7. Video decoder only when needed.
8. Release audio decoder/output after final session when safely possible.
   - Original final build still had ~0.39% idle CPU from AAC polling.
   - If safe to release/recreate, do it and test.
   - If reliability suffers, report instead of forcing the change.
9. Losing discovery while the box is actually asleep is acceptable.

---

# 10. Prevent known Kutu Mirror privacy/UX bugs from the start

Final production `MirrorActivity`:
- no MAIN
- no LAUNCHER
- no LEANBACK_LAUNCHER
- `android:exported="false"`
- excluded/auto-removed from Recents where supported
- absent from launcher lists
- opened internally only during genuine mirroring

## Critical network-trigger rule

A bare TCP connection to port 7000 must not open UI.

These must not open MirrorActivity:
- port scan
- TCP connect/disconnect
- `/info`
- discovery
- negotiation that never reaches active mirroring

Open the Activity only once the receiver enters genuine mirroring, matching the tested final behaviour of launching from the equivalent of `onMirrorRunning(true)` rather than generic connection initialization.

If MirrorActivity binds and neither mirroring nor a genuine PIN UI is active, finish immediately.

## Stale-frame prevention

On every termination path:
- Stop Mirroring
- Mac disconnect
- BACK disconnect
- network loss
- decoder error
- service teardown
- Activity finish

do complete cleanup:
- stop/flush/release video decoder
- stop/release audio decoder/output
- blank output to black before teardown when needed
- detach/release Surface/SurfaceTexture
- destroy retained frame state
- save no bitmap/frame/screenshot cache
- destroy/hide SurfaceView outside active mirroring
- finish Activity
- release mirroring wakelock

A later session/Activity must never show the previous sender's final frame.

---

# 11. Receiver control and update behaviour

Create a tiny `ReceiverControlActivity` only for starting the service.

Requirements:
- no launcher category
- no visible UI
- `Theme.NoDisplay` or equivalent
- starts receiver service
- immediately finishes
- accepts no arbitrary commands

It may be exported because Kutu Home is a separate package, but it must only start the receiver.

Do not export MirrorActivity.

Use a non-exported receiver for:
- `BOOT_COMPLETED`
- `MY_PACKAGE_REPLACED`

After Kutu Mirror updates, the receiver must return without requiring a full reboot.

---

# 12. Kutu Mirror security/build

Final release:
- release-signed
- not debuggable
- `allowBackup=false`
- private storage
- no saved screen/audio content
- no unnecessary exported components
- no analytics/ads/cloud
- no unnecessary remote networking beyond local AirPlay

Original Android 9 permissions were:
- INTERNET
- CHANGE_WIFI_MULTICAST_STATE
- WAKE_LOCK
- RECEIVE_BOOT_COMPLETED
- FOREGROUND_SERVICE

Use minimum required permissions for the actual Android version.

PIN support may remain available but default **off** to match the original setup. Warn that with PIN off, another device on the same trusted LAN can attempt to mirror.

Generate a local release signing key outside git and tell the human to back it up.

---

# 13. Install/test Kutu Mirror

Before install report:
- upstream commit
- local fork commit
- APK path/size/SHA-256
- package/version
- minSdk/targetSdk
- ABI(s)
- permissions
- exported components
- native libs/dependencies
- signing cert SHA-256

Install only after audit succeeds.

For a newly installed/stopped package, start `ReceiverControlActivity` via ADB rather than adding a launcher tile just for first launch.

Verify:
- service running
- port 7000 listening
- `_airplay._tcp` = Kutu
- valid `_raop._tcp`
- no launcher icon
- no crash loop
- no idle mirroring wakelock

---

# 14. Mandatory real AirPlay tests

Automated network probes do not replace these.

## iPhone/iPad
Ask human:
1. open recognizable content with sound
2. Control Centre → Screen Mirroring → Kutu
3. verify portrait aspect-fit
4. rotate/open landscape
5. verify audio
6. Stop Mirroring

Measure/check:
- H.264 hardware decoder if available
- no HEVC on H.264-only build
- audio path
- session RAM/CPU
- wakelock
- return HOME
- receiver re-advertises
- no stale frame

## Mac
Ask human to connect via native macOS Screen Mirroring, verify picture/sound, disconnect, then verify cleanup.

## BACK test
Ask human:
1. start iPhone mirroring
2. press BACK once on TV remote
3. sender disconnects
4. TV returns HOME
5. Kutu re-advertises

## Port-probe regression
With no active mirroring, make harmless port 7000 connections and `/info`-style requests.

Verify:
- zero MirrorActivity launches
- foreground app not interrupted
- receiver still works

If any probe creates a black screen, fix it before continuing.

---

# 15. Build Kutu Home only after Kutu Mirror is stable

Create:

```text
App: Kutu Home
Package: local.kutu.home
```

Architecture:
- plain Java/platform Views preferred
- no ads/recommendations/Play Next
- no analytics/telemetry/cloud/accounts
- no network permission
- no background service
- no wakelock
- no boot receiver
- no database
- no GMS dependency
- avoid AndroidX/Compose unless unavoidable
- essentially zero idle CPU

Final Kutu Home may request only `REQUEST_DELETE_PACKAGES` if needed for Android's uninstall-confirmation UI.

Never silently uninstall.

Set:
- `allowBackup=false`
- `debuggable=false`

---

# 16. Ask which apps belong in initial dock

Do not hard-code the original user's seven apps.

Read launchable TV apps from the actual device.

Prefer Leanback launch intents, with safe normal-launcher fallback.

Ask:

> Which apps do you want in the initial Kutu Home dock, and in what order?

Allow any number.

Seed these only if no saved dock exists. Never overwrite user layout on update.

---

# 17. Use supplied background exactly

If a background image is supplied:
- use that exact image
- do not redraw/reinterpret
- preserve composition
- locally optimize only if visually indistinguishable
- fit actual 16:9 output without stretching
- avoid unnecessary crop
- bundle locally
- prefer density-neutral drawable where appropriate
- no network loading
- no animated wallpaper
- no runtime blur
- no continuous processing

---

# 18. Kutu Home final visual spec

Use the final design, not the discarded prototype.

## Home
- calm upper area
- time/date top-right
- dark navy/charcoal text over light background
- one centered translucent dark-glass shelf low on screen
- icons in one horizontal row
- utility chips below:
  - Ekran Yansıtma
  - Ayarlar
  - Tüm Uygulamalar
- focused app name shown once
- minimal text
- safe overscan margins

## Icons
Do not use inconsistent TV banners.

For each app:
1. prefer TV/Leanback activity icon if better
2. else app icon
3. correctly render adaptive icons
4. normalize to consistent rounded-square cards
5. transparent logos get suitable near-black inner background
6. avoid firmware masks/cropping
7. monogram fallback only if needed

## Focus
Focused tile:
- brightens
- scales ~114%
- lifts
- tasteful accent glow
- label visible

Do not rely on white ring alone.

Use short ~150–180 ms animations.

On weak GPUs, reduce glow/elevation first.

**Resting icons must not be tinted by their focus glow.**

---

# 19. Editable dock/persistence

Use SharedPreferences or equivalent lightweight platform store.

Stable key can be `home/dock_v1`.

Persist:
- ordered package list
- last-known human-readable label for each favourite

Requirements:
- add from All Apps
- remove from Home without uninstalling
- reorder
- survive restart/reboot/APK update
- never reset custom layout

Missing app:
- no crash
- dim unavailable tile
- remembered label if available
- long press can remove

Zero favourites:
- `+` tile to All Apps

Fewer than 7:
- shelf centered/shrinks

More than 7:
- single row
- ~7 visible slots
- horizontal scroll inside shelf
- clip to rounded shelf
- subtle faded edge
- never draw off-shelf icons

---

# 20. Robust physical-remote long press

Handle:
- DPAD_CENTER
- ENTER
- system long-press flag
- repeat events
- system timeout / ~500–550 ms

Do not rely only on `setOnLongClickListener`.

**Long press must never fall through into app launch.**

Verify on real remote.

---

# 21. Context menus

## Home app
- Taşı
- Ana ekrandan kaldır
- Uygulama bilgisi
- Uygulamayı kaldır only for normal uninstallable user app

## All Apps
If absent from Home:
- Ana ekrana ekle
- Uygulama bilgisi
- Uygulamayı kaldır if allowed

If already on Home:
- Ana ekrandan kaldır
- Uygulama bilgisi
- Uygulamayı kaldır if allowed

UI:
- dark glass
- D-pad focus trapped
- BACK closes
- subtle veil
- no square elevation/shadow artifact
- centered wrapped text

Use Android standard App Info and uninstall confirmation.

Never silently uninstall/disable apps.

---

# 22. Move mode

1. long press app
2. choose `Taşı`
3. selected tile lifts
4. LEFT/RIGHT moves
5. OK saves
6. BACK cancels/restores
7. leaving before confirm cancels safely

Hint:

```text
◀ ▶ taşı · OK onayla · GERİ iptal
```

---

# 23. All Apps / Settings / Screen Mirroring panel

## All Apps
List user-facing launchable TV apps.

Exclude:
- Kutu Home
- Kutu Mirror
- internal/non-user-facing components

Use same normalized icon system. Center title/wrapped labels.

## Settings
Use standard Android Settings action first. Detect firmware-specific fallback only if required. Do not hard-code Xiaomi component on non-Xiaomi device.

## Screen Mirroring
Do **not** open MirrorActivity.

Open an in-launcher panel:

```text
Ekran Yansıtma

iPhone veya iPad
Denetim Merkezi → Ekran Yansıtma → Kutu

Mac
Denetim Merkezi → Ekran Yansıtma → Kutu
```

Receiver states:
- green ready
- orange installed but stopped
- neutral installed/state unavailable
- red not installed

If local `/proc/net/tcp[6]` status is readable without network permission, use it. On newer Android where restricted, gracefully show neutral state. Do not add network permission just for status.

If stopped, `Alıcıyı başlat` calls `ReceiverControlActivity`, never MirrorActivity.

Panel must be centered dark glass, no square shadow artifact, BACK closes.

---

# 24. Build/test Kutu Home as NORMAL APP first

Generate a local signing key outside repo and back it up.

Report build metadata/hash/permissions/components/dependencies.

Install Kutu Home.

**Do not make it default HOME yet.**
**Do not disable stock launcher yet.**
**Do not disable recommendations/PatchWall/MiLink yet.**

Launch through Leanback.

Test:
- visual quality
- overscan
- background
- clock/date
- focus on every app
- D-pad
- Settings
- All Apps
- Screen Mirroring panel
- app launches
- BACK
- missing app
- >7 scrolling
- no off-shelf icon drawing
- no translucent shadow rectangle

---

# 25. Hands-off protocol

When sending ADB key events, tell human:

> HANDS OFF: put the remote down until I say this test block is finished.

Do not combine ADB navigation with human remote input.

---

# 26. Mandatory real-remote test

Ask human:
1. short OK opens app
2. hold OK ~1 sec opens context menu, app does not launch
3. BACK closes menu
4. Taşı works
5. BACK cancels move
6. OK saves move
7. App Info opens
8. uninstall confirmation opens on normal user app
9. cancel uninstall

Do not proceed to HOME replacement until this passes.

---

# 27. Persistence tests

Verify:
- custom dock survives reboot
- custom dock survives `adb install -r`
- remembered labels survive update
- Kutu Mirror restarts after its APK update via `MY_PACKAGE_REPLACED`

Use/remove throwaway test app only if needed.

---

# 28. Final Kutu Mirror hardening verification

Verify:
- MirrorActivity exported=false
- no launcher categories
- explicit external launch denied where testable
- internal AirPlay still opens it
- absent from app lists
- no Recents snapshot
- iPhone works
- Mac works
- BACK works
- no stale frame
- port 7000 probe opens no UI
- boot auto-start works
- update auto-restart works

If any privacy test fails, do not switch HOME.

---

# 29. Attempt default HOME safely

Snapshot:
- current HOME
- package states
- stock launcher PSS
- Kutu Home PSS
- recommendations PSS
- MemAvailable

Try:

```bash
adb -s <DEVICE> shell cmd package set-home-activity local.kutu.home/.HomeActivity
```

Verify:

```bash
adb -s <DEVICE> shell cmd package resolve-activity --brief   -a android.intent.action.MAIN   -c android.intent.category.HOME
```

Also test physical HOME.

If Kutu Home becomes HOME normally, do not disable stock launcher.

If command says Success but stock HOME still wins:

1. identify exact stock HOME package/activity on this device
2. confirm it can be re-enabled
3. add rollback:
   ```bash
   pm enable --user 0 <STOCK_HOME_PACKAGE>
   cmd package set-home-activity <STOCK_HOME_ACTIVITY>
   ```
4. verify Kutu Home installed/enabled
5. show human exact disable and rollback
6. ask explicit approval

At this checkpoint, approve **Disable stock launcher** only if all Kutu Home tests passed and rollback is shown.

If approved:

```bash
adb -s <DEVICE> shell pm disable-user --user 0 <VERIFIED_STOCK_HOME_PACKAGE>
adb -s <DEVICE> shell cmd package set-home-activity local.kutu.home/.HomeActivity
```

Never uninstall stock launcher.

Verify immediately. If failure, restore stock HOME immediately.

---

# 30. Reboot after HOME switch

Reboot without manually launching Kutu Home.

Verify:
- normal boot
- Kutu Home auto-HOME
- no chooser
- no persistent black screen
- no crash loop
- remote works
- dock persists
- Kutu Mirror starts
- AirPlay returns

Temporary `FallbackHome` for a few seconds during boot may be normal.

---

# 31. Measure real HOME savings

After settling:
- Kutu Home PSS
- stock launcher state/PSS
- recommendations PSS
- MemAvailable
- Kutu Home CPU ~60 s
- services
- wakelocks

No synthetic benchmarks.

Reference only from original Mi Box:
- stock HOME ~59 MB
- settled Kutu Home ~10.5 MB
- Kutu Home idle CPU ~0.016% of one core

---

# 32. Mandatory normal-use checkpoint

Stop and ask human to use normally:
- HOME
- major streaming apps
- voice search
- Settings
- Screen Mirroring

Do not do final debloat until they confirm all normal.

---

# 33. Final low-risk cleanup

Inspect actual candidates.

Possible categories:
- stock recommendations
- manufacturer content/home feed
- manufacturer channel feed
- manufacturer phone-remote/discovery
- unused alternative launcher

Original Mi Box reference only:

```text
com.google.android.tvrecommendations
com.mitv.tvhome.atv
com.mitv.tvhome.michannel
com.mitv.milinkservice
com.teslacoilsw.launcher
```

Verify identity, dependency/client, residency and feature usage before disabling.

Use only `pm disable-user --user 0`.

Add undo to restore script.

Preserve:
- Assistant
- Chromecast
- DRM
- Bluetooth
- networking
- `com.android.providers.tv`
- CEC

Short smoke test:
- HOME
- voice search
- representative streaming app
- YouTube/video app
- one AirPlay connection
- Settings

---

# 34. Final security review

Focus on this actual box:
- listening TCP/UDP ports
- ADB/debugging
- Kutu exported components
- permissions
- port 7000
- ReceiverControlActivity
- screen/frame storage
- Recents
- release/debuggable flags
- world-readable files
- unnecessary services
- apps allowed to install unknown APKs
- firmware patch age

Classify:
- Critical
- High
- Medium
- Low
- Informational

Auto-fix only low-risk hardening that cannot reasonably break TV use.

Do not automatically revoke third-party updater/install permissions if that changes app function; report them.

Reference risks from original:
- network ADB left open: High until project end
- AirPlay PIN off: Medium on untrusted LAN
- sideloaded apps with install permission: Medium depending on app
- old firmware patch: inherent/residual
- ReceiverControlActivity exported: Low because it only starts an already-always-on receiver

---

# 35. Final performance summary

Use existing measurements, no long benchmark.

Compare baseline vs final:
- stock HOME vs Kutu Home PSS
- recommendations
- manufacturer content/home
- manufacturer remote/discovery
- analytics
- Kutu Mirror idle PSS
- Kutu Home idle CPU
- MemAvailable only as indicative if uptimes differ

Original Mi Box reference:
- launcher-related processes roughly 120 MB → 20–25 MB
- roughly 95 MB less resident memory while adding AirPlay

Do not promise identical results elsewhere.

---

# 36. Final report and rollback

Write final report:
- device/firmware
- baseline
- every package changed
- every undo
- final HOME
- Kutu Home build/hash
- Kutu Mirror build/hash
- iPhone/Mac tests
- real remote test
- reboot
- performance before/after
- security findings/fixes
- remaining risks
- signing-key locations
- exact rollback

Syntax-check `RESTORE-ALL`.

If removing Kutu apps:
1. restore stock HOME first
2. then uninstall Kutu user apps if requested

Tell human to back up signing keys.

---

# 37. Last action: turn ADB/debugging off

Only after:
- final report exists
- restore script is complete
- no more ADB work is needed
- final real iPhone AirPlay session passes

tell the human to turn off USB/network debugging on the TV/box, and optionally Developer Options.

Do not turn it off yourself if it would strand the session.

Once human confirms debugging is off, mark project complete.

---

# 38. Stop conditions

Stop and report rather than improvise if:
- ADB disappears repeatedly for unknown reason
- package purpose is unclear
- a system disable breaks boot/HOME
- AirPlay performance is unacceptable
- previous mirrored content can reappear
- port 7000 probe opens UI
- real remote long press launches app
- Kutu Home cannot reliably return HOME
- stock launcher rollback cannot be proven
- source/build integrity is questionable

The goal is a stable, reversible, lightweight TV appliance, not maximum debloat.
