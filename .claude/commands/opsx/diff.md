# opsx diff

Analyze and summarize differences between Fleet configurations, database migrations, or code changes to help understand the impact of modifications.

## Usage

```
/opsx diff [target] [options]
```

## Arguments

- `target`: What to diff (migration, config, schema, pr)
- `options`: Additional context or comparison points

## Behavior

When invoked, this command will:

1. **Identify the diff target** — Determine what type of comparison is being requested (migration files, config changes, schema evolution, or PR changes)
2. **Gather context** — Collect relevant files, git history, or database schema information
3. **Analyze impact** — Assess the risk and scope of changes
4. **Summarize findings** — Provide a structured summary with actionable insights

## Examples

### Diff a database migration
```
/opsx diff migration 20240115_add_user_roles
```
Analyzes the migration file and reports:
- Tables/columns added, modified, or dropped
- Index changes
- Data transformation risks
- Rollback complexity

### Diff configuration changes
```
/opsx diff config staging production
```
Compares environment configurations and highlights:
- Security-sensitive differences
- Feature flag divergence
- Infrastructure parameter changes

### Diff a PR
```
/opsx diff pr 1234
```
Summarizes pull request changes:
- Files changed by category (API, DB, UI, tests)
- New dependencies introduced
- Breaking changes detected
- Test coverage delta

## Output Format

The command produces a structured report:

```
## Diff Summary: <target>

### Overview
- Type: <migration|config|schema|pr>
- Risk Level: <low|medium|high|critical>
- Reversible: <yes|no|partial>

### Changes
<categorized list of changes>

### Impact Analysis
<description of downstream effects>

### Recommendations
<actionable next steps>
```

## Risk Assessment

Risk levels are assigned based on:
- **Low**: Additive changes only (new columns with defaults, new tables)
- **Medium**: Modifications to existing structures with backward compatibility
- **High**: Destructive changes, constraint modifications, or data transformations
- **Critical**: Changes affecting authentication, authorization, or data integrity

## Notes

- For migration diffs, always check if a corresponding `down` migration exists
- Config diffs should flag any secrets or credentials accidentally included
- PR diffs should cross-reference the linked GitHub issue for context
- When comparing schemas, use the canonical source in `server/datastore/mysql/schema.sql`
