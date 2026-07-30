# n8n-support-ops

Nine n8n workflows that run a support engineering operation: intake, SLAs, queue health, staffing, quality, metrics, and an audit trail. Built in n8n Cloud, exported here as importable JSON.

These are working designs with mock data at the source nodes. Every mock node is labeled with the production swap (Plain, Zendesk, Jira, Metabase, BambooHR), so each workflow goes live by replacing one node and connecting credentials. The patterns come from operations I ran in production for years: an SLA program with proactive alerting that took compliance from 75% to 95%+, ticket-category deflection, executive escalation ownership, and org-wide AI tooling deployment.

## The workflows

| Workflow | Operations job | What it does |
|---|---|---|
| [SLA Sentinel](workflows/sla-sentinel.json) | KPIs and alerting | Checks the queue every 15 minutes, computes % of SLA consumed per ticket, alerts at 75%, escalates at 90%. Quiet when the queue is healthy. |
| [AI Ticket Triage Agent](workflows/ai-ticket-triage-agent.json) | Intake | LLM triage of inbound tickets: severity, category, business impact, and a suggested first response. Critical issues escalate to a human with customer, ARR, impact, and data up front. |
| [Stuck-Ticket Detective](workflows/stuck-ticket-detective.json) | Queue health | Daily sweep for tickets idle 48h+ or stuck in reply ping-pong, grouped by why they are stuck, each with a concrete next action. Deliberately no AI: deterministic rules are the right tool here. |
| [Monday Ops Brief](workflows/monday-ops-brief.json) | Team leadership | Pulls weekly metrics Monday 8am, and an LLM writes the ops narrative: deltas, wins, concerns, talking points for the team meeting. |
| [Coverage + Burnout Radar](workflows/coverage-burnout-radar.json) | Staffing | Plans next week every Friday: coverage gaps across two regional support windows given PTO, plus a workload-balance check that flags anyone carrying over 30% of volume. |
| [AI Agent QA Auditor](workflows/ai-agent-qa-auditor.json) | Quality | Daily skeptical audit of a sample of tickets an AI support agent resolved, producing a false-resolution rate. Deflection only counts if the answers were right. |
| [Team Metrics To Slack](workflows/team-metrics-to-slack.json) | Metrics and morale | Weekly leaderboard to a Slack channel: top performer, most improved, fastest first response. Coaching flags go to the lead privately. Praise in public, coach in private. |
| [Audit Trail Ledger](workflows/audit-trail-ledger.json) | Evidence | A reusable sub-workflow any other workflow calls to log who/what/when/source to one ledger, plus a weekly evidence digest. Audit season becomes a filter on one sheet. |
| [JML Lifecycle (Okta + Jamf)](workflows/jml-lifecycle-okta-jamf.json) | IT lifecycle | HR events drive identity and device actions: joiners get an Okta account, birthright groups, and their laptop assigned in Jamf; leavers get sessions revoked, account deactivated, and the laptop locked by MDM command with an IT-only PIN. Mover branch left as an exercise. |

## The team lead pack

The people-and-program half of running a support engineering team:

| Workflow | Lead job | What it does |
|---|---|---|
| [Ramp Program Orchestrator](workflows/ramp-program-orchestrator.json) | Onboarding engineers | A new-hire event dispatches the full 3-week ramp: shadow week, supervised week with 100% QA sampling, solo week, and graduation criteria measured from queue data instead of vibes. Hire, mentor, and lead each get their packet; mentors get a 30% queue reduction on paper, because mentoring that is not resourced is theater. |
| [Playbook Gap Miner](workflows/playbook-gap-miner.json) | Documentation | Weekly: finds ticket categories recurring 5+ times with no playbook, drafts the stub (symptoms, diagnostics, resolution, escalate-when), and queues it for human review. The every-escalation-becomes-documentation rule, enforced by a robot. |
| [Ticket QA Sampler](workflows/ticket-qa-sampler.json) | Quality of human replies | Weekly stratified sample per engineer, scored on a 4-part rubric (empathy, accuracy, completeness, next step) with one coaching sentence each. Lead-only output that feeds 1:1s, never the channel. |
| [CSAT Detractor Loop](workflows/csat-detractor-loop.json) | Closing the loop | Every low score gets a theme and a from-the-lead follow-up draft that names the failure plainly and says what changes. Drafts await human approval; apologies are never automated. |
| [Linear Cycle Hygiene](workflows/linear-cycle-hygiene.json) | Engineering escalations | Midweek sweep: stale escalations sorted by ARR at stake, issues missing from any cycle, and shipped fixes whose customers have not been told yet. No AI; determinism wins here. |
| [Volume Spike Early-Warning](workflows/volume-spike-early-warning.json) | Queue spikes | Hourly intake vs same-weekday baseline; on a spike, checks category concentration to distinguish one incident wearing twenty tickets from organic volume, and prescribes the right response for each. |

## The IT operations pack

Three more workflows from the corporate IT side of the house, same honest mock-data framing:

| Workflow | IT ops job | What it does |
|---|---|---|
| [Cert + Credential Expiry Sentinel](workflows/cert-credential-expiry-sentinel.json) | Outage prevention | Daily sweep of a credential registry (SAML certs, API tokens, TLS, integration secrets) with 30/14/7-day alert tiers. Born from a real production outage caused by an expired credential; the failure class never recurred once pre-expiry alerting existed. |
| [SaaS License Reclaim Report](workflows/saas-license-reclaim.json) | Cost control | Monthly cross of seat assignments against last-login data; idle seats become a dollars-per-year reclaim report your CFO will actually read, and ammunition for renewal negotiations. |
| [License Confirm-or-Release](workflows/license-confirm-or-release.json) | Cost control, human-in-the-loop | Finds idle seats across the stack and asks each user directly with approve/release buttons in the email (send-and-wait). Keep, release, and no-reply tallies go to IT; releases are reversible same-day, and nothing is revoked silently. |
| [Annual Entitlement Recertification](workflows/annual-entitlement-recertification.json) | Least privilege, yearly | Every manager gets an attestation packet for their reports (every app, every permission level, stale and privileged grants flagged), while usage data drives role-bundle trim proposals: if under 20% of a role uses an app, it stops being birthright. |
| [Quarterly Access Review Orchestrator](workflows/quarterly-access-review.json) | Least privilege | Sends each system's data owner their review packet (privileged roles and exception grants first), calls out exceptions with no expiry, and logs every dispatch to the shared Audit Trail Ledger sub-workflow. Workflows composing workflows. |

## Importing

Open a new workflow in n8n, copy a JSON file's contents, and paste directly onto the canvas (or use Import from File). Then connect credentials: Gmail for alerts, OpenAI (or any chat model) for the three AI workflows, an Okta API token and Jamf client for the JML workflow, Google Sheets for the ledger, and a Slack incoming webhook for the leaderboard.

## Design principles

- Alert before breach, not after. Every monitor fires on trajectory, not failure.
- Escalations lead with customer, ARR, business impact, and data.
- AI where judgment scales (triage, narratives, audits), plain rules where determinism wins (SLA math, coverage math).
- Every automated action leaves an audit trail.
- A customer having to chase you is a failure.

## Related projects

[okta-as-code](https://github.com/emiliorobles24/okta-as-code) (identity state as Terraform) · [endpoints-as-code](https://github.com/emiliorobles24/endpoints-as-code) (fleet policy as Terraform with CI) · [mdm-from-scratch](https://github.com/emiliorobles24/mdm-from-scratch) (device management architecture) · [iam-from-scratch](https://github.com/emiliorobles24/iam-from-scratch) (identity architecture) · [jira-from-scratch](https://github.com/emiliorobles24/jira-from-scratch) (company-wide service management design) · [mdm-compliance-dashboard](https://github.com/emiliorobles24/mdm-compliance-dashboard) (three MDMs reconciled into one source of truth)

Built with AI coding agents in the loop and every workflow reviewed, tested, and owned by me.
