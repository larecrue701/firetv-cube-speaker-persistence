# Setup Guide

## Scope

This guide describes a practical setup flow for reproducing the persistent internal speaker workaround on a Fire TV Cube 3rd gen (`gazelle`).

This is intended to make the process as close to "plug and play" as possible, but it still assumes:

- a Fire TV Cube 3rd gen (`gazelle`)
- ADB access
- Launcher Manager installed
- Projectivy Launcher installed
- the published unsigned helper APK:
  - `firetv-cube-speaker-helper-final-unsigned-1.0.apk`
- the ability to self-sign APKs before installation

## Before you start

Make sure you have:

- Fire TV Cube 3rd gen (`gazelle`)
- ADB debugging enabled
- Launcher Manager installed and working
- Projectivy Launcher installed
- `firetv-cube-speaker-helper-final-unsigned-1.0.apk`
- a local APK signing method (`apksigner` or equivalent)
- a recovery path if the launcher state breaks

## What you need to prepare

You need two things:

1. a helper APK that:
   - listens for boot events
   - schedules delayed execution
   - can trigger the audio routing helper logic

2. a helper runtime APK or dex payload available at:

```sh
/cache/firetv-helper.apk
```

This helper payload must expose the equivalent of:

- `org.ftvcube.helper.DualModeClient`
- `org.ftvcube.helper.ForceAudio`

The repository APK is intentionally unsigned. Users are expected to sign it themselves before installation.

## Install order

Follow this order exactly.

### 1. Install Projectivy Launcher

Install and verify that Projectivy launches normally.

### 2. Install Launcher Manager

Install Launcher Manager and verify:

- it opens correctly
- `Custom Launcher Support` can be enabled
- `Projectivy Launcher` can be selected as the custom launcher

### 3. Set Launcher Manager / Projectivy path

Inside Launcher Manager:

- enable `Custom Launcher Support`
- keep `Proxy Launcher Mode` enabled
- set custom launcher to `Projectivy Launcher`

Verify that pressing `Home` lands on Projectivy.

### 4. Install the boot helper APK

Sign and install `firetv-cube-speaker-helper-final-unsigned-1.0.apk`.

It contains:

- `BootReceiver`
- `MainActivity`
- `ApplyService`
- `BootJobService`

The package name in the fully anonymized build is:

```sh
org.ftvcube.speakerhelper
```

### 5. Push the helper runtime payload

Push the helper runtime payload so it is available at:

```sh
/cache/firetv-helper.apk
```

### 6. Launch the helper app once manually

This step is critical.

The helper app must be launched once manually after install so Android stops treating it as never-launched.

Without this step, boot persistence may be unreliable.

## Expected logic at boot

At boot, the helper should:

1. receive boot-related broadcast(s)
2. schedule delayed execution with `AlarmManager`
3. schedule a persistent fallback with `JobScheduler`
4. start Launcher Manager
5. wait briefly for Launcher Manager's localhost shell bridge
6. run the speaker-force command
7. retry several times in the background
8. return control to Projectivy

## Working command pattern

The command pattern used in the documented setup is:

```sh
export CLASSPATH='/cache/firetv-helper.apk'; ( i=0; while [ $i -lt 6 ]; do app_process /system/bin org.ftvcube.helper.DualModeClient tv >/dev/null 2>&1; app_process /system/bin org.ftvcube.helper.ForceAudio speaker; i=$((i+1)); if [ $i -lt 6 ]; then sleep 15; fi; done ) >/dev/null 2>&1 & am start -n com.spocky.projengmenu/.ui.home.MainActivity
```

## First validation

After installation and first manual launch:

1. return to Home
2. verify that Projectivy still works
3. trigger the helper manually if your build supports it
4. confirm that audio is routed to the Cube speaker

## Reboot validation

Then reboot and verify:

1. the device does not fall back to a black-screen launcher path
2. Home eventually lands on Projectivy
3. media audio is still routed to the Cube speaker

## If it does not work

Check these first:

- was the unsigned APK signed before installation?
- was the helper app launched once manually after installation?
- is Launcher Manager still set to `Custom Launcher Support : Enabled`?
- is `Projectivy Launcher` still selected?
- is `/cache/firetv-helper.apk` present?
- is Launcher Manager available and functioning?

Then consult [TROUBLESHOOTING.md](TROUBLESHOOTING.md).

## What would make this truly plug and play

To make this repository fully reproducible for most users, publish:

1. a signed release APK
2. the helper runtime payload (`firetv-helper.apk`)
3. exact install commands
4. firmware notes
5. screenshots for each important UI step

Without those artifacts, this repository is still more of an advanced guide than a one-click solution.
