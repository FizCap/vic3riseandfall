# Victoria3Timeline Modding Instructions

## Introduction & Purpose
This file merges all key lessons, best practices, and conventions for modding Victoria 3 in the Victoria3Timeline project. It is designed for both new contributors and Copilot to avoid common pitfalls and follow vanilla patterns.

---

## 1. Lessons Learned & Common Pitfalls

### Research Debuff/Game Rule Implementation
- Register on_actions with `effect = { ... }` blocks; never use `effect =` inside scripted effects.
- Permanent/recurring country modifiers go in `static_modifiers`, not `scripted_modifiers`.
- Game rule logic must be placed correctly; debuffs only apply when the rule is enabled.
- Use Paradox-standard localization keys (e.g., `rule_et_research_debuff_rule`).
- Use block syntax for temporary modifiers; omit `days` for permanent ones.
- Always check for existing modifiers before adding new ones.
- Add logic to remove time-limited modifiers when their window ends (e.g., after 1820).
- Add localization for all scripted effects that show up in UI/logs.

### Random Building Generation
- Use `create_building`, not `add_building`, in scripted effects.
- Adjust randomization weights for realistic building distribution.
- For effects that should never generate 0 (e.g., barracks), remove the 0-weighted entry.
- Ensure all scripted effects are actually called from on_actions/events.
- Use correct syntax: `create_building = { building = building_xxx level = N reserves = 1 }`.

### Disabling Journal Entries by Game Rule
- Add game rule checks in the `possible` block of journal entries to fully disable them.
- Use `has_game_rule` with the correct rule name.
- Placing the check in `is_shown_when_inactive` only hides the entry; use `possible` for disabling activation.

---

## 2. Best Practices
- Follow vanilla patterns for on_actions, scripted effects, and localization keys.
- Use the correct folder for static vs. scripted modifiers.
- Double-check game rule logic and flag usage.
- Use block syntax for temporary modifiers, omit days for permanent ones.
- Always check for existing modifiers before adding new ones.
- Add logic to remove time-limited modifiers when their window ends.
- Make sure all scripted effects and on_actions have proper localization.

---

## 3. Scope Usage & Patterns
- Each effect/trigger/on_action starts in a default scope (country, character, state, etc.).
- Switch scope explicitly using keywords: `owner = { ... }`, `ruler = { ... }`, `state = { ... }`.
- Use `any_`, `every_`, and `random_` for looping/acting on multiple entities.
- Use `save_scope_as = my_scope` and reference as `scope:my_scope` for dynamic scopes.
- Always use the `scope:` prefix for dynamic scope references.
- See vanilla files and `docs/` for supported scopes and examples.

---

## 4. Directory Structure & File Placement
- Place new on_actions in `common/on_actions/`.
- Place new scripted effects in `common/scripted_effects/` (prefix: `et_`).
- Place new scripted triggers in `common/scripted_triggers/` (prefix: `et_`).
- Place new scripted modifiers in `common/scripted_modifiers/` (prefix: `et_`).
- Follow vanilla folder structure for all custom scripting.

---

## 5. Scripting Keyword Patterns
- `any_` is a trigger (checks all elements)
- `random_` is an effect (acts on one element)
- `every_` is an effect (acts on all elements)

---

## 6. Debugging & Validation
- Validate all referenced scripted effects and modifiers exist.
- Check that on_actions are registered in the main file.
- Trace the flow from trigger to effect and validate scope usage at each step.
- Use `get_errors` to check for syntax issues.
- Refer to `docs/` for up-to-date lists and debugging guides.

---

## 7. Modding Conventions
- All files and localization keys for new features use the `et_` prefix.
- Prefer referencing/extending existing scripted blocks over duplicating logic.
- Converter-generated files use `zzz_` or `99_` prefix.
- Localization keys for new features use the `et_` or `zzz_` prefix.
- Document intent with comments, especially for complex scope switches.

---

## 8. Example Patterns

### On_Action Registration & Scripted Effect
```vic3
et_homeland_cleanup = {
    effect = {
        et_homeland_cleanup_script = yes
    }
}
```

### Character Creation for Heirs
```vic3
create_character = {
  age = { 0 8 }
  culture = ruler.culture
  religion = ruler.religion
  heir = yes
}
```

### Preventing Modifier Loops
```vic3
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

## 9. Useful Documentation Files
- `docs/effects.log`: List of effects and their supported scopes.
- `docs/triggers.log`: List of triggers and their supported scopes.
- `docs/modifiers.log`: List of modifiers and their descriptions.
- `docs/on_actions.log`: List of on_actions and their expected scopes.
- `docs/event_targets.log`: List of event targets and their input/output scopes.

---

## 10. Developer Workflow
- No build/test automation; validate changes by launching Victoria 3 with the mod enabled.
- Update both relevant `common/` files and localization for new mechanics.
- Coordinate changes across `common/`, `events/`, and `localization/` for converter logic.
- Use `docs/` for up-to-date lists of custom scripting elements.

---

For questions about scripting syntax or mod structure, consult the `docs/` folder or reference Paradox modding wikis. If unclear, search the mod files or vanilla game files for examples.
