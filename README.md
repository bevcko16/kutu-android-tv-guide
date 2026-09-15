# Kutu Android TV Guide

A Claude Code + ADB workflow for optimizing older Android TV devices, adding AirPlay Screen Mirroring and building a lightweight custom launcher.

Originally developed and tested on:

- Xiaomi Mi Box 3
- Android TV 9

## Full step-by-step guide

The full user guide is published separately on Substack:

**[Read the full guide](https://berksim.substack.com/p/mi-box-android-tv-guide-optimise)**

## Files

### MIBOX-CLAUDE-MASTER-GUIDE.md

Download this file and give it to Claude Code after your Android TV device is connected over ADB.

Claude uses it to:

- inspect your specific Android TV device
- create a read-only baseline
- build a reversible rollback system
- perform conservative debloating
- build and test Kutu Mirror
- build and test Kutu Home
- safely replace the stock launcher where appropriate
- perform final security and performance checks

### Kutu Home background

Download the included background image and give it to Claude Code together with the master guide.

You can also use your own 16:9 image instead.

## Important

Do not manually copy Xiaomi package names to another Android TV device.

The master workflow first inspects the actual connected device and adapts the process accordingly.

System-package changes are designed to remain reversible.

## Disclaimer

This is an enthusiast project for technically comfortable users.

Android TV firmware differs by manufacturer and model. Keep rollback access available and do not disable packages whose purpose is unclear.
