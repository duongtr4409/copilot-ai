# Migration Quality Checklist

## Safety
- Migration files are versioned and strictly ordered.
- Liquibase changeSet IDs are unique and immutable after release.
- Destructive operations are isolated and explicitly approved.
- Rollback or fallback path is documented.

## Compatibility
- Expand-then-contract is used for backward-compatible rollout.
- Old and new app versions can coexist during transition.
- Schema changes are aligned to release plan.

## Performance
- Index strategy is defined and tested on realistic data volume.
- Lock duration risk is assessed for each DDL operation.
- Backfill jobs are chunked and monitored.

## Observability
- Migration execution logs are centralized.
- Success/failure metrics and alert thresholds are defined.
- Post-migration verification queries are documented.

## Environment Sync
- Dev, Staging, Production apply identical migration history.
- Migration checksums are tracked and reviewed in release notes.
- Drift detection process exists and is run before production rollout.
- Liquibase validate/updateSQL is executed before update in production.
