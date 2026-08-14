# How It Works

## Goal

The goal is to keep media audio routed to the Fire TV Cube 3rd gen internal speaker after boot.

By default, audio may fall back to HDMI / TV output after reboot or after parts of the media stack restart.

## Core mechanism

The actual audio routing change is done through internal Android / Amazon audio APIs.

The key call is conceptually equivalent to:

```java
AudioSystem.setForceUse(FOR_MEDIA, FORCE_SPEAKER);
