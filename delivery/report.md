# postmortem-maker — bounty delivery report

Bounty: Frantic #83 — runx skill: postmortem maker ($9). A published, installable runx skill that turns incident
fragments (timeline events, alerts, deploy events, chat notes, and a postmortem policy) into a traceable
postmortem where every timeline entry and root-cause claim cites input evidence, unresolved facts go to
unknowns, and a gated publish proposal is emitted only when the evidence is consistent — never posting or
assigning anything.

## Evidence

- **Package**: `epistemedeus/postmortem-maker@sha-1d497f39e48b` — published to the runx registry via `runx login --provider github --for publish; runx registry publish ./skills/postmortem-maker/SKILL.md --registry https://api.runx.ai`.
- **runx CLI**: `runx-cli 0.6.14` for every publish/install/dogfood/verify step.
- **Public URL**: https://runx.ai/x/epistemedeus/postmortem-maker@sha-1d497f39e48b
- **Source**: https://github.com/epistemedeus/postmortem-maker/tree/9e7094355f6a43a347b5bee780b45271ca3a8358
- **PR**: https://github.com/runxhq/runx/pull/258 (head `0da32427b41d0669664c606feb3fbd8f3adbbf99`) adds `skills/postmortem-maker/{X.yaml,SKILL.md,run.mjs,fixtures,harness}`; raw [X.yaml](https://raw.githubusercontent.com/epistemedeus/runx/0da32427b41d0669664c606feb3fbd8f3adbbf99/skills/postmortem-maker/X.yaml) and [SKILL.md](https://raw.githubusercontent.com/epistemedeus/runx/0da32427b41d0669664c606feb3fbd8f3adbbf99/skills/postmortem-maker/SKILL.md) are fetchable from the head commit.
- **Install**: `runx add epistemedeus/postmortem-maker@sha-1d497f39e48b --registry https://api.runx.ai` — clean install verified: status=installed, digest `sha256:12c1939b81f0e3f6dc68cea5b372f3efe425c8d78820c30f808408243ec922fe` matches the registry digest.
- **Registry read**: `runx registry read epistemedeus/postmortem-maker@sha-1d497f39e48b --registry https://api.runx.ai --json` resolves the published metadata and digests (publisher epistemedeus, trust tier community, profile_digest `7a4d2967a6541ec55ee891fca251ebfa7c9383b77b58b86366053095ca2ba8e1`).
- **Local harness**: passed (3/3, 0 assertion errors, with the X.yaml expect.output packet assertions) — cases `consistent-evidence-yields-proposal` (sealed), `ambiguous-evidence-withholds-proposal` (sealed), `refuses-without-policy-or-evidence` (failure); the consistent case emits a proposal, the ambiguous case seals with unknowns and withholds it, the no-policy/no-evidence case is the governed stop.
- **Hosted harness**: green — the registry publish gate ran the hosted harness including the refuses-without-policy-or-evidence stop case.
- **Dogfood**: `runx skill epistemedeus/postmortem-maker@sha-1d497f39e48b --registry https://api.runx.ai --input-json incident_timeline='...' --input-json alerts='...' --input-json deploy_events='...' --input-json chat_notes='...' --input-json postmortem_policy='...' --receipt-dir <dir> -j` sealed receipt `runx:receipt:sha256:dcf429329ec50227edbe7c9a4285b5c2a92b6e9f5504ee7ba782090cbdf4c60c`; `runx verify --receipt receipt.json --json` → **valid**.
- **Postmortem decision**: the dogfood input yields timeline_count=7 (every entry evidence-cited), impact window 2026-03-14T09:10:00Z → 09:55:00Z (45 minutes, 1 critical alert fired and resolved), root_cause **determined** (deploy of checkout-api 2026.03.14.1, citing deploy_events[0], alerts[0], incident_timeline[1], chat_notes[0]), unknowns=0, action_items=2, postmortem status ready_for_review, proposal status **proposed**.
- **Gated proposal**: `publish_proposal.performed_by = doc-publisher` (send-as for comms), `requires_approval = true`; postmortem-maker posts nothing and assigns no live tasks.
- **Verify it yourself**: install, run the dogfood command, then `runx verify --receipt receipt.json --json` with the public key in [verification.json](./verification.json).

## How a new user adopts it

1. `runx add epistemedeus/postmortem-maker@sha-1d497f39e48b --registry https://api.runx.ai`
2. `runx skill epistemedeus/postmortem-maker@sha-1d497f39e48b --registry https://api.runx.ai --input-json incident_timeline='...' --input-json alerts='...' --input-json deploy_events='...' --input-json chat_notes='...' --input-json postmortem_policy='...' --receipt-dir <dir> -j`
3. `runx verify --receipt receipt.json --json` (public key in verification.json) → valid.
