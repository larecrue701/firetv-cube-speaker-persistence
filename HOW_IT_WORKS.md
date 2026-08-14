# How It Works

## Goal

The goal is to keep media audio routed to the Fire TV Cube 3rd gen internal speaker after boot.

By default, audio may fall back to HDMI / TV output after reboot or after parts of the media stack restart.

## Core mechanism

The actual audio routing change is done through internal Android / Amazon audio APIs.

The key call is conceptually equivalent to:

```java
AudioSystem.setForceUse(FOR_MEDIA, FORCE_SPEAKER);
```

Amazon-specific constants are used for the Fire TV audio stack.

## Helper app structure

The helper app contains:

- `BootReceiver`
- `MainActivity`
- `ApplyService`
- `BootJobService`

### `BootReceiver`

Listens for:

- `LOCKED_BOOT_COMPLETED`
- `BOOT_COMPLETED`
- `MY_PACKAGE_REPLACED`

It schedules both:

- an `AlarmManager` delayed execution
- a persistent `JobScheduler` fallback

### `MainActivity`

Used as a one-shot wake activity.

This is important because Android may keep a newly installed package in a never-launched state until it is started once manually.

### `ApplyService`

Runs the actual boot-time apply logic.

### `BootJobService`

Acts as a persistent fallback in case the initial boot-time apply path is not enough.

## Why a simple boot receiver was not enough

A plain boot receiver was not reliable enough because:

1. the helper package was initially treated as never-launched
2. `BOOT_COMPLETED` delivery alone was not always enough
3. Launcher Manager's localhost shell bridge was not always ready immediately at boot

Reliability improved after combining:

- direct-boot awareness
- delayed execution
- persistent job scheduling
- a first manual launch of the helper app

## Why Launcher Manager was used

Launcher Manager provides a local privileged shell bridge that can be used to execute commands from localhost once its process is up.

That bridge was used to run the helper logic without modifying system partitions.

## Working boot command pattern

A working pattern was:

```sh
export CLASSPATH='/cache/firetv-helper.apk'; ( i=0; while [ $i -lt 6 ]; do app_process /system/bin com.aziz.DualModeClient tv >/dev/null 2>&1; app_process /system/bin com.aziz.ForceAudio speaker; i=$((i+1)); if [ $i -lt 6 ]; then sleep 15; fi; done ) >/dev/null 2>&1 & am start -n com.spocky.projengmenu/.ui.home.MainActivity
```

## What this command does

- sets the helper APK in `CLASSPATH`
- triggers dual-mode / device-control behavior
- forces audio to the Cube speaker
- retries several times in case the audio stack resets during boot
- returns to Projectivy quickly

## Why retries mattered

A single apply call was sometimes not enough.

During boot, other services could reassert default routing or restart parts of the media pipeline.

Running several delayed retries made the speaker route stick much more reliably.

## Important state transition

A major reliability improvement came after launching the helper app once manually.

Before that, the package could remain in a state equivalent to:

- `stopped=true`
- `notLaunched=true`

After the first launch, boot persistence became much more reliable.

## Summary

The persistence strategy was:

1. install helper app
2. launch helper app once manually
3. listen for boot-related broadcasts
4. schedule delayed and persistent execution
5. use Launcher Manager as command bridge
6. force speaker routing repeatedly in the background
7. return to Projectivy
