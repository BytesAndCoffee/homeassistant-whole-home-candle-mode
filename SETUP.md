# Setup

## 1. Enable packages in Home Assistant

In `configuration.yaml`:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

If you already use packages, leave this alone.

## 2. Copy the package file

Copy:

- `whole_home_candle_mode.yaml`

into your Home Assistant packages directory, for example:

- `config/packages/whole_home_candle_mode.yaml`

## 3. Replace the placeholder entity IDs

The package ships with placeholders such as:

- `light.REPLACE_SINGLE_TARGET`
- `light.REPLACE_LAMP_2`
- `light.REPLACE_LAMP_3`

You need to replace these with real light entities from your installation.

### Example

```yaml
light.REPLACE_SINGLE_TARGET
```

might become:

```yaml
light.side_table_lamp
```

And the whole-home list might become:

```yaml
- light.side_table_lamp
- light.desk_lamp
- light.floor_lamp_left
- light.floor_lamp_right
```

## 4. Add more lamps if needed

Whole-home mode uses:
- one starter automation
- one script per lamp

So if you want more than the sample lamp count, duplicate one of the `candle_walk_lamp_X` script blocks and add it to the `parallel:` starter list.

## 5. Restart Home Assistant

Restart HA after saving the package.

## 6. Find the new switches

After restart, look for:

- `switch.candle_mode_single`
- `switch.candle_mode_all`

Friendly names should appear as:
- **Candle Mode Single**
- **Whole Home Candle Mode**

## 7. Test single mode first

Recommended order:
1. turn on the single-mode switch
2. confirm the target lamp turns on
3. confirm it flickers smoothly
4. then test whole-home mode

## Troubleshooting

### Nothing appears after restart
- confirm packages are enabled
- confirm the YAML file is in the packages directory
- check HA logs for YAML errors

### Switches appear but lamp does nothing
- confirm your light entity IDs are correct
- confirm the bulbs support brightness and color temperature

### Whole-home mode turns lamps on but they just sit there
- check that each per-lamp script points at the correct light entity
- make sure each script is listed in the `parallel:` starter automation

### Whole-home mode feels wrong
- remove overhead/task lights first
- start with lamps and accent lights
