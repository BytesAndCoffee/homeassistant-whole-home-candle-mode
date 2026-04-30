# Customizing

## Main knobs

You can tune six big things:

1. brightness range
2. color temperature range
3. step size
4. loop cadence
5. flare/falter chance
6. cozy-mode envelope

## Brightness clamp

Current normal clamp is roughly:
- **60% to 80%**
- with a rare **85% flare**
- and a rarer dim falter

In the package this is represented by brightness values:
- `153` minimum
- `204` maximum
- `217` flare
- `140` falter

Why those numbers:
- Home Assistant brightness uses `0-255`
- `153 / 255 ≈ 60%`
- `204 / 255 ≈ 80%`
- `217 / 255 ≈ 85%`

## Color temperature clamp

Current range:
- **2202K to 2502K**

Lower Kelvin is warmer.

## Step size

Step size controls how much each update can move from the current state.

### Smaller steps
Feels:
- softer
- smoother
- cozier

### Larger steps
Feels:
- livelier
- more active
- more flame-like

Look for values like:
- `range(-18, 19)`
- `range(-16, 17)`

Those are the per-step deltas.

## Cadence

Current loop cadence is:
- **100ms to 300ms**

Good practical bands:
- `140ms to 320ms` = calmer
- `100ms to 300ms` = current default
- `80ms to 180ms` = aggressive

## Flare chance

Current flare chance:
- **1 in 10**

Current falter chance:
- **1 in 20**

These are controlled by:

```jinja2
(range(1, 11) | random) == 1
(range(1, 21) | random) == 1
```

Examples:
- `range(1, 21)` = 1 in 20, rarer flare
- `range(1, 6)` = 1 in 5, more frequent flare
- `range(1, 40)` = 1 in 40, very rare falter

## Cozy mode

Cozy mode swaps the normal envelope for a much dimmer one:
- brightness: **10% to 30%**
- color temp: **2202K to 2302K**

Because cozy mode is a separate global switch, you can toggle it while single or whole-home candle mode is already running.

## Per-lamp independence

Keep this.

Whole-home mode works because:
- each lamp has its own runner
- each runner has its own bounded walk
- the master switch only multiplexes start/stop control

If every lamp shares one absolute value at once, the whole room looks synchronized and fake.

## Scaling up

If you add more lamps:
1. add the light to the master switch target lists
2. duplicate one `candle_walk_lamp_X` script block
3. point it at the new entity
4. add that script to the `parallel:` starter automation

That keeps performance and orchestration predictable.
