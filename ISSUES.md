# Inside Success — Cross-Repo Issues Log

Org-wide findings that span multiple repositories, or that don't belong to
any single project's own issue tracker. Individual repos keep their own
`ISSUES.md` (see `Team-Brains`, `brians-2nd-brain-integration-work`,
`llm_client`, `graph-ontology-extraction`, `graph-retrieval`, `project-meta`,
and others) for project-scoped findings — this file is only for things that
cut across repos or belong to the org as a whole.

Add a new entry any time a cross-repo gap is discovered. Do not delete
resolved entries — mark them `[RESOLVED]`.

---

## ISSUE-001 — 9 repos have recurring CI/pipeline failures, unaddressed since at least 2026-08-25

**Discovered:** 2026-09-03, reviewing ~9 days of Brian's automated "Devz"
dev-agent log-alert DMs (an hourly LLM log-summarizer). The same handful of
failures get reworded and re-reported every hour with no one having fixed
the underlying issue.

**Status:** Open

**The gap — deduplicated by repo/root-cause:**

| Repo | Issue |
|---|---|
| `2nd-brain-plan-repo` | Conformance/custody check fails — dashboard's pinned `source_revision.commit_sha` isn't an ancestor of what's actually in the validated git commit. The single most-repeated issue in the whole review window. |
| `ISTV-Lead-Scrapper` | **Repository not found** — CI can't resolve the repo at all, not a code failure. Likely renamed/deleted/access issue. |
| `second-brain-mega` | Refresh workflow fails — missing required `token` input. |
| `inside-success-mega` | Recurring failures in the refresh workflow — same shape as `second-brain-mega`'s, may share a root cause. |
| `ai-marketing-tool-backend` | Recurring Postgres unique-constraint/duplicate-key violations on `content_transitions` and `users` tables in CI tests; one flaky test ("app sessions not returning to baseline"). |
| `Sales-Recruiting-Data-Agent` | Backup/snapshot job fails, exit code 1 — flagged by the alert itself as a potential data-loss risk. |
| `grounded-research` | Pipeline `validate` job fails repeatedly. |
| `skills-library` | Validation job fails, no clear error in truncated logs. |
| `Mock-Call-Agent` | Persona-fingerprint contract test failures; separate CI test-job failures. |

**Investigated separately — a security flag, checked directly, not just relayed:**
`Mock-Call-Agent` was flagged once (2026-09-01) for "potential proxy secret
exposed in logs," never repeated in the following ~9 days of alerts. Checked
`.github/workflows/ci.yml` and `deploy.yml` (only legitimate stdin/file
patterns, nothing printed to logs) and `frontend/app/api/backend/[...path]/route.ts`
where `USER_PROXY_SECRET` is actually used (error logging there only logs
messages/status, never the secret value, guarded by an explicit missing-secret
check). No live leak found — most likely a one-time false read by the
summarizer, not a real exposure. Worth a second look from whoever owns
`Mock-Call-Agent` for certainty, not urgent.

**Required repair:** None of this needs a reply to the alert bot — it needs
someone to actually fix each underlying issue once. `ISTV-Lead-Scrapper`
(repo not found) and the two `-mega` repos (missing token input) look like
the fastest fixes — config/access problems, not debugging. `2nd-brain-plan-repo`'s
custody drift is the most-repeated and likely highest-value fix given
frequency.

**Source:** Brian's weekly-plans repo has the full per-message review:
[`work-results/11-devz-log-review-20260903.md`](https://github.com/BrianMills2718/weekly-plans/blob/main/work-results/11-devz-log-review-20260903.md)
(personal repo, not Inside Success org-owned — linked for detail, not as
the source of record for this entry).
