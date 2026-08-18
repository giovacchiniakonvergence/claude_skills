# Trigger (Bridge) Job Constraints

A trigger (bridge) job starts a downstream pipeline and never reaches a runner. It accepts a closed
set of keywords, listed under `### trigger` in [ci/yaml/_index.md](./ci/yaml/_index.md):
`allow_failure`, `extends`, `needs` (not `needs:project`), `only`/`except`, `parallel`, `rules`,
`stage`, `trigger`, `variables`, `when` (`on_success`, `on_failure`, `always`, `manual`),
`resource_group`, `environment`.

Runner-oriented constraints do not apply to these jobs. Their absence is not a gap:

- `timeout` is rejected: `jobs:<name> config contains unknown keys: timeout`. The bridge ends when
  the downstream pipeline ends, which carries its own timeouts. Do not report the missing timeout
  as a gap
- `script`, `image`, `tags`, `cache`, `artifacts` do not apply. Their absence is not a finding
- `inherit: default: false` only does something when a `default:` block is actually in scope.
  Verify before crediting or requiring it

Behavioral contract of the bridge itself:

- `strategy: depend` makes the bridge adopt the downstream result; without it the job succeeds as
  soon as the downstream pipeline is created
- `trigger:forward:yaml_variables` defaults to true, `pipeline_variables` to false. Forwarding
  pipeline variables grants the upstream project a lever on downstream behavior: flag it as a
  decision, do not add it silently
- Upstream variables take precedence over same-name variables in the downstream project

SOURCE: `### trigger` section of [ci/yaml/_index.md](./ci/yaml/_index.md) and
[ci/pipelines/downstream_pipelines.md](./ci/pipelines/downstream_pipelines.md), as vendored under
`references/` by `scripts/sync_gitlab_docs.py`.
