<!-- Copilot / AI agent instructions for repository contributors -->
# Copilot Instructions — home-assistant-lutron-sunnata-blueprints

Purpose
- Help an AI agent quickly understand and make safe, high-value changes to this repository: Home Assistant automation blueprints for Lutron Sunnata keypads.

Quick repo snapshot
- Primary content: `blueprints/automation/*.yaml` — Home Assistant blueprint files.
- No build system or automated tests in repo; validation is done by Home Assistant or YAML linters.

Big picture (architecture & why)
- This repo contains Home Assistant automation blueprints (YAML) that implement button detection and LED synchronization logic for Lutron Sunnata keypads.
- Interaction flow: `lutron_caseta` integration → `lutron_caseta_button_event` events → blueprint `trigger` (press/release) → blueprint logic (`wait_for_trigger`, `choose`, `parallel`) → `!input` actions executed (scripts, services).
- Example files: `blueprints/automation/lutron-sunnata-keypad-click-long-click-double-click.yaml` and `blueprints/automation/lutron-sunnata-keypad-led-control-blueprint.yaml`.

Key patterns and conventions (find and follow these exactly)
- Blueprint inputs: inputs live under `blueprint.input` and are referenced with `!input`. E.g. button actions are grouped as `button_1_group` / `button_2_group`, etc.
- Leap number mapping: the code maps physical keypad actions to `leap_1`..`leap_4`, and `leap_18`/`leap_19` for Lower/Raise. See `variables: double_click_actions` in the main blueprint.
- Detection logic: long press detection uses `wait_for_trigger` with `long_press_timeout`, then uses a `parallel` block to run the long-press sequence while waiting for release to compute `long_press_duration`.
- Event trigger type: `lutron_caseta_button_event` filtered by `device_id` and `action`. Use these exact keys when adding or modifying triggers.
- Configuration defaults: timeouts and limits are defined as numeric inputs (`long_press_timeout`, `double_click_timeout`, `max_long_press_duration`). Keep ranges consistent with existing selectors.

Integration points & external dependencies
- Requires the Home Assistant `lutron_caseta` integration (see model filters in each blueprint's `selector.device`).
- End users validate behavior by importing the blueprint into Home Assistant or copying the YAML to `config/blueprints/automation/` and reloading automations (see README.md usage instructions).

Editing and validation guidance for AI edits
- Small changes (typo fixes, docs, default values): edit YAML and keep `blueprint:` schema intact. Preserve `domain: automation` and `homeassistant.min_version` keys.
- Logic changes (triggers/variables/actions): keep the mapping structure intact — `variables` map leap names to `!input` entries; `actions` use `choose`, `wait_for_trigger`, and `parallel` patterns. Mirror existing structure for readability.
- When adding inputs, follow existing naming: `button_<n>_single_click`, `button_<n>_double_click`, `button_<n>_long_press`, `button_<n>_release` and place them under the corresponding `button_<n>_group` block.
- Validate YAML with a linter (recommended): `yamllint` or Home Assistant's blueprint importer. Manual functional validation requires a running Home Assistant instance with `lutron_caseta`.

Examples (actionable snippets from repo)
- Trigger listens for Lutron events:
  - `event_type: lutron_caseta_button_event` with `event_data.device_id: "{{ sunnata_keypad_device }}"`
- Variable mapping (from blueprint):
  - `double_click_actions: leap_1: !input button_1_double_click` (and similar entries for each physical button)
- Long press duration is calculated as `long_press_duration: "{{ (as_timestamp(now()) - as_timestamp(long_press_start_time)) * 1000 }}"` and passed to release actions.

What NOT to change
- Do not reformat or remove the `blueprint:` header or change `domain: automation` structure.
- Avoid renaming `!input` identifiers and leap-number keys — these are referenced across triggers, variables, and actions.

If you need more context
- Read the README at [README.md](README.md) for usage and troubleshooting guidance.
- Inspect the two main blueprints in `blueprints/automation/` for patterns and examples.

After edits
- Suggest the smallest possible patch and describe intent in PR description: what user-visible behavior changed and how to validate in Home Assistant.

Questions or missing info
- If anything is unclear, ask the maintainer for a Home Assistant validation workflow or an example integration test scenario.

— End of instructions
