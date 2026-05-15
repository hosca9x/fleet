# Rollback Command

Rollback a previously applied Fleet infrastructure change or database migration.

## Usage

```
/opsx rollback [target] [--to <version|timestamp>] [--dry-run] [--force]
```

## Arguments

- `target` — The component to rollback: `migration`, `config`, `deployment`, or `all`
- `--to` — Specific version, migration ID, or timestamp to rollback to
- `--dry-run` — Preview what would be rolled back without making changes
- `--force` — Skip confirmation prompts (use with caution)

## Behavior

When invoked, this command will:

1. **Identify the current state** by reading the active deployment metadata and migration history
2. **Determine the rollback target** based on provided arguments or interactively prompt
3. **Validate rollback feasibility** — check for irreversible migrations, data loss risks, or dependency conflicts
4. **Present a rollback plan** showing exactly what will change
5. **Execute the rollback** after confirmation (unless `--force` is set)
6. **Verify post-rollback health** by running basic connectivity and sanity checks

## Examples

### Rollback the last database migration
```
/opsx rollback migration
```

### Rollback to a specific migration version
```
/opsx rollback migration --to 20231105120000
```

### Preview a deployment rollback without applying
```
/opsx rollback deployment --dry-run
```

### Force rollback all components to a timestamp
```
/opsx rollback all --to 2024-01-15T10:30:00Z --force
```

## Safety Checks

Before executing any rollback, the agent MUST:

- Confirm the target environment (never assume production)
- Check if the rollback is destructive (e.g., drops columns, deletes data)
- Verify a recent backup exists if data loss is possible
- Warn about any downstream services that may be affected
- Log the rollback action with timestamp and operator identity

## Output Format

The agent should produce structured output:

```
[ROLLBACK PLAN]
Target:      migration
Current:     20240115_add_policy_results_table
Rolling to:  20240110_add_host_software_index
Migrations:  3 will be reversed
Data risk:   LOW — index-only changes
Backup:      fleet-db-2024-01-15-09:00 (available)

[MIGRATIONS TO REVERSE]
  - 20240115_add_policy_results_table (drops table: policy_results)
  - 20240113_add_host_display_name_index (drops index)
  - 20240111_add_software_cve_column (drops column: software.cve)

Proceed? [y/N]
```

## Error Handling

- If no rollback point is available, report clearly and suggest alternatives
- If the database is unreachable, abort immediately with connection details
- If a partial rollback occurs, report exactly which steps succeeded and which failed
- Never leave the system in an unknown intermediate state without reporting it

## Related Commands

- `/opsx apply` — Apply pending migrations or config changes
- `/opsx status` — View current deployment and migration state
- `/opsx diff` — Compare two versions or states
- `/opsx archive` — Archive current state before making changes
