# CLAUDE.md — home-assistant-lutron-sunnata-blueprints

## Project Overview

Home Assistant automation blueprints (YAML) for Lutron Sunnata RadioRA3 keypads. Extends keypad functionality beyond Lutron Designer by enabling advanced button detection (single-click, double-click, long press) and LED synchronization with non-Lutron devices.

## Repository Structure

```
blueprints/automation/
  lutron-sunnata-keypad-click-long-click-double-click.yaml  # Button press detection
  lutron-sunnata-keypad-led-control-blueprint.yaml           # LED-to-entity sync
  lutron-sunnata-keypad-combination-lock.yaml                # Button sequence detection
```

No build system, test suite, or CI/CD. Validation is manual via `yamllint` or Home Assistant blueprint import.

## Tech Stack

- **Language:** YAML (Home Assistant blueprint format) with Jinja2 templates
- **Platform:** Home Assistant (min version 2024.6.0)
- **Integration:** `lutron_caseta` (Caseta/RadioRA3)

## Key Patterns

### Interaction Flow

`lutron_caseta` integration -> `lutron_caseta_button_event` events -> blueprint trigger (press/release) -> blueprint logic (`wait_for_trigger`, `choose`, `parallel`) -> `!input` actions executed

### Naming Conventions

- Button inputs: `button_<n>_single_click`, `button_<n>_double_click`, `button_<n>_long_press`, `button_<n>_release`
- Input groups: `button_<n>_group`
- Special buttons: `button_lower_*`, `button_raise_*`
- Leap mapping: `leap_1`-`leap_4` (buttons 1-4), `leap_18` (lower), `leap_19` (raise)
- Advanced settings grouped under `advanced_settings_group`

### Detection Logic

- Long press: `wait_for_trigger` with `long_press_timeout` (default 500ms), then `parallel` block for long-press action + release detection
- Double click: `wait_for_trigger` with `double_click_timeout` (default 250ms)
- `long_press_duration` calculated in ms and passed to release actions
- Combination lock: stores progress in `input_text` helper, validates against allowed tokens, resets on timeout (default 2s)

### Configuration Defaults

| Setting | Default | Range | Step |
|---------|---------|-------|------|
| Long Press Timeout | 500ms | 100-2000ms | 50ms |
| Double Click Timeout | 250ms | 100-1000ms | 50ms |
| Max Long Press Duration | 60s | 1-3600s | 1s |
| Combination Lock Timeout | 2s | 1-10s | 1s |

## Editing Guidelines

- Preserve `blueprint:` schema structure, `domain: automation`, and `homeassistant.min_version` keys
- Do NOT rename `!input` identifiers or leap-number keys — they are referenced across triggers, variables, and actions
- Follow existing naming conventions when adding inputs
- Mirror existing `choose`, `wait_for_trigger`, and `parallel` patterns for logic changes
- Variables map leap names to `!input` entries; keep this mapping structure intact
- Validate YAML with `yamllint` before submitting changes
- Suggest the smallest possible patch; describe user-visible behavior changes

## Supported Devices

Sunnata 2-button, 3-button (with raise/lower), and 4-button keypads, including hybrid variants. All filtered by `integration: lutron_caseta` in device selectors.
