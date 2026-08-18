# Documentation Authority Map

A bulleted list inside a topic page is a summary and may be incomplete. The reference table, or the
keyword's own section, is the contract. When they disagree, the table wins.

## Resolution Rules

These override the model's recollection and any summary list:

- The model must settle a documentation question against the authoritative page listed in the map
  below, never against a bulleted list found in a topic page
- The model must establish which keywords a job type accepts before requiring or recommending one
- The model must treat a lint or dry-run result that contradicts its reading of the documentation as
  evidence that the reading is wrong: re-check the authoritative page before reporting a finding. A
  passing verification is not "works but outside the contract"

## Authority Map

| Question | Authority | Not this |
|---|---|---|
| Is this variable usable in `include:` / `include:rules`? | `references/ci/variables/predefined_variables.md`, `Availability` column: `Pre-pipeline` variables are the only ones admitted | The list under "Use variables with `include`" in `ci/yaml/includes.md`. It omits `Pre-pipeline` variables that do work, `CI_COMMIT_TAG` among them |
| When does this variable exist at all? | "Variable availability" in `predefined_variables.md`: `Pre-pipeline` (before pipeline creation, usable in `include:rules`), `Pipeline` (usable in job `rules`), `Job-only` (script only, unusable in `workflow`, `include`, `rules`, trigger jobs) | Recollection |
| Which keywords does this job type accept? | The keyword's own section in `ci/yaml/_index.md`; `### trigger` carries a closed list | A generic pipeline checklist |
| Which variables win, upstream or downstream? | `ci/pipelines/downstream_pipelines.md`: upstream takes precedence over same-name downstream variables | Recollection |
| Can a pipeline be auto-cancelled? | `ci/pipelines/settings.md` plus `workflow:auto_cancel:on_new_commit` in `ci/yaml/_index.md` (`conservative` default, `interruptible`, `none`). Multi-project downstream pipelines are not auto-cancelled from upstream | Recollection |
| Which regex engine do `rules:if` expressions use? | `ci/yaml/_index.md`: RE2, unanchored partial match, `=~` and `!~`. `rules:changes:regexp` and `rules:exists:regexp` use Ruby's engine instead | Recollection |

The model must open the authoritative file. The Documentation Index generated at the end of SKILL.md
lists what exists, not what decides.

## Empirical Arm — Job Simulation via the CI Lint API

`glab ci lint` reports only valid/invalid, and `--include-jobs` may print nothing. To see which jobs
a ref would actually produce, call the CI Lint API directly. This is the strongest available check on
`rules` and `include:rules`, because GitLab resolves the real templates and evaluates the real
predefined variables for that ref:

```bash
PROJ="group%2Fsubgroup%2Fproject"   # path URL-encoded
python3 -c "
import json
json.dump({'content':open('.gitlab-ci.yml').read(),
           'dry_run':True,'include_jobs':True,'ref':'REF'}, open('/tmp/lint.json','w'))"

# The explicit header is mandatory: without it, glab returns HTTP 415
glab api --method POST --header "Content-Type: application/json" \
  "projects/$PROJ/ci/lint" --input /tmp/lint.json |
  python3 -c "
import json,sys
d=json.load(sys.stdin)
print(d['valid'], d['errors'], d['warnings'])
print([j['name'] for j in (d.get('jobs') or [])])"
```

- The model must run this once per ref class the configuration distinguishes (branch, default
  branch, tag, and a merge request if `workflow` admits one), and compare the resulting job lists
  against the intent stated in the comments
- The response also carries `merged_yaml`: grep it to prove whether a conditional `include:`
  resolved for that ref
- To learn whether a keyword is admitted in a context, add it, lint, and read the error. This
  settles a keyword question faster and more reliably than prose

SOURCE: [CI Lint API](https://docs.gitlab.com/api/lint/) (accessed 2026-08-18) and the vendored
pages named in the map above.
