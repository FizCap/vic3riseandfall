````instructions
- Place new on_actions in `common/on_actions/`



## Understanding and Using Scopes

### What Are Scopes?

In Paradox scripting, **scope** determines what entity (country, character, state, etc.) a block of code refers to. Many bugs are caused by referencing the wrong scope, so always check and set scope explicitly.

### Key Scope Patterns

- **Default Scope:** Each on_action, effect, or trigger starts in a default scope (e.g., character, country, state). The default is set by the pulse or event that fires it.
- **Switching Scope:** Use keywords like `owner = { ... }`, `ruler = { ... }`, or `state = { ... }` to switch to another entity’s scope.
- **Nested Scopes:** You can nest scope switches to reach the correct context for your logic.

#### Example: Character On_Action with Country Check

```plaintext
et_random_character_death = {
  trigger = {
    is_character_alive = yes
    year >= 1454
    NOT = { owner = { has_technology_researched = pharmaceuticals } }
  }
  effect = {
    # ...effect logic...
  }
}
```
- Here, the on_action is in **character** scope (because it’s a character pulse).
- To check a country property, use `owner = { ... }` to switch to the character’s country.

#### Example: Accessing Monarch from Country Scope

```plaintext
trigger = {
  ruler = { has_trait = trait_et_great_reformer }
}
```
- This checks if the country’s ruler has a specific trait.

#### Example: State Scope from Country

```plaintext
effect = {
  random_owned_state = {
    # Now in state scope
    add_building = building_railway
  }
}
```

### Scope Reference

- **owner**: Switches from character to their country, or from state to owning country.
- **ruler**: Switches from country to its ruler character.
- **state**: Switches from country to a specific state.
- **any_** and **every_**: Loops over all entities of a type, changing scope for each iteration.

### Best Practices

- **Always check the docs:** See `docs/on_actions.log`, `docs/triggers.log`, and `docs/effects.log` for supported scopes.
- **Validate with vanilla:** If unsure, check `game/common/on_actions/00_code_on_actions.txt` for vanilla examples.
- **Explicit is better:** If your logic fails, try adding explicit scope switches.
- **Document your intent:** Comment scope switches in complex logic for future maintainers.

### Common Pitfalls

- **Forgetting to switch scope:** E.g., checking a country property in character scope without `owner = { ... }`.
- **Misusing `effect =` in scripted effects:** Only use `effect =` in on_actions, never in scripted effects themselves.
- **Assuming pulse scope:** Always confirm what entity the pulse or event is acting on.

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

Note: The `ai_strategies` scripting files use their own unique set of commands and structure, which are not fully documented in the standard scripting docs. For details and examples, consult the new reference file in the `docs/` folder or review the vanilla files in `common/ai_strategies/`.

If you do not know what to do, you can search the mod files or the vanilla game files for examples. There are many examples and commands found in almost every file, in both this mod and the vanilla files. Use these as references before asking for help.

## Character and Heir System Debugging

When working with character systems (especially heir creation), keep these critical points in mind:

### Character Creation for Heirs
Always include `heir = yes` when creating heir characters:
```
create_character = {
  age = { 0 8 }
  culture = ruler.culture
  religion = ruler.religion
  heir = yes  # Essential - without this, character won't be designated as heir
}
```

### On_Action Scope Management
- `on_monthly_pulse_character` operates in **character scope**, despite documentation saying "none"
- Use `owner = { ... }` to access country properties from character scope
- Use `ruler = { ... }` to access monarch from country scope
- Always validate scope by checking vanilla examples in `game/common/on_actions/00_code_on_actions.txt`

### Preventing Logic Loops in Modifier Systems
When creating systems that add/remove modifiers, include guards to prevent infinite loops:
```
et_heir_storage = {
  trigger = {
    is_monarch = yes
    NOT = { has_modifier = et_has_an_heir }  # Don't add if already present
    owner = { NOT = { any_scope_character = { is_heir = yes is_character_alive = yes } } }  # Don't add if heir exists
  }
  effect = {
    et_heir_storage_effect = yes
  }
}
```

### Debugging Process
1. Verify all referenced scripted effects and modifiers exist
2. Check that on_actions are properly registered in the main file
3. Trace the complete flow from trigger to effect
4. Validate scope usage at each step
5. Test probability values for game balance
6. Use `get_errors` tool to check for syntax issues

For detailed debugging information, see `docs/heir_system_debugging_guide.md`.
````
