# Whole Home Candle Mode Architecture

This document describes how the Home Assistant candle mode package works.

## High-level flow

```text
Virtual switch
Whole Home Candle Mode
        |
        v
input_boolean
candle_mode_whole_room = on
        |
        v
Starter automation
fans out in parallel
        |
        v
One script runner per lamp
independent bounded random walk
        |
        v
Lamp updates
brightness, kelvin, transition, delay
```

Sequence:

- User turns on **Whole Home Candle Mode**
- Home Assistant turns on `input_boolean.candle_mode_whole_room`
- A starter automation launches one script per lamp in parallel
- Each script waits for its lamp to report `on`
- Each script runs its own loop until the mode or lamp turns off

## Per-lamp algorithm

Each lamp runner does this:

1. Wait for the lamp to be on
2. Read current brightness and color temperature
3. Decide whether to flare this cycle
4. If not flaring, apply a bounded random walk step
5. Clamp values into the allowed envelope
6. Write the new light state with a short transition
7. Sleep for 100 ms to 300 ms
8. Repeat

## Current tuning

- Brightness clamp: 60% to 80%
- Rare flare: 85%
- Flare chance: 1 in 10
- Color temperature clamp: 2202 K to 2502 K
- Per-bulb cadence: 100 ms to 300 ms

## Why this feels better than pure randomization

Naive randomization jumps to unrelated absolute values, which tends to look jittery.

A bounded random walk:

- preserves continuity
- creates local drift
- lets each lamp feel alive
- avoids fake synchronized motion

## Python-esque pseudocode

```python
import random
import time

MIN_BRIGHTNESS = 153   # 60%
MAX_BRIGHTNESS = 204   # 80%
FLARE_BRIGHTNESS = 217 # 85%

MIN_KELVIN = 2202
MAX_KELVIN = 2502


def clamp(value, low, high):
    return max(low, min(high, value))


def candle_step(current_brightness, current_kelvin):
    flare_now = random.randint(1, 10) == 1

    if flare_now:
        next_brightness = FLARE_BRIGHTNESS
    else:
        next_brightness = clamp(
            current_brightness + random.randint(-16, 16),
            MIN_BRIGHTNESS,
            MAX_BRIGHTNESS,
        )

    next_kelvin = clamp(
        current_kelvin + random.randint(-55, 55),
        MIN_KELVIN,
        MAX_KELVIN,
    )

    transition = random.choice([0.12, 0.16, 0.22, 0.28])
    delay_ms = random.randint(100, 300)

    return {
        "brightness": next_brightness,
        "kelvin": next_kelvin,
        "transition": transition,
        "delay_ms": delay_ms,
    }


def run_lamp(mode_is_on, lamp_is_on, read_state, write_state):
    while mode_is_on() and lamp_is_on():
        current = read_state()

        step = candle_step(
            current_brightness=current["brightness"],
            current_kelvin=current["kelvin"],
        )

        write_state(
            brightness=step["brightness"],
            color_temp_kelvin=step["kelvin"],
            transition=step["transition"],
        )

        time.sleep(step["delay_ms"] / 1000)
```

## Whole-home orchestration

Whole-home mode does **not** use one giant serialized loop.

Instead, it does this:

```python
def start_whole_home(lamps):
    for lamp in lamps:
        start_in_parallel(run_lamp_for, lamp)
```

That design is important because it:

- reduces startup serialization
- prevents one long-running script from blocking others
- keeps each lamp independently animated

## Mental model

Think of it as:

- one master mode switch
- one supervisor automation
- many tiny lamp workers
- each worker simulating one small flame source
