# Victoria 3 AI Strategies Scripting Reference

This document summarizes the structure, commands, and conventions used in Victoria 3's `ai_strategies` scripting files. These files control AI behavior and are found in `common/ai_strategies/`.

---

## Overview
- AI strategies define how the AI prioritizes actions, goals, and behaviors for countries.
- Each strategy is a block with a unique name and contains parameters, triggers, and weights that influence AI decision-making.
- Many commands and parameters in `ai_strategies` are unique and not found in standard scripting docs.

## Structure & Key Commands

- **ai_strategy_name = { ... }**  
  The main block for each AI strategy. Each strategy has a unique name (e.g., `ai_strategy_default`).

- **icon = ...**  
  Path to the icon for the strategy (for reference/documentation; often not shown in-game).

- **will_form_power_bloc = { ... }**  
  Trigger block. If true for any active strategy, the AI will try to form a power bloc.

- **desired_tax_level / max_tax_level / min_tax_level**  
  Sets the AI's preferred, maximum, and minimum tax levels.

- **undesirable_infamy_level / unacceptable_infamy_level = { value = ... }**  
  Infamy thresholds for AI behavior regarding wargoals.

- **ideological_opinion_effect_mult = { value = ... }**  
  Multiplier for how much government ideology differences affect AI diplomatic acceptance.

- **revolution_aversion = { value = ... if = { ... } }**  
  Aversion to enacting laws that could spark civil war, with conditional modifiers.

- **min_law_chance_to_pass / max_progressiveness / max_regressiveness = { value = ... }**  
  Law-passing thresholds and AI willingness to be progressive or regressive.

- **diplomatic_play_support / diplomatic_play_neutrality / diplomatic_play_boldness = { value = ... if = { ... } }**  
  Modifiers for AI behavior in diplomatic plays, with detailed conditional logic.

- **wargoal_maneuvers_fraction = { value = ... if = { ... } }**  
  Fraction of maneuvers AI is willing to use for wargoals, with conditional modifiers.

- **change_law_chance = { value = ... }**  
  Chance each update that the AI will try to change a law.

- **pro_interest_groups / anti_interest_groups = { ... }**  
  Interest groups the AI prefers or avoids in government.

- **institution_scores = { institution_name = { value = ... } ... }**  
  AI preference for investing in specific institutions.

- **obligation_value = { value = ... if = { ... } multiply = { ... } }**  
  How much the AI values obligations from other countries, with complex conditional logic.

- **state_value / treaty_port_value = { value = ... if = { ... } }**  
  How much the AI desires a state or treaty port, with many nested conditions.

- **subject_value = { ... }**  
  How much the AI desires another country as a subject.

---

## Example Pattern

```vic3
ai_strategy_default = {
    icon = "gfx/interface/icons/ai_strategy_icons/placate_population.dds"
    desired_tax_level = medium
    max_tax_level = high
    min_tax_level = low
    undesirable_infamy_level = { value = 50 }
    unacceptable_infamy_level = { value = 100 }
    ideological_opinion_effect_mult = { value = 1.0 }
    revolution_aversion = {
        value = 5
        if = {
            limit = { has_journal_entry = je_sick_man_syria game_date < 1845.1.1 }
            add = 100
        }
    }
    # ...more parameters and conditional logic...
}
```

---

## Notes & Best Practices
- Not all triggers and effects from other scripting contexts are valid here; some are exclusive to AI logic.
- Use the vanilla files in `common/ai_strategies/` as references for valid commands, structure, and best practices.
- Many blocks support conditional logic using `if = { ... }`, `else_if = { ... }`, and `else = { ... }`.
- Some parameters (like `diplomatic_play_support`) require a `desc` for tooltip localization.
- For complex AI behavior, study the default and other vanilla strategies for patterns and advanced usage.

---

For further details, always consult the latest vanilla files and experiment in-game to validate AI behavior.
