# Release Radar: a stack-agnostic playbook

Rob Browning · first written November 2026

A one-page, read-only dashboard that answers four questions about every open release: **what's due, what needs attention, where to test it, and how far along it is.** I first built it for a web team with Jira, a Claude Dashboard and a Claude scheduled task. This version strips out the specific stack so the pattern can be rebuilt on any tracker, environment ladder or hosting option.

## 1. The problem it solves

Teams running several releases in parallel spread the truth across the tracker's releases page, the sprint board and individual tickets. Nobody has one view of date, risk, environment and progress together. The radar puts all four on one page, refreshed automatically, without anyone touching a ticket.

**Design principles**

- **Read-only.** The radar and its refresh never comment on, edit or move a ticket. Anything meant for a ticket comes back as text to paste.
- **Show the evidence.** Any inferred value (for example the environment) is labelled as inferred and cites the tickets it came from.
- **Human override wins.** A value set by a person is never overwritten by the automated refresh.
- **Fresh where it's cheap, cached where it's expensive.** Release names and dates are read live; counts and inferences come from a scheduled refresh.
- **Stale data is flagged, not hidden.** Each source shows its last-updated time.

## 2. What the page shows

| Part | Content |
| --- | --- |
| Summary line | The next release not yet past its date, its countdown and environment; counts of releases due within a week, overdue, and live but still open. |
| Timeline | One dot per release at its release date, labelled name · date · environment, with a "today" line. |
| Release cards | Name (linked to the tracker's release report), countdown pill, release date, environment and its source, progress bar with counts, release-prep ticket key, status and target date, and a one or two sentence evidence note. |
| Sources panel | Last refresh time per data source; stale warning after a threshold (default 72h). |

### Countdown flags

| Flag | Rule |
| --- | --- |
| Red "Overdue N days" | Release date passed, version still unreleased. |
| Red "Today / Tomorrow / N days" | 0 to 3 days to go. |
| Amber | 4 to 7 days to go. |
| Green "Out · mark released" | Evidence says it's live but the tracker still lists it unreleased. |
| Grey | More than a week away. |

Thresholds are configuration, not code.

### Progress buckets

Map your workflow statuses to five buckets that mirror the team board. Example mapping:

| Bucket | Example statuses (adapt to your workflow) |
| --- | --- |
| Done | Anything in the "done" status category (Closed, Done, Sign Off) |
| In QA / review | Ready for QA, QA, Review |
| In dev / code review | In Development, Code Review, Ready for Code Review |
| Blocked | Blocked, On Hold |
| Not started | Everything else (New, To Do, In Analysis, Ready for Dev) |

Keep "Not started" as the catch-all so new statuses never go uncounted.

## 3. Data model

Three sources, joined on the release or version id.

1. **Releases (live).** Unreleased versions from the tracker, filtered by a name rule (for example the prefix `Web`) and an exclusion list (for example the backlog version). Read with the viewer's own credentials on page load.
2. **Progress (refreshed file, for example `progress.json`).** Per release: total and bucket counts; the release-prep ticket's key, status and the production date written in its description.
3. **Environments (refreshed file, for example `environments.json`).** Per release: environment, source (`inferred` or `manual`), evidence text, last updated.

```json
// environments.json (one row per release)
{ "versionId": "12345", "env": "UAT", "envSource": "inferred",
  "evidence": "Prep ticket ABC-101 says 'UAT build'; bug ABC-140 raised on UAT 2 days ago.",
  "updated": "2026-11-10T08:31:00Z" }
```

## 4. Environment inference

Most trackers have no field for "which environment is this release on", so the refresh infers it.

**Ladder** (define your own): `Dev < QA < Staging < UAT < Live`. The furthest-along signal wins.

**Signals, in order of strength**

1. Release-prep ticket wording ("Deploy to Staging", "UAT build", "released").
2. Comments on the prep ticket (for example "patch fix was released on 26.10" means Live).
3. The environment field on bugs in the release updated in the last 14 days.

**Fallback rules**

- Mostly not started: Dev.
- No signal: Unknown.
- Any `manual` row is copied forward unchanged.
- Bugs that are *fixes landing in* a release (raised against another environment) are not evidence of where that release is deployed.

**Known failure mode:** prep-ticket descriptions are often copied from earlier releases and keep old dates. Always show both the tracker release date and the prep ticket's target date so the mismatch is visible.

## 5. Refresh job

Runs on a schedule (default weekdays 08:30 local), unattended, no questions asked.

1. Load the current environments file (keep manual rows).
2. List unreleased versions; apply the name filter and exclusions.
3. For each version: count tickets per bucket; find the release-prep ticket (for example component = Release), its status and the production date in its description.
4. Infer the environment (section 4); write one or two sentences of evidence citing ticket keys.
5. Publish new progress and environments files; remove the previous ones.
6. Send a 3 to 5 line summary: due within 3 days, overdue or live but unreleased, environment changes since the last run, releases added or dropped.

**Ops checks:** the scheduler must be allowed to run without manual approval; alert or flag stale data after 72h; offer an on-demand "refresh now" for mid-day updates.

## 6. Swapping the stack

| Layer | Original | Alternatives |
| --- | --- | --- |
| Tracker | Jira (versions, fixVersion, JQL) | Azure DevOps (iterations and releases, WIQL), Linear (cycles and projects, GraphQL), GitHub (milestones, Issues and Projects API), GitLab (milestones, releases API), YouTrack |
| Release grouping | Jira fixVersion | Milestone, iteration path, release label, project or cycle |
| Prep ticket | Component = Release | Label, issue type, or naming convention |
| Environment signal | Ticket text plus the bug Environment field | Deployment API (Argo CD, Octopus, GitHub Deployments, Vercel, Azure Pipelines environments). Use this first if available; text inference becomes the fallback |
| Dashboard | Claude Dashboard artifact | Static HTML on any host, Grafana, Streamlit, Retool, a Confluence or Notion page, an internal web app |
| Refresh | Claude scheduled task | CI cron (GitHub Actions, Jenkins, GitLab CI), cloud scheduler, serverless function |
| Notification | Push notification | Slack or Teams webhook, email |
| Inference engine | Claude reading ticket text | An LLM call in the job, or rules and regex on keywords |

If a real deployment API exists, the environment stops being an inference and becomes a fact. That is the biggest single reliability upgrade.

## 7. Build checklist

- [ ] Tracker project key or workspace, and read-only credentials
- [ ] Release name filter and exclusions
- [ ] Status to bucket mapping agreed with the team board
- [ ] Environment ladder and signal sources defined
- [ ] Release-prep ticket convention identified
- [ ] Countdown thresholds (default red at 3 days or fewer, amber 4 to 7)
- [ ] Dashboard host and sharing model (who can view vs override)
- [ ] Schedule, timezone and notification channel
- [ ] Stale-data threshold
- [ ] Manual override path tested (the override survives a refresh)

## 8. Acceptance criteria

- Every unreleased version matching the filter appears exactly once; excluded versions never appear.
- Bucket counts sum to the release total.
- Countdown colours match the threshold table on boundary days (3, 4, 7, 8, overdue by 1).
- A manual environment survives at least two refreshes unchanged.
- Every inferred environment shows evidence citing at least one ticket, or is marked Unknown.
- A release that is live but unreleased in the tracker shows the green "Out · mark released" flag.
- Data older than the stale threshold is visibly marked.
- The job makes zero write calls to the tracker (verify via the audit log or API scopes).

## 9. Starter prompt (AI-built version)

> Build me a dashboard called "\<Team> Release Radar" from \<tracker>, read-only: never comment on, edit or transition anything.
>
> Source: the unreleased \<versions/milestones> in \<project>, only those whose name matches "\<filter>", excluding \<backlog>. Cross-check the active sprint on \<board link>.
>
> For each release show: the release date (live) with a countdown, red if overdue or 3 days or fewer, amber 4 to 7, green "Out · mark released" if live but still unreleased; which environment it's on (\<ladder>), inferred from the release-prep ticket's description and comments and the environment of recent bugs, with the evidence, overridable by me and kept across refreshes; ticket counts by board column (Done / In QA / In dev / Not started / Blocked) as a progress bar; the release-prep ticket's key, status and production date.
>
> Add a timeline of all releases (name, date, environment on each label) and a one-line summary of what's next and what's overdue.
>
> Then schedule a refresh every weekday at \<time> \<timezone> that updates counts and environments and sends me a short summary.

## 10. Limits and good practice

- Live dates follow each viewer's own tracker access; without it they see only cached counts.
- Counts and environments are as of the last refresh.
- Better tracker hygiene makes a better radar: put the target date in the prep ticket title and keep its "Deploy to ..." line current.
- Viewers can read; only the owner (or a defined role) can override environments.
