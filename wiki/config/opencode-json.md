---
title: "opencode.json"
description: "Configuration file for opencode.ai skill permissions and schema validation"
category: "config"
source_files:
  - "opencode.json"
created: "2026-04-27"
last_updated: "2026-04-27"
---

# opencode.json

The `opencode.json` file is the root configuration file for the opencode.ai integration. It defines the JSON schema validation and permission rules for skill access within the project.

## Schema

```json
{
  "$schema": "https://opencode.ai/config.json",
  "permission": {
    "skill": {
      "<skill-pattern>": "<allow|deny>"
    }
  }
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `$schema` | string | JSON Schema URL for validation (readonly, must be `https://opencode.ai/config.json`) |
| `permission` | object | Top-level container for permission rules |
| `permission.skill` | object | Skill access control rules keyed by pattern |
| `permission.skill.<pattern>` | string | Permission action: `"allow"` or `"deny"` |

## Skill Permission Configuration

The `skill` section controls access to skills based on naming patterns. Each key is a glob-style pattern matched against skill names.

### Pattern Matching

- `*` matches any sequence of characters
- Patterns are evaluated in order; first match determines the outcome

### Example Configurations

**Allow all wiki skills:**

```json
{
  "$schema": "https://opencode.ai/config.json",
  "permission": {
    "skill": {
      "wiki-*": "allow"
    }
  }
}
```

**Allow specific skills, deny others:**

```json
{
  "$schema": "https://opencode.ai/config.json",
  "permission": {
    "skill": {
      "wiki-*": "allow",
      "*-internal": "deny"
    }
  }
}
```

**Restrict to read-only skills:**

```json
{
  "$schema": "https://opencode.ai/config.json",
  "permission": {
    "skill": {
      "wiki-read": "allow",
      "wiki-write": "deny"
    }
  }
}
```

## Current Project Configuration

The llm-wiki-opencode project uses the following configuration:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "permission": {
    "skill": {
      "wiki-*": "allow"
    }
  }
}
```

This grants the `wiki-*` pattern (matching `wiki-conventions`, `wiki-templates`, etc.) full access via the `allow` action.
