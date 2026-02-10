# Lutron Sunnata Keypad Combination Lock

Home Assistant Blueprint for using a Lutron Sunnata RadioRA3 keypad as a "combination lock", where an event is triggered after a specific sequence of button presses (single click, double click, or long press) is detected.

## Goals

- Enable secure or hidden functionality (unlocking doors, disarming alarms, etc.) by requiring
  a specific button sequence
- Support complex passcode patterns mixing single clicks, double clicks, and long presses

## Usage

The **Lutron Sunnata Keypad Combination Lock** blueprint monitors button events and tracks a sequence of actions. When the sequence matches your configured passcode, it triggers your chosen action.

| Lutron Sunnata Keypad Combination Lock |
| -------- |
| [![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/bengber/home-assistant-lutron-sunnata-blueprints/blob/main/blueprints/automation/lutron-sunnata-keypad-combination-lock.yaml) |

### Important

Button release events are only fired when the button is configured as ***Single Action*** in the Lutron designer software. This means the combination lock will not work correctly for buttons configured as ***Scene Toggle*** or ***Zone Toggle***.

1. Install the **Lutron Sunnata Keypad Combination Lock** blueprint using the badge above.
2. In the Lutron Designer software, set the buttons you want to use to *Single Action*.
3. Create two helper entities (see *Installation*):
   - An `input_text` helper to store the in-progress passcode
   - An `input_datetime` helper (with time enabled) to track the last button press
4. Create a **Lutron Sunnata Keypad Combination Lock** automation, defining your passcode sequence and the action to trigger when matched.

## Supported Devices

- RRST-W2B-XX (Sunnata 2-Button Keypad)
- RRST-HN2B-XX (Sunnata Hybrid 2-Button Keypad)
- RRST-W3RL-XX (Sunnata 3-Button Keypad with Raise/Lower)
- RRST-HN3RL-XX (Sunnata Hybrid 3-Button Keypad with Raise/Lower)
- RRST-W4B-XX (Sunnata 4-Button Keypad)
- RRST-HN4B-XX (Sunnata Hybrid 4-Button Keypad)

## Requirements

- Home Assistant 2024.6 or newer
- Lutron Caseta integration configured
- At least one Sunnata keypad connected to your Lutron controller
- Two helper entities (see Installation section)

## Installation

### Option 1: Direct Import (Recommended)

Click the badge above or use this link in Home Assistant:

```
Settings → Automations & Scenes → Blueprints → Import Blueprint
```

Then paste the raw GitHub URL:
```
https://github.com/bengber/home-assistant-lutron-sunnata-blueprints/blob/main/blueprints/automation/lutron-sunnata-keypad-combination-lock.yaml
```

### Option 2: Manual Installation

1. Download the `lutron-sunnata-keypad-combination-lock.yaml` file
2. Copy it to your Home Assistant `config/blueprints/automation/` directory
3. Restart Home Assistant or reload automations

### Creating Required Helper Entities

Before creating an automation, you need to create two helper entities:

1. **Input Text Helper** (for passcode progress):
   ```
   Settings → Devices & Services → Helpers → Create Helper → Text
   Name: "Keypad Passcode Progress"
   Max length: 255
   ```

2. **Input Datetime Helper** (for last press time):
   ```
   Settings → Devices & Services → Helpers → Create Helper → Date and/or time
   Name: "Keypad Last Press Time"
   ✓ Date
   ✓ Time
   ```

## Configuration Options

### Passcode Sequence

Define your passcode as a comma-separated list of button actions. Each action should be one of:

- `button_1_single_click`, `button_1_double_click`, `button_1_long_press`
- `button_2_single_click`, `button_2_double_click`, `button_2_long_press`
- `button_3_single_click`, `button_3_double_click`, `button_3_long_press`
- `button_4_single_click`, `button_4_double_click`, `button_4_long_press`
- `button_lower_single_click`, `button_lower_double_click`, `button_lower_long_press`
- `button_raise_single_click`, `button_raise_double_click`, `button_raise_long_press`

**Example:**
```
button_1_single_click,button_2_single_click,button_1_double_click,button_3_long_press
```

### Passcode Matched Action

Define what happens when the passcode sequence is successfully entered. This can be any Home Assistant action (lights, scripts, scenes, notifications, etc.).

### Helper Entities (required)

These are used to store the passcode state as it is being entered.

- **Passcode In-Progress**: Select the `input_text` helper to store the current sequence
- **Last Press Time**: Select the `input_datetime` helper to track timing between presses

### Advanced Settings

- **Long Press Timeout** (default: 500ms) - How long to hold before triggering a long press
- **Double Click Timeout** (default: 250ms) - Maximum time between clicks for double-click detection
- **Next-In-Sequence Timeout** (default: 3s) - Time allowed between sequence steps before the passcode resets. If the user does not press the next keypress in time, they will need to start the code again. The default will need to be raised if the passocde requires very long button presses.
- **Debug Mode** (default: off) - Enable Home Assistant notifications showing detailed passcode progress and timeouts.

## Example Use Cases

### Example 1: Simple 4-Button Code to Unlock Door

**Passcode Sequence:**
```
button_1_single_click,button_2_single_click,button_3_single_click,button_4_single_click
```

**Passcode Matched Action:**
```yaml
- action: lock.unlock
  target:
    entity_id: lock.front_door
- action: notify.mobile_app
  data:
    message: "Front door unlocked via keypad"
```

### Example 2: Complex Pattern with Mixed Actions

**Passcode Sequence:**
```
button_1_double_click,button_2_long_press,button_1_single_click,button_raise_single_click
```

**Passcode Matched Action:**
```yaml
- action: scene.turn_on
  target:
    entity_id: scene.guest_mode
- action: persistent_notification.create
  data:
    title: "Guest Mode Activated"
    message: "Guest mode activated via keypad combination"
```

### Example 3: Panic Button Sequence

**Passcode Sequence:**
```
button_lower_long_press,button_raise_long_press,button_lower_long_press
```

**Passcode Matched Action:**
```yaml
- action: script.turn_on
  target:
    entity_id: script.emergency_lights_and_siren
- action: notify.family
  data:
    message: "EMERGENCY: Panic button activated at home"
    data:
      priority: high
```

### Example 4: Hidden Admin Menu

**Passcode Sequence:**
```
button_1_single_click,button_1_single_click,button_2_single_click,button_2_single_click
```

**Passcode Matched Action:**
```yaml
- action: input_boolean.turn_on
  target:
    entity_id: input_boolean.admin_mode
- action: light.turn_on
  target:
    entity_id: light.status_indicator
  data:
    color_name: purple
    brightness_pct: 100
```

### Example 5: Multi-Stage Lighting Control

**Passcode Sequence:**
```
button_3_double_click,button_3_long_press
```

**Passcode Matched Action:**
```yaml
- action: script.turn_on
  target:
    entity_id: script.party_mode_lighting_sequence
```

## How It Works

1. Each button press is detected and classified (single click, double click, or long press)
2. The action is appended to the passcode progress (`input_text` helper)
3. If the progress doesn't match the start of your configured sequence, it resets
4. If no button is pressed within the timeout period, the progress resets
5. When the progress exactly matches the configured sequence, your action triggers and the progress resets

## Troubleshooting

### Passcode Not Triggering

1. Enable **Debug Mode** in Advanced Settings to see progress notifications
2. Verify your helper entities are configured correctly
3. Check that button behaviors are set to *Single Action* in Lutron Designer
4. Ensure your passcode sequence uses valid action names (check spelling and underscores)

### Sequence Keeps Resetting

1. Check the **Next-In-Sequence Timeout** - you may need to increase it if you're pressing buttons slowly
2. Enable **Debug Mode** to see when and why timeouts occur
3. Verify the sequence you're entering matches the configured sequence exactly

### Wrong Action Triggering

Double-check your passcode sequence configuration for typos or incorrect action names. The blueprint validates action names and will show a persistent notification if invalid tokens are detected.

## Related Blueprints

For general button event handling and LED sync behavior, see the main [Lutron Sunnata Keypad Blueprints](README.md).

## Contributing

Found a bug or have a feature request? Please open an issue on GitHub!

## License

This blueprint is released under the Apache License 2.0.
