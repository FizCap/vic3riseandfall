# Heir System Debugging Guide

## Overview
This document outlines the debugging process and lessons learned from fixing the heir modifier system in Victoria 3 Timeline mod. The heir system consists of two main components: heir storage (adding heir modifiers to monarchs) and heir production (creating actual heir characters).

## System Architecture

### Components
1. **`et_heir_storage`** (on_action) - Triggered monthly for each character
2. **`et_heir_storage_effect`** (scripted effect) - Randomly adds heir modifier to monarch
3. **`et_heir_production`** (on_action) - Triggered monthly, creates heirs for countries with the modifier
4. **`et_has_an_heir`** (static modifier) - Applied to monarchs who are "expecting" an heir

### Triggering Structure
```
on_monthly_pulse_character = {
  on_actions = {
    et_heir_storage      # Adds heir modifier to eligible monarchs
    et_heir_production   # Creates heirs for countries with the modifier
  }
}
```

## Common Issues and Solutions

### Issue 1: Missing `heir = yes` in Character Creation
**Problem**: Creating characters without designating them as heirs
```vic3
create_character = {
  age = { 0 8 }
  culture = ruler.culture
  religion = ruler.religion
  # Missing: heir = yes
}
```

**Solution**: Always include `heir = yes` when creating heir characters
```vic3
create_character = {
  age = { 0 8 }
  culture = ruler.culture
  religion = ruler.religion
  heir = yes  # This is essential!
}
```

### Issue 2: Insufficient Trigger Conditions
**Problem**: On_actions triggering repeatedly without proper guards
```vic3
et_heir_storage = {
  trigger = {
    is_monarch = yes  # Too simple - will trigger every month
  }
  effect = {
    et_heir_storage_effect = yes
  }
}
```

**Solution**: Add comprehensive conditions to prevent unwanted behavior
```vic3
et_heir_storage = {
  trigger = {
    is_monarch = yes
    NOT = { has_modifier = et_has_an_heir }  # Don't add if already has modifier
    owner = { NOT = { any_scope_character = { is_heir = yes is_character_alive = yes } } }  # Don't add if heir already exists
  }
  effect = {
    et_heir_storage_effect = yes
  }
}
```

### Issue 3: Scope Confusion in On_Actions
**Problem**: Misunderstanding which scope an on_action operates in
- `on_monthly_pulse_character` operates in **character scope** (despite docs saying "none")
- This means triggers like `is_monarch = yes` work directly
- To access country properties, use `owner = { ... }`

**Key Learning**: Always check vanilla examples in `game/common/on_actions/00_code_on_actions.txt` for scope confirmation.

## Debugging Methodology

### Step 1: Verify Component Existence
1. Check that all referenced scripted effects exist
2. Verify modifier definitions are present
3. Confirm on_actions are properly registered

### Step 2: Trace the Flow
1. **Entry Point**: Check `et_on_actions.txt` for proper triggering
2. **Storage**: Verify `et_heir_storage` conditions and effects
3. **Production**: Confirm `et_heir_production` creates proper heirs
4. **Cleanup**: Ensure modifiers are removed after heir creation

### Step 3: Validate Scope Usage
- `on_monthly_pulse_character` = character scope
- Use `owner = { ... }` to access country properties
- Use `ruler = { ... }` in country scope to access monarch

### Step 4: Check for Logic Loops
Ensure the system doesn't create infinite loops:
- Don't add modifier if already present
- Don't create heir if one already exists
- Remove modifier after heir creation

## Best Practices

### Trigger Design
```vic3
et_heir_storage = {
  trigger = {
    is_monarch = yes                           # Base requirement
    NOT = { has_modifier = et_has_an_heir }    # Prevent duplicates
    owner = {
      NOT = {
        any_scope_character = {
          is_heir = yes
          is_character_alive = yes
        }
      }
    }                                          # Only if no existing heir
  }
  effect = {
    et_heir_storage_effect = yes
  }
}
```

### Character Creation
```vic3
create_character = {
  age = { 0 8 }
  culture = ruler.culture
  religion = ruler.religion
  heir = yes                    # Essential for heir designation
  # Optional: Add traits, names, etc.
}
```

### Probability Balancing
- Monthly percentages should be low (1-10%)
- Consider yearly probability: 5% monthly â‰ˆ 50% yearly
- Test different values for game balance

## Tools Used for Debugging

### File Search Tools
- `file_search` - Find files by pattern
- `grep_search` - Search for text within files
- `semantic_search` - Natural language code search

### Documentation References
- `docs/on_actions.log` - Expected scopes and triggers
- `docs/effects.log` - Available effects
- `docs/triggers.log` - Available triggers

### Validation Tools
- `get_errors` - Check syntax errors
- `read_file` - Examine file contents
- Victoria 3 in-game error console

## Testing Checklist

When implementing or modifying heir systems:

1. [ ] All referenced scripted effects exist
2. [ ] Static modifiers are properly defined
3. [ ] On_actions are registered in the main file
4. [ ] Trigger conditions prevent infinite loops
5. [ ] Character creation includes `heir = yes`
6. [ ] Scope usage is correct (character vs country)
7. [ ] Probability values are balanced
8. [ ] No syntax errors in any files
9. [ ] Test in-game to verify functionality

## Common File Locations
- On_actions: `common/on_actions/`
- Scripted effects: `common/scripted_effects/`
- Static modifiers: `common/static_modifiers/`
- Documentation: `docs/`

## Additional Notes
- Always test changes in-game before considering them complete
- Use descriptive comments in code for future debugging
- Consider edge cases like existing heirs, dead monarchs, etc.
- Balance gameplay impact with historical plausibility

## Heir Naming Limitations and Best Practices

### Inheriting Last Names in Heir Creation

Victoria 3's scripting system does not support dynamic inheritance of a character's last name (e.g., `last_name = ruler.last_name` is not valid). Attempts to reference another character's name property in `create_character` will not work and will result in either errors or the game generating a random name based on culture.

#### What Works
- If you omit `first_name` and `last_name` in `create_character`, the game will auto-generate a culturally appropriate name for the new character.
- You can specify a static last name (e.g., `last_name = "Kastrioti"`), but this must be hardcoded and cannot be dynamically inherited from the ruler.
- Character templates can be used for pre-defined dynasties, but not for dynamic inheritance.

#### What Does Not Work
- `last_name = ruler.last_name` or similar dynamic references are not supported in effect blocks.
- There is no built-in effect or trigger to copy a property from one character to another at creation.

#### Best Practice
- For generic heir creation, omit the `first_name` and `last_name` fields and let the game generate a name based on the heir's culture.
- For historical or special cases, use character templates or hardcoded names.

**Example:**
```vic3
create_character = {
  age = { 0 8 }
  culture = ruler.culture
  religion = ruler.religion
  heir = yes
}
```

This will ensure the heir is created with a plausible name, even if it does not match the ruler's last name.