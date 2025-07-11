- Place new on_actions in `common/on_actions/`

For any on_action created, ensure it includes some sort of pulse at the top (e.g., monthly, yearly, or custom pulse trigger).

## On_Action and Scripted Effect Patterns
Most on_action effects should simply have an `effect =` block that calls a scripted effect. For example:

```
et_homeland_cleanup = {
    effect = {
        et_homeland_cleanup_script = yes
    }
}
```

This pattern means the on_action triggers a scripted effect, which contains the actual logic.

**Important:** In scripted effects themselves, do **not** use `effect =`. Scripted effects should only contain the effect logic directly. Always remove any `effect =` block from inside scripted effects.
## Scripted_ Folders: Where to Place New Files
When adding new files for custom scripting, follow the vanilla folder structure:

- Place new scripted effects in `common/scripted_effects/`
- Place new scripted triggers in `common/scripted_triggers/`
- Place new scripted modifiers in `common/scripted_modifiers/`
- Place new scripted buttons in `common/scripted_buttons/`
- Place new scripted GUIs in `common/scripted_guis/`
- Place new scripted lists in `common/scripted_lists/`
- Place new scripted progress bars in `common/scripted_progress_bars/`
- Place new scripted rules in `common/scripted_rules/`
- Place new on_actions in `common/on_actions/`

For this mod, always use the `et_` prefix for new files in these folders (e.g., `et_my_new_effect.txt`).
## Scripting Keyword Patterns
* `Any_` is a trigger; it checks conditions on all elements
* `Random_` is an effect; it acts on one element
* `Every_` is an effect; it acts on all elements
# Copilot Instructions for Victoria3Timeline Mod

**Timeline Setting:** This mod takes place in 1444, starting at the end of the Europa Universalis IV era and extending the Victoria 3 timeline.
## Project Overview
- This is a total conversion mod for Victoria 3, extending the timeline and adding new mechanics, events, cultures, and countries, with a focus on historical plausibility and performance.
- The mod structure mirrors Paradox's own, with most content in `common/`, `events/`, `localization/`, and `docs/`.
- Key design goal: enable smooth conversion from Europa Universalis IV to Victoria 3, supporting a wide range of historical scenarios.

## Directory Structure & Key Files
- `common/`: Core game definitions (countries, cultures, scripted effects, decisions, etc.).
  - `scripted_effects/`: Contains reusable effect blocks (see `et_annex_decentralized_effects.txt`).
  - `decisions/`: Major gameplay decisions, including conversion-specific logic (see `zzz_converter_decisions.txt`).
  - `country_definitions/`, `cultures/`: Large, auto-generated files for converted content.
- `events/`: Historical and converter-specific events, organized by region or feature.
- `localization/`: All in-game text, with English as the primary language for new content.
- `docs/`: Generated documentation for custom triggers, effects, modifiers, on_actions, and event targets. Use these as references for scripting.

## Modding Patterns & Conventions
- All files should start with the prefix `et_` as a naming convention for this mod.
- Scripted effects and triggers are heavily reused; prefer referencing or extending existing blocks over duplicating logic.
- Converter-generated files are always prefixed with `zzz_` or `99_` to distinguish from hand-written content.
- Localization keys for new features use the `et_` or `zzz_` prefix.
- Performance optimizations: features like culture merging and decentralized nation annexation are controlled by game rules and settings, with logic in both scripted effects and localization.
- Comments in files often credit original authors or sources for transparency.

## Developer Workflows
- No build or test automation; changes are loaded directly in-game. Always validate changes by launching Victoria 3 with the mod enabled.
- For documentation, refer to the `docs/` folder for up-to-date lists of custom scripting elements.
- When adding new mechanics, update both the relevant `common/` files and ensure localization is provided in `localization/english/`.
- For converter logic, coordinate changes across `common/`, `events/`, and `localization/` to ensure consistency.

## Integration Points
- The mod is designed to work with save converters from EU4; expect large, auto-generated files and edge-case handling for non-standard countries/cultures.
- No external code dependencies; all scripting is in Paradox's scripting language.

## Example Patterns
- Scripted effect: see `common/scripted_effects/et_annex_decentralized_effects.txt` for reusable annexation logic.
- Decision logic: see `common/decisions/zzz_converter_decisions.txt` for how converter-specific decisions are structured.
- Localization: see `localization/english/extended_timeline_general_l_english.yml` for naming and description conventions.

---

For questions about scripting syntax or mod structure, consult the `docs/` folder or reference Paradox modding wikis. If unclear, ask for clarification or examples from the user.
