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

## Launcher Manager opens visibly at boot

### Symptom

Launcher Manager settings screen appears during boot before returning to the launcher.

### Cause

Launcher Manager is used as a command bridge and may briefly appear while the helper waits for its localhost shell server.

### Mitigation

Reducing the wait time before sending the command helped reduce the visible delay.

A retry loop running in the background also helped return to Projectivy faster.

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

## STBEmu steals focus after reboot

### Symptom

After reboot, STBEmu resumes or takes focus unexpectedly.

### Cause

STBEmu may restore its previous task aggressively depending on state.

### Workaround

Force-stop STBEmu before reboot if needed, or avoid leaving it as the last unstable foreground task before restarting.

## Keyboard gets stuck

### Symptom

The on-screen keyboard appears and becomes difficult to dismiss.

### Notes

This can depend on app behavior and not just system behavior.

In some setups, Amazon Fire TV IME also produced crashes during early boot before user-unlock state was complete.

### Recommendation

Avoid relying on a remapped search path unless you have a safe escape route.

## IME crash in logs

### Symptom

Crash logs show failures from `com.amazon.tv.ime`.

### Example pattern

An observed crash pattern was:

- `SharedPreferences in credential encrypted storage are not available until after user is unlocked`

### Interpretation

This looked like a real early-boot IME problem, but in the tested setup it did not appear to be the main cause of the persistent black-screen Home issue.

## Recovery advice

If experimenting with this setup:

- keep ADB available
- change one thing at a time
- verify after each reboot
- keep a known-good launcher path
- avoid stacking multiple unrelated experiments before testing
