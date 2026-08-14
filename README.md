# Fire TV Cube 3rd Gen Speaker Persistence

Persistent internal speaker audio workaround for Fire TV Cube 3rd gen (`gazelle`) using Launcher Manager and a boot helper.

## Overview

This repository documents an experimental workaround to keep media audio routed to the Fire TV Cube 3rd gen internal speaker after boot instead of falling back to HDMI / TV audio.

This setup was built and tested on a Fire TV Cube 3rd gen (`gazelle`) in a modified environment where stock Amazon behavior was already partially changed.

## What this does

This workaround aims to:

- force media output to the Cube internal speaker
- survive reboots
- avoid modifying `/system` or `/vendor`
- return to Projectivy after boot handling

## What this does not do

This is **not**:

- a permanent root method
- an official Amazon feature
- a clean system-level patch
- guaranteed to work on every firmware or setup

## Main idea

A helper app runs at boot and:

1. waits for Launcher Manager to become available
2. uses Launcher Manager as a privileged command bridge
3. triggers dual-mode / device-control behavior
4. forces speaker routing through internal audio APIs
5. retries several times in the background
6. returns control to Projectivy

## Setup

See [SETUP.md](SETUP.md) for the step-by-step installation flow.

## How it works

See [HOW_IT_WORKS.md](HOW_IT_WORKS.md) for the technical explanation.

## Troubleshooting

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for common failure cases and recovery notes.

## Downloads / Releases

If you want this repository to become truly reproducible for other users, publish the actual runtime artifacts in GitHub Releases:

- helper APK
- helper runtime payload
- exact version notes
- tested firmware notes

Suggested release contents:

- `helper-app.apk`
- `firetv-helper.apk`
- short release notes

## Tested on

- Fire TV Cube 3rd gen (`gazelle`)
- Projectivy Launcher
- Launcher Manager
- validated on August 14, 2026

## Known limitations

- Launcher Manager may still appear briefly during boot depending on timing
- behavior may vary by firmware
- launcher state may need to be restored if Home handling changes
- this is still an advanced workaround, not an official feature

## Repository contents

- `README.md` - overview
- `SETUP.md` - installation flow
- `HOW_IT_WORKS.md` - technical explanation
- `TROUBLESHOOTING.md` - common issues and recovery notes
- `DISCLAIMER.md` - usage and risk disclaimer
- `CHANGELOG.md` - repository history

## Warning

This is an experimental workaround.

Use it only if you understand the risks and have a recovery path available.

See [DISCLAIMER.md](DISCLAIMER.md).
