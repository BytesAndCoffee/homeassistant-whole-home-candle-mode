# Whole Home Candle Mode

Small Home Assistant package for candle-like lighting.

Files:
- `whole_home_candle_mode.yaml` - the package
- `OVERVIEW.md` - what it is and how it behaves
- `SETUP.md` - install instructions
- `CUSTOMIZING.md` - tuning guide
- `ARCHITECTURE.md` - GitHub-friendly architecture notes
- `ARCHITECTURE.typ` - Typst source version of the architecture notes

This package gives you:
- a single-lamp candle mode switch
- a whole-home candle mode switch
- bounded-random-walk flicker
- independent per-lamp movement in whole-home mode
- rare brightness flares for more believable flame behavior

## Inspired by

This package was inspired by the Home Assistant Community post:

- [A simple single automation candle effect for Hue White Ambience (and other) bulbs](https://community.home-assistant.io/t/a-simple-single-automation-candle-effect-for-hue-white-ambience-and-other-bulbs/235023)

That post is a great simple starting point.

## What this changes

Compared with the original single-automation approach, this package adds:
- virtual mode switches
- single-lamp and whole-home modes
- bounded random walk instead of absolute random jumps
- independent per-lamp motion in whole-home mode
- a rare 85% brightness flare for more flame-like behavior
- one runner per lamp, started in parallel, to reduce whole-room lag

So the original idea stays recognizable, but the implementation is tuned more like a reusable ambient-lighting system.
