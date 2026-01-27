# Lutron Sunnata Keypad Blueprints

Blueprints for integrating Lutron Sunnata RadioRA3 keypads with Home Assistant.

##Goals##

- Enable controls (buttons, LEDs) to operate with non-Lutron devides in equivalent ways as Lutron devices are 
  configured in RadioRA3
- Support additional kinds of keypad interactions, specifically double clicking and long press/release

## Usage

The functionality is packaged as two separate automation blueprints, because the automations are called at different times with different concurrency needs.

- **Lutron Sunnata Keypad Click/Long-Click/Double-Click** - for handling different kinds of button events
- **Lutron Sunnata Keypad LED Control** - synchronizes the keypad LEDs to the states of external devices, replicating the *Scene Toggle* or *Zone Toggle* options in the Lutron designer software, but including non-Lutron devices

| Lutron Sunnata Keypad Click/Long-Click/Double-Click | Lutron Sunnata Keypad LED Control |
| -------- | ------- |
| [![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/bengber/home-assistant-lutron-sunnata-blueprints/blob/main/blueprints/automation/lutron-sunnata-keypad-click-long-click-double-click.yaml)  | [![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/bengber/home-assistant-lutron-sunnata-blueprints/blob/main/blueprints/automation/lutron-sunnata-keypad-led-control-blueprint.yaml)    |


**Important:** Button release events are only fired when the button is configured as ***Single Action*** in the Lutron designer software. This means that *Lutron Sunnata Keypad Click/Long-Click/Double-Click* will not work correctly for buttons configured as ***Scene Toggle*** or ***Zone Toggle*** in the software. Both of these behaviors can be easily emulated by binding the scripts to a [Stateful Scene](https://github.com/hugobloem/stateful_scenes) switch (for *Scene Toggle*) or to a [Group](https://www.home-assistant.io/integrations/group/) entity (for *Zone Toggle*).

1. Install both the **Lutron Sunnata Keypad Click/Long-Click/Double-Click** and **Lutron Sunnata Keypad LED Control** blueprints above.
2. For the buttons you want to automate, go the Lutron Designer software and set the behavior to *Single Action*.
3. Create a **Lutron Sunnata Keypad Click/Long-Click/Double-Click** automation for that keypad, defining appropriate actions for the button or buttons.
4. Create a **Lutron Sunnata Keypad LED Control** automation for the keypad, defining the entity to synchronize to LED state for each button you want to control.

## Supported Devices

- RRST-W2B-XX (Sunnata 2-Button Keypad)
- RRST-HN2B-XX (Sunnata Hybrid 2-Button Keypad)
- RRST-W3RL-XX (Sunnata 3-Button Keypad)
- RRST-HN3RL-XX (Sunnata Hybrid 3-Button Keypad)
- RRST-W4B-XX (Sunnata 4-Button Keypad)
- RRST-HN4B-XX (Sunnata Hybrid 4-Button Keypad)

## Requirements

- Home Assistant 2024.6 or newer
- Lutron Caseta integration configured
- At least one Sunnata keypad connected to your Lutron controller

## Installation

### Option 1: Direct Import (Recommended)

Click the badge above or use this link in Home Assistant:

```
Settings → Automations & Scenes → Blueprints → Import Blueprint
```

Then paste the raw GitHub URL of the blueprint file.

### Option 2: Manual Installation

1. Download the `lutron-sunnata-keypad-click-long-click-double-click.yaml` and `lutron-sunnata-keypad-led-control-blueprint.yaml` files
2. Copy it to your Home Assistant `config/blueprints/automation/` directory
3. Restart Home Assistant or reload automations


## Configuration Options

### Button Groups

Each button (1-4, Lower, Raise) has its own collapsible group with four action types:

- **Single Click** - Fires immediately for a quick press (or after double-click timeout if double-click is configured)
- **Double Click** - Fires when the button is pressed twice quickly (leave empty to disable detection)
- **Long Press** - Fires when the button is held down (leave empty to disable detection)
- **Release After Long Press** - Fires when a long-pressed button is released

### Advanced Settings

- **Long Press Timeout** (default: 500ms) - How long to hold before triggering a long press
- **Double Click Timeout** (default: 250ms) - Maximum time between clicks for double-click detection

### Detection Logic

The blueprint intelligently handles button press detection:

1. **If both long press AND double click are disabled** → Single click fires immediately (fastest response)
2. **If long press OR double click is enabled** → Blueprint waits to determine the press type
3. **Long press takes priority** → If held beyond the timeout, long press fires
4. **Double click detection** → Waits for a second press within the timeout window
5. **Single click as fallback** → If no double-click detected, single click fires


## Example Use Cases

### Example 1: Light with Different Brightness Settings
Use a single button for on/dim/off behavior in a light. Also remmember to associate the light with the Button 1 LED using the *Lutron Sunnata Keypad LED Control* blueprint.

**Button 1 - Single Click:** Turn on at 50% brightness
```yaml
- action: light.turn_on
  target:
    entity_id: light.bedroom
  data:
    brightness_pct: 50
```

**Button 1 - Double Click:** Turn on at 100% brightness
```yaml
- action: light.turn_on
  target:
    entity_id: light.bedroom
  data:
    brightness_pct: 100
```

**Button 1 - Long Press:** Turn off
```yaml
- action: light.turn_off
  target:
    entity_id: light.bedroom
```

### Example 2: Alarm Control with LED Warning
Arms an alarm when the button is pressed, lighting the LEDS when the alarm is armed. Also remmember to associate the alarm_control_panel with the Button 2 LED using the *Lutron Sunnata Keypad LED Control* blueprint.

**Button 2 - Single Click:** Arm home with 30-second warning
```yaml
- action: script.arm_alarm_after_thirty_seconds
  data:
    arm_mode: arm_home
```

**Button 2 - Double Click:** Arm away with 30-second warning
```yaml
- action: script.arm_alarm_after_thirty_seconds
  data:
    arm_mode: arm_away
```

*(Note: Requires a separate `arm_alarm_after_thirty_seconds` script)*

### Example 3: Scene Control
Uses a Stateful Scene switch to keep LED in sync. Also remmember to associate the switch with the Button 3 LED using the *Lutron Sunnata Keypad LED Control* blueprint.

**Button 3 - Single Click:** Activate "Movie Time" scene
```yaml
- action: scene_switch.turn_on
  target:
    entity_id: scene_switch.movie_time
```

**Button 3 - Double Click:** Activate "Bedtime" scene
```yaml
- action: scene_switch.turn_on
  target:
    entity_id: scene_switch.bedtime
```

### Example 4: Multi-Action Sequence

**Lower Button - Long Press:** Turn off all lights in sequence
In this case, the Button 4 LED can emulate *Zone Toggle* behavior (LED lit if *any* light is on) by creating a **Group** of lights and associating that Group to Button 4 using the *Lutron Sunnata Keypad LED Control* blueprint.

```yaml
- action: light.turn_off
  target:
    area_id: living_room
- delay:
    seconds: 1
- action: light.turn_off
  target:
    area_id: kitchen
```


## Troubleshooting

### Single Click Feels Slow

If only single clicks are configured, the response should be immediate. If it feels slow:
1. Check if you accidentally added a double-click or long-press action
2. Remove any empty/placeholder actions from double-click and long-press fields


## Contributing

Found a bug or have a feature request? Please open an issue on GitHub!

## License

This blueprint is released under the Apache License 2.0.
