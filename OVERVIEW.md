# Overview

## What this is

`whole_home_candle_mode.yaml` is a Home Assistant package that creates three virtual candle-mode switches:

- `switch.candle_mode_single`
- `switch.candle_mode_all` (friendly name: **Whole Home Candle Mode**)
- `switch.candle_mode_cozy`

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
- cozy clamp: **10% to 30%**
- rare bright flare: **85%** in normal mode, reduced in cozy mode
- rare dim falter: **55%** in normal mode, reduced in cozy mode
- flare chance: **1 in 10** per update
- falter chance: **1 in 20** per update

### Color temperature
- normal clamp: **2202K to 2502K**
- cozy clamp: **2202K to 2302K**

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
- cozy mode is global and changes the active candle envelope live
- turning off the controlled lamp turns single mode off
- turning a mode off turns its lamps off
- each runner waits briefly for its lamp to report `on` before entering the loop
- each runner tolerates transient command errors so one timeout does not kill the effect

## Best use case

This package works best with:
- warm white or tunable white bulbs
- lamps and accent lights
- not harsh overhead task lighting
