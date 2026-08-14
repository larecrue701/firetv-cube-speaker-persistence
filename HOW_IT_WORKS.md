HOW_IT_WORKS.md
# How It Works

## Goal

The goal is to keep media audio routed to the Fire TV Cube 3rd gen internal speaker after boot.

By default, audio may fall back to HDMI / TV output after reboot or after parts of the media stack restart.

## Core mechanism

The actual audio routing change is done through internal Android / Amazon audio APIs.

The key call is conceptually equivalent to:

```java

AudioSystem.setForceUse(FOR_MEDIA, FORCE_SPEAKER);
Amazon-specific constants are used for the Fire TV audio stack.
Helper app structure
The helper app contains:
BootReceiver
MainActivity
ApplyService
BootJobService
BootReceiver
Listens for:
LOCKED_BOOT_COMPLETED
BOOT_COMPLETED
MY_PACKAGE_REPLACED
It schedules both:
an AlarmManager delayed execution
a persistent JobScheduler fallback
MainActivity
Used as a one-shot wake activity.
This is important because Android may keep a newly installed package in a never-launched state until it is started once manually.
ApplyService
Runs the actual boot-time apply logic.
BootJobService
Acts as a persistent fallback in case the initial boot-time apply path is not enough.
Why a simple boot receiver was not enough
A plain boot receiver was not reliable enough because:
the helper package was initially treated as never-launched
BOOT_COMPLETED delivery alone was not always enough
Launcher Manager's localhost shell bridge was not always ready immediately at boot
Reliability improved after combining:
direct-boot awareness
delayed execution
persistent job scheduling
a first manual launch of the helper app
Why Launcher Manager was used
Launcher Manager provides a local privileged shell bridge that can be used to execute commands from localhost once its process is up.
That bridge was used to run the helper logic without modifying system partitions.
Working boot command pattern
A working pattern was:
export CLASSPATH='/cache/firetv-helper.apk'; ( i=0; while [ $i -lt 6 ]; do app_process /system/bin com.aziz.DualModeClient tv >/dev/null 2>&1; app_process /system/bin com.aziz.ForceAudio speaker; i=$((i+1)); if [ $i -lt 6 ]; then sleep 15; fi; done ) >/dev/null 2>&1 & am start -n com.spocky.projengmenu/.ui.home.MainActivity
What this command does
sets the helper APK in CLASSPATH
triggers dual-mode / device-control behavior
forces audio to the Cube speaker
retries several times in case the audio stack resets during boot
returns to Projectivy quickly
Why retries mattered
A single apply call was sometimes not enough.
During boot, other services could reassert default routing or restart parts of the media pipeline.
Running several delayed retries made the speaker route stick much more reliably.
Important state transition
A major reliability improvement came after launching the helper app once manually.
Before that, the package could remain in a state equivalent to:
stopped=true
notLaunched=true
After the first launch, boot persistence became much more reliable.
Summary
The persistence strategy was:
install helper app
launch helper app once manually
listen for boot-related broadcasts
schedule delayed and persistent execution
use Launcher Manager as command bridge
force speaker routing repeatedly in the background
return to Projectivy

---

## `TROUBLESHOOTING.md`

```md
# Troubleshooting

## Black screen on Home

### Symptom

Pressing `Home` or rebooting leads to a black screen.

### Possible cause

The Home handler may have fallen back to Amazon launcher instead of the custom launcher path.

### Notes

In one tested setup, the actual black screen was caused by the Amazon launcher path, while Projectivy itself still rendered correctly.

Launcher Manager state also mattered:

- `Custom Launcher Support` could be reset to `Disabled`
- the selected custom launcher could revert away from Projectivy

### What to check

- is `Custom Launcher Support` enabled in Launcher Manager?
- is `Projectivy Launcher` selected as the target launcher?
- does Projectivy render correctly if launched directly?

---

## Launcher Manager opens visibly at boot

### Symptom

Launcher Manager settings screen appears during boot before returning to the launcher.

### Cause

Launcher Manager is used as a command bridge and may briefly appear while the helper waits for its localhost shell server.

### Mitigation

Reducing the wait time before sending the command helped reduce the visible delay.

A retry loop running in the background also helped return to Projectivy faster.

---

## Audio falls back to HDMI after reboot

### Symptom

Speaker routing works manually but does not survive reboot.

### Possible causes

- helper app never launched manually after install
- package still treated as never-launched
- Launcher Manager shell bridge not ready in time
- only one apply attempt was made

### Things that helped

- launching the helper app once manually
- using both `AlarmManager` and `JobScheduler`
- multiple delayed retries of the speaker-force command

---

## Projectivy no longer resumes after boot

### Symptom

The system no longer lands on Projectivy after boot.

### Possible causes

- Launcher Manager custom launcher support got disabled
- selected custom launcher changed
- Home resolution fell back to a stock launcher path

### Check

Open Launcher Manager and verify:

- `Custom Launcher Support : Enabled`
- target launcher = `Projectivy Launcher`

---

## STBEmu steals focus after reboot

### Symptom

After reboot, STBEmu resumes or takes focus unexpectedly.

### Cause

STBEmu may restore its previous task aggressively depending on state.

### Workaround

Force-stop STBEmu before reboot if needed, or avoid leaving it as the last unstable foreground task before restarting.

---

## Keyboard gets stuck

### Symptom

The on-screen keyboard appears and becomes difficult to dismiss.

### Notes

This can depend on app behavior and not just system behavior.

In some setups, Amazon Fire TV IME also produced crashes during early boot before user-unlock state was complete.

### Recommendation

Avoid relying on a remapped search path unless you have a safe escape route.

---

## IME crash in logs

### Symptom

Crash logs show failures from `com.amazon.tv.ime`.

### Example pattern

An observed crash pattern was:

- `SharedPreferences in credential encrypted storage are not available until after user is unlocked`

### Interpretation

This looked like a real early-boot IME problem, but in the tested setup it did not appear to be the main cause of the persistent black-screen Home issue.

---

## Recovery advice

If experimenting with this setup:

- keep ADB available
- change one thing at a time
- verify after each reboot
- keep a known-good launcher path
- avoid stacking multiple unrelated experiments before testing
