# Disclaimer

This repository documents an experimental workaround for persistent internal speaker audio on the Fire TV Cube 3rd gen (`gazelle`).

## No warranty

Everything here is provided **as is**, with no warranty of any kind.

I make no guarantee that this method will:
- work on your device
- remain compatible with future firmware
- be safe in all configurations
- avoid regressions or side effects

## Risk

This setup relies on undocumented or hidden platform behavior, helper apps, and launcher-based command execution.

Possible risks include:
- boot issues
- launcher issues
- audio routing issues
- UI instability
- loss of settings after reboot
- incompatibility with firmware or app updates

## No responsibility

If you use anything from this repository, you do so **at your own risk**.

I am not responsible for:
- data loss
- device instability
- broken app behavior
- soft-brick scenarios
- factory reset requirements
- any other damage or inconvenience

## No official support

This is **not** an official Amazon solution and is not endorsed by Amazon, Projectivy, Launcher Manager, or any other vendor.

## System modification warning

Even if this method does not require direct modification of `/system` or `/vendor`, it still interacts with internal Android / Fire OS behavior in ways that may not be supported.

Be careful, test incrementally, and keep a recovery path available.

## Recommendation

Before trying anything:
- understand each step
- keep ADB access available
- avoid changing multiple things at once
- be prepared to recover the device if needed
