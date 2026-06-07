# pear-refurb-trigger

External cron trigger layer for starting the watcher workflow from an external HTTP cron service, with a disabled GitHub native schedule template kept for future use.

This repository receives a trigger event, logs timing metadata, and starts the watcher repository through GitHub Actions `workflow_dispatch`.

## Scope

This version:

- accepts `repository_dispatch` events with type `external_tick`
- supports manual `workflow_dispatch` runs for testing
- keeps a disabled GitHub native `schedule` template for future use
- logs UTC and JST timestamps
- logs GitHub run metadata
- logs optional `client_payload.source` and `client_payload.sent_at`
- dispatches the watcher workflow

This repository does not access watcher targets, state storage, notifier dispatch, or mail delivery.

## Workflow

Workflow name:

```text
External Trigger Stub
```

Enabled events:

```yaml
repository_dispatch:
  types:
    - external_tick
workflow_dispatch:
```

Native GitHub schedule support is kept in the workflow as a commented template and is currently disabled:

```yaml
schedule:
  - cron: "7,17,27,37,47,57 * * * *"
```

Run history is checked in the GitHub Actions UI. The timestamps in each run can be used to compare trigger timing with observed GitHub Actions execution times.

The workflow maps event sources to `trigger_source`:

```text
repository_dispatch -> external-dispatch
workflow_dispatch -> manual
```

If native schedule is re-enabled later, schedule events map to `github-schedule`.

The trigger then calls the watcher workflow dispatch API with that `trigger_source` input.

## Configuration

Configure the following GitHub Secret:

```text
WATCHER_DISPATCH_TOKEN
```

Configure the following GitHub Variables:

```text
WATCHER_REPO
WATCHER_WORKFLOW
WATCHER_REF
```

Recommended values:

```text
WATCHER_REPO=<OWNER>/<WATCHER_REPOSITORY>
WATCHER_WORKFLOW=<WATCHER_WORKFLOW_FILE>
WATCHER_REF=main
```

`WATCHER_DISPATCH_TOKEN` must be able to dispatch the configured watcher workflow. Do not log the token or Authorization header.

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

The workflow intentionally logs only timing metadata, GitHub run metadata, trigger source, watcher repository/workflow identifiers, and watcher dispatch HTTP status.

## Out of Scope

The following are intentionally outside this repository's initial scope:

- external cron service configuration
- GitHub PAT creation
- disabling or changing the existing watcher schedule
- changing existing watcher or notifier repositories
