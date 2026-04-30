# Overview

## What this is

`whole_home_candle_mode.yaml` is a Home Assistant package that creates two virtual candle-mode switches:

- `switch.candle_mode_single`
- `switch.candle_mode_all` (friendly name: **Whole Home Candle Mode**)

The goal is to make lamps behave more like flame, not like a random-number generator.

## Core idea

Instead of jumping to a brand new random brightness and color temperature on every loop, each lamp uses a **bounded random walk**:

- read the current state
- nudge brightness by a small `+/-` amount
- nudge color temperature by a small `+/-` amount
- clamp those values inside a believable candle envelope

That gives you motion that feels continuous.

## Why it looks better

Naive randomization tends to look like jitter.
Bounded random walks look more like:
- flame drift
- slight surges and dips
- local variation
- continuity over time

## Current behavior

### Brightness
- normal clamp: **60% to 80%**
- rare flare: **85%**
- flare chance: **1 in 10** per update

### Color temperature
- clamp: **2202K to 2502K**

### Cadence
- per-bulb update loop: **100ms to 300ms**

## Modes

### Single mode
Controls one chosen lamp.

### Whole-home mode
Controls a list of lamps, with each lamp moving independently.

This package now uses **one script runner per lamp** in whole-home mode, with a master switch that fans out and starts them in parallel.
That matters because synchronized or serialized lamps look fake, laggy, or both.

## Safety / behavior rules

- single mode and whole-home mode are mutually exclusive
- turning off the controlled lamp turns single mode off
- turning a mode off turns its lamps off
- each runner waits briefly for its lamp to report `on` before entering the loop

## Best use case

This package works best with:
- warm white or tunable white bulbs
- lamps and accent lights
- not harsh overhead task lighting
