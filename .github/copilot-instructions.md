# Notes: Random Building Generation Lessons Learned (2025-07-16)

### Common Mistakes & Fixes
- **Wrong Effect Command:** Used `add_building` instead of `create_building`. Only `create_building` works for scripted effects in Victoria 3.
- **Randomization Weights:** Initial weights for urban/industrial buildings were too high, resulting in too many buildings. Adjusted to 75% chance for none, 20% for level 1, 5% for level 2 for more realistic distribution.
- **Barracks Generation:** Originally allowed 0 barracks; user wanted 1-5 always. Fixed by removing the 0 option and distributing weights evenly across levels 1-5.
- **Effect Not Triggering:** Scripted effect (`et_add_random_buildings`) was not being called from on_actions or other logic. Always ensure new effects are actually invoked in the game flow.
- **Syntax Consistency:** All building creation must use the correct syntax: `create_building = { building = building_xxx level = N reserves = 1 }`.
- **Testing & Iteration:** Multiple rounds of testing and user feedback were needed to tune the weights and logic for desired gameplay results.

### Best Practices
- Always check the effect/trigger syntax in the docs or vanilla files before implementing.
- Use `random_list` for weighted randomization, and adjust weights to match gameplay intent.
- For effects that should never generate 0 (like barracks), remove the 0-weighted entry entirely.
- Document all changes and logic decisions in the instructions for future maintainers.
- Validate that all scripted effects are actually called from on_actions or events.
- When in doubt, test in-game and iterate based on results and player feedback.

# Disabling Journal Entries Based on Game Rules (Victoria 3 Modding)

## Best Practice Summary (from Turtle Island case)

To disable a journal entry when a specific game rule is enabled (e.g., culture merging):

- Add a check for the game rule in the `possible` block of the journal entry, like this:
  ```vic3
  possible = {
      ...existing conditions...
      NOT = { has_game_rule = culture_merging_enabled }
  }
  ```
- This ensures the journal entry cannot be started or completed if the game rule is enabled, regardless of other conditions.
- Placing the check in `is_shown_when_inactive` only hides the entry, but does not prevent it from being triggered or completed if already active. The `possible` block is the most reliable place for disabling activation.
- Always verify the exact name of the game rule and use the correct syntax for `has_game_rule`.
- If you want to hide the entry as well, you can also add the check to `is_shown_when_inactive`, but the `possible` block is essential for full disabling.

## Debugging Tips
- If the entry does not appear, test by removing the game rule check to isolate the issue.
- Confirm all other conditions are met for your country/state.
- Use vanilla files and documentation to verify syntax and scope.

---
# Scope Usage and Best Practices (Victoria 3)

## Understanding Scopes

- **Every effect and trigger only works in certain scopes** (e.g., country, state, state_region, character, etc.). Always check the docs for supported scopes before using an effect or trigger.
- **ROOT** refers to the original scope when the effect or trigger was called. **THIS** is the current scope. Use these for context passing, but prefer named scopes for clarity in complex scripts.
- **Saving Scopes:** Use `save_scope_as = my_scope` to save the current scope for later use. You can then reference it as `scope:my_scope` in effects or triggers.
- **Dynamic Scopes:** When using a saved scope, always reference it as `scope:your_scope_name` (e.g., `add_claim = scope:colonizer`). The `scope:` prefix is required for dynamic scope usage.
- **Scope Chains:** You can chain scopes, e.g., `owner = { save_scope_as = colonizer }` then later `add_claim = scope:colonizer` in a different scope.
- **Event Targets:** Some event targets (like `scope:suez_scope`) are set up in the event’s `immediate` or `option` blocks and then referenced in options or effects.

## Example Patterns

- Save a country scope, then use it later:
  ```vic3
  owner = { save_scope_as = colonizer }
  state_region = {
    add_claim = scope:colonizer
  }
  ```
- Use a saved scope in an event option:
  ```vic3
  s:STATE_SINAI = {
    add_claim = scope:suez_scope
  }
  ```

## Best Practices

- Always check the supported scopes for an effect or trigger in the docs.
- When using a saved scope, always use the `scope:` prefix.
- Save the scope before leaving it (before switching to another scope).
- Use clear and unique names for saved scopes to avoid confusion.
- Use ROOT and THIS for context, but prefer named scopes for clarity in complex scripts.

## Common Pitfalls

- Using a scope name without the `scope:` prefix for dynamic references (e.g., `add_claim = colonizer` will not work, must be `add_claim = scope:colonizer`).
- Not saving the scope before switching, resulting in an undefined reference.
- Using an effect or trigger in the wrong scope (e.g., `add_claim` only works in `state_region` scope).

Refer to vanilla events (like `canal_events.txt`) for working examples of advanced scope usage.
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

---
## Quick Reference: Scripting Conventions and Best Practices (from docs)

### Understanding Scope Keywords: ROOT, THIS, owner, prev, FROM, target
In Paradox scripting, these keywords help you control and reference different scopes:

- **ROOT**: The original scope when the effect or trigger was called. It’s the “starting point” and is preserved even as you switch scopes. For example, if an effect is called on a country, ROOT will always be that country, even if you switch to a state or character inside the effect.
- **THIS**: The current scope. As you use scope-switching commands (like `owner = { ... }` or `state_region = { ... }`), THIS changes to whatever you’re currently operating on.
- **owner**: A scope switch keyword. It changes the current scope to the owner of the current object (e.g., from a state to the country that owns it, or from a character to their country).
- **prev**: The previous scope before the last scope switch. Useful in loops or nested effects.
- **FROM**: Used in some event/trigger contexts to refer to the scope that triggered the event (not always available).
- **target**: Used when you have a named target in an event or effect.

**Example:**
```vic3
# In state scope
owner = { add_claim = THIS }  # THIS is the state, owner is the country
state_region = { add_claim = ROOT }  # ROOT is whatever called the effect (could be a country)
```

**Summary Table:**

| Keyword   | What it means                        | When to use                        |
|-----------|--------------------------------------|------------------------------------|
| ROOT      | The original scope                   | When you want to refer to the caller |
| THIS      | The current scope                    | When you want to refer to the current object |
| owner     | Switches to the owner of THIS        | To act in the owner’s scope        |
| prev      | The previous scope                   | For nested/looped effects          |
| FROM      | The event’s source scope (sometimes) | In events/triggers                 |
| target    | A named target scope                 | When set by an event/effect        |

### Linking Custom On_Actions to Vanilla Triggers
To hook custom logic into a vanilla on_action (such as `on_colony_created`), use this two-step pattern:

1. **On_Action Registration Block:**
   Register your custom on_action to fire on the vanilla trigger:
   ```vic3
   my_custom_action = {
       on_actions = {
           on_colony_created
       }
   }
   ```
2. **Effect Block:**
   Define the effect for your custom on_action, calling your scripted effect:
   ```vic3
   my_custom_action = {
       effect = {
           my_scripted_effect = yes
       }
   }
   ```
This pattern is required for custom logic to hook into vanilla on_actions in Victoria 3 modding. Always use this two-step approach for new on_action logic.

### Important Warning: Do Not Invent Syntax
- Do not make up or invent syntaxes (e.g., `has_colony`).
- Most supported syntaxes are listed in the `docs/` folder (`effects.log`, `triggers.log`, etc.).
- Always check the documentation before using a new effect, trigger, or keyword.
- If unsure, search the docs or vanilla files for valid examples.

### Scopes
- Each effect, trigger, or on_action starts in a default scope (e.g., country, character, state).
- Switch scope explicitly using keywords like `owner = { ... }`, `ruler = { ... }`, `state = { ... }`.
- Use `any_`, `every_`, and `random_` for looping or acting on multiple entities.
- Always check and set scope explicitly to avoid bugs.

#### Example Scope Switches
```plaintext
owner = { ... }         # Switch from character/state to country
ruler = { ... }         # Switch from country to its ruler character
random_owned_state = { ... }  # Switch from country to a random state
```

### On_Actions
- Place new on_actions in `common/on_actions/`.
- Most on_actions should have an `effect = { ... }` block that calls a scripted effect.
- Example:
  ```
  et_homeland_cleanup = {
      effect = {
          et_homeland_cleanup_script = yes
      }
  }
  ```
- In scripted effects, do NOT use `effect =`; only use it in on_actions.

### Scripted Effects/Triggers/Modifiers
- Place new scripted effects in `common/scripted_effects/` (prefix: `et_`).
- Place new scripted triggers in `common/scripted_triggers/` (prefix: `et_`).
- Place new scripted modifiers in `common/scripted_modifiers/` (prefix: `et_`).
- Follow vanilla folder structure for all custom scripting.

### Scripting Keyword Patterns
- `any_` is a trigger (checks all elements)
- `random_` is an effect (acts on one element)
- `every_` is an effect (acts on all elements)

### Character and Heir System
- Always use `heir = yes` when creating heirs:
  ```
  create_character = {
    age = { 0 8 }
    culture = ruler.culture
    religion = ruler.religion
    heir = yes
  }
  ```
- For on_actions like `on_monthly_pulse_character`, use `owner = { ... }` to access country properties from character scope.

### Debugging and Validation
- Validate all referenced scripted effects and modifiers exist.
- Check that on_actions are registered in the main file.
- Trace the flow from trigger to effect and validate scope usage at each step.
- Use `get_errors` to check for syntax issues.
- Refer to `docs/on_actions.log`, `docs/triggers.log`, `docs/effects.log` for supported scopes and examples.

### General Best Practices
- All files and localization keys for new features use the `et_` prefix.
- Prefer referencing/extending existing scripted blocks over duplicating logic.
- For converter logic, coordinate changes across `common/`, `events/`, and `localization/`.
- Document intent with comments, especially for complex scope switches.

---
## Useful Documentation Files
- `docs/effects.log`: List of effects and their supported scopes.
- `docs/triggers.log`: List of triggers and their supported scopes.
- `docs/modifiers.log`: List of modifiers and their descriptions.
- `docs/on_actions.log`: List of on_actions and their expected scopes.
- `docs/event_targets.log`: List of event targets and their input/output scopes.

---
## Example: Preventing Modifier Loops
```plaintext
et_heir_storage = {
  trigger = {
    is_monarch = yes
    NOT = { has_modifier = et_has_an_heir }
    owner = { NOT = { any_scope_character = { is_heir = yes is_character_alive = yes } } }
  }
  effect = {
    et_heir_storage_effect = yes
  }
}
```

---
## For More Details
- See the `docs/` folder for up-to-date lists and debugging guides.
- If unsure, check vanilla files in `game/common/` for examples.

---

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
