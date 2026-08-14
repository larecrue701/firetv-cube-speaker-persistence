# Fire TV Cube 3rd Gen Speaker Persistence

Persistent internal speaker audio workaround for Fire TV Cube 3rd gen (gazelle) using Launcher Manager and a boot helper.

## What this does

This workaround keeps media audio routed to the Cube internal speaker after boot instead of falling back to HDMI/TV audio.

## Warning

This is an advanced workaround that relies on hidden/internal behavior and Launcher Manager as a command bridge.

Use at your own risk.

## How it works

At boot, a helper app:

- waits for Launcher Manager to become available
- runs a dual-mode/device-control call
- forces speaker routing with internal audio APIs
- retries several times in the background
- returns to Projectivy

## Notes

- no `/system` or `/vendor` modification
- no permanent root required
- tested on Fire TV Cube 3rd gen (`gazelle`)
