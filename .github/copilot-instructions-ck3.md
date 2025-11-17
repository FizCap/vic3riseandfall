# Copilot instructions for the Rise and Fall mod â€” LLM-friendly (CK3)

Purpose
- Make this repository immediately usable by automated agents and human contributors. This file combines repo-specific rules with concrete CK3 modding patterns and an explicit LLM "contract" so an agent knows how to act safely and effectively.

Quick start checklist (high value, do these first)
- Open `descriptor.mod` and confirm: name, path, supported_version.
- Read `docs/triggers.log` and `docs/event_scopes.log` to confirm scope tokens and trigger signatures.
- Search `events/`, `common/`, and `localization/english/` for the IDs you plan to change.
- Never edit the global `game/` folder â€” place overrides under this repo.

LLM contract (short, strict)
- Always prefer vanilla engine patterns from `game/` and `docs/` logs. Do not invent new engine tokens.
- For any change: produce a 2â€“4 bullet contract (inputs, outputs, data shape, error modes).
- Run local validation steps after edits: syntax check (brace balance), duplicate-ID scan, and verify localization coverage.
- When editing multiple files, include a concise summary list of changed files and why.
- If a required file or engine log is missing, stop and ask for it instead of guessing.

Repository structure (one-paragraph)
- CK3 mod; data-driven text files. No compile step â€” the game loads scripts directly. Primary folders: `common/` (defines, scripted_effects, modifiers), `events/`, `localization/english/`, and `docs/` (engine-generated reference logs).

Minimal engineering contract for script changes
- Inputs: file(s) modified (path + brief purpose), expected in-game scope(s), and any new localization keys.
- Outputs: changed game behavior (short description), localization entries, and tests/method to validate.
- Success criteria: files load without engine errors, loc keys present, and behavior reproduces in a minimal in-game test.
- Error modes: missing loc keys (raw keys shown), silent trigger scope mismatches, duplicate IDs causing collisions.

Best-practice checklist for edits
- Use `riseandfall.` as your ID prefix when adding events/effects/modifiers.
- Place permanent modifiers in `common/static_modifiers/`; put temporary or computed modifiers in `common/scripted_modifiers/`.
- Register `on_actions` in `common/on_action/` and call scripted effects with `effect = { your_effect = yes }`.
- Always add `l_english:` entries for any UI-facing keys.
- Use `docs/` logs to validate valid tokens and scopes before saving changes.

Edge cases & gotchas (short list)
- Mis-scoped trigger: trigger silently does nothing â€” verify scope via `docs/event_scopes.log`.
- Missing localization: the game displays raw keys â€” add corresponding `localization/english/*.yml` entries.
- Duplicate IDs: first file wins â€” run a duplicate-ID scan before committing.
- Infinite modifier loops: always check `has_modifier` before applying a modifier and include removal logic.

Scope patterns & helpers
- Default scope depends on file location. To change scope explicitly use `owner = { ... }`, `ruler = { ... }`, `state = { ... }`.
- Iteration helpers: `any_` (trigger check), `random_` (effect helper for a single element), and `every_` (apply to all matched elements).
- Save dynamic scopes with `save_scope_as = X` and reference them as `scope:X` when needed.

Small examples (copyable patterns)
- on_action registration
  riseandfall_example_on_action = {
      effect = {
          riseandfall.example_effect = yes
      }
  }

- scripted effect calling create_character
  riseandfall.make_heir = {
    create_character = {
      age = { 0 8 }
      culture = ruler.culture
      religion = ruler.religion
      heir = yes
    }
  }

- defensive modifier application
  riseandfall_safe_apply = {
    if = {
      limit = { NOT = { has_modifier = riseandfall_temp_mod } }
      add_modifier = { name = riseandfall_temp_mod duration = 365 }
    }
  }

Debugging & quality gates (run these before committing)
1) Brace balance & syntax: visually or via editor plugin.
2) Duplicate-ID scan (suggested helper): `riseandfall/tools/duplicate-ids.ps1`.
3) Missing localization scan: `riseandfall/tools/missing-loc.ps1` (find referenced keys without `localization/` entries).
4) Smoke test: load CK3 with the mod enabled and verify the minimal repro (one event or effect triggers as expected).

Developer helper scripts (suggested)
- `riseandfall/tools/duplicate-ids.ps1` â€” scans repo for duplicate event/effect IDs.
- `riseandfall/tools/missing-loc.ps1` â€” finds referenced localization keys that are missing in `localization/english/` files.

When to ask for help
- Provide failing engine log lines, the exact file(s) you edited, and a minimal reproduction scenario when asking for troubleshooting.

Files & docs to consult
- `docs/triggers.log`, `docs/event_scopes.log`, `docs/effects.log`, `docs/modifiers.log`, `docs/on_actions.log`, `docs/event_targets.log`.

---

# CK3 -> Victoria 3 porting

When porting CK3 code or GUI patterns into Victoria 3: the game reuses many GUI tokens and some scripted systems, but tokens and scopes sometimes differ. Use these steps:

1) Search the V3 `game/` folder for the token or GUI name you want to copy.
2) If a token is missing, consult the V3 `docs/` logs (regenerate them if needed) to see what the engine supports.
3) GUI: check `game/gui/window_component_library.gui` and `game/gui/frontend/` â€” V3 keeps some CK3 names (like `button_standard`) as aliases so front-end copy/paste from CK3 may work; always open the V3 file to verify assets.
4) `on_actions` in V3 have names like `on_monthly_pulse`, `on_game_started_after_lobby` â€” review their existing behaviour before adding new ones.
5) V3 `journal_entries` use `concept_journal_entry` and `je_` localization keys rather than CK3 `je_` patterns in some cases; check `game/localization/english/`.

These notes are not exhaustive; when in doubt, ask for the specific token or file and I'll compare it with both CK3 and V3 `docs` outputs.