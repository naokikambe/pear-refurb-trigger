# pear-refurb-trigger

External cron trigger stub for checking whether an HTTP-based cron service can trigger GitHub Actions at a stable interval.

This repository is intentionally minimal. It records that an external service called GitHub's `repository_dispatch` endpoint by creating a GitHub Actions run and logging basic timing metadata.

## Scope

This first version only:

- accepts `repository_dispatch` events with type `external_tick`
- supports manual `workflow_dispatch` runs for testing
- logs UTC and JST timestamps
- logs GitHub run metadata
- logs optional `client_payload.source` and `client_payload.sent_at`

This version does not integrate with the watcher, notifier, state storage, repository dispatch to another repository, or mail delivery.

## Workflow

Workflow name:

```text
External Trigger Stub
```

Supported events:

```yaml
repository_dispatch:
  types:
    - external_tick
workflow_dispatch:
```

Run history is checked in the GitHub Actions UI. The timestamps in each run can be used to compare the external cron service's intended interval with observed GitHub Actions execution times.

## Manual Test

The workflow can be started manually from the GitHub Actions UI or with GitHub CLI:

```bash
gh workflow run "External Trigger Stub" --repo <OWNER>/pear-refurb-trigger
```

## repository_dispatch

External cron services can call GitHub's repository dispatch endpoint. Use placeholders for credentials and runtime values:

```bash
curl -X POST \
  -H "Authorization: Bearer <TOKEN>" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/<OWNER>/pear-refurb-trigger/dispatches \
  -d '{"event_type":"external_tick","client_payload":{"source":"external-cron","sent_at":"<ISO8601>"}}'
```

The token must not be committed, logged, or shared. Do not log Authorization headers.

## Security

This is a public repository. Do not put tokens, Authorization header values, private URLs, or operational secrets in files, logs, workflow output, issues, or documentation.

This repository does not require GitHub Secrets in its initial version.

## Out of Scope

The following are intentionally outside this repository's initial scope:

- external cron service configuration
- GitHub PAT creation
- disabling or changing the existing watcher schedule
- triggering the existing watcher workflow
- changing existing watcher or notifier repositories
