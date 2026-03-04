# {{product_name}} — Data Model

## AppNest storage

All persistent state must use the AppNest **$db** API. Types: `string`, `number`, `list`, `map`, `boolean`. Do not use in-memory state for cross-invocation data. See `appnest-tools/appnest-governance/02-sdk/05-SDK-Reference.md`.

---

## Core entities

{{core_entities}}

| Entity | Purpose | Key pattern (e.g. for $db) | $db type | Notes |
|--------|---------|-----------------------------|----------|--------|
| {{entity_1_name}} | {{entity_1_purpose}} | {{entity_1_key_pattern}} | {{entity_1_db_type}} | {{entity_1_notes}} |
| {{entity_2_name}} | {{entity_2_purpose}} | {{entity_2_key_pattern}} | {{entity_2_db_type}} | {{entity_2_notes}} |
| {{entity_3_name}} | {{entity_3_purpose}} | {{entity_3_key_pattern}} | {{entity_3_db_type}} | {{entity_3_notes}} |

## Storage keys

{{storage_keys}}

- **Key format:** Keys are strings (max 1000 chars). Document prefix and variable parts (e.g. `mappings_{{surveyId}}`).
- **Value shape:** For `string` type, document whether value is JSON-serialised; for `map`/`list`, document structure.

## Sensitive data

- **Secrets:** Stored only via platform (installation_params, oauth). Never in $db as plain text unless encrypted by platform.
- **PII:** {{pii_handling}}

## External system data (if any)

{{external_system_data}}

(If the app pushes or pulls data from an external system, document which fields are stored in $db for idempotency or retry (e.g. external IDs, last sync timestamp).)
