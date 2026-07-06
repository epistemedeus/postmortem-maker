# postmortem-maker

A [runx](https://runx.ai) skill that turns incident fragments into a
**traceable postmortem** — every timeline entry and root-cause claim cites
input evidence, unresolved facts go to `unknowns[]` instead of being asserted,
and a **gated publish proposal** is emitted only when the evidence is
consistent. It never posts, publishes, or assigns anything itself.

- **Offline, no side effects beyond its own artifacts.** No network, no comms,
  no task assignment, no code execution; the only writes are the
  `evidence.json`/`report.md` artifacts inside the skill directory
  (`output_dir`, default `artifacts/`, path-confined).
- **Grounded in evidence refs.** Every record gets a stable ref
  (`incident_timeline[2]`, `alerts[0]`, …); every claim cites the refs it came
  from. Untimed, contradictory, or missing facts become `unknowns`, not facts.
- **Cautious root cause.** A cause is asserted only when exactly one candidate
  deploy preceding the incident start is corroborated by a timeline event or
  chat note and disputed by none; multiple corroborated candidates, any
  dispute, or no corroboration ⇒ `undetermined`.
- **Gated proposal.** A `publish_proposal` (performed by the downstream
  `doc-publisher` executor, or `send-as` for comms) is emitted only when the
  policy allows it and `unknowns` is empty — fail closed.
- **Policy-aware.** `postmortem_policy` governs publish gating, required
  sections, and action-item limits; missing policy is a governed refusal.

## Layout

```
skills/postmortem-maker/
  SKILL.md      # skill card and full documentation
  X.yaml        # execution profile, policy, and typed inputs/outputs
  run.mjs       # dependency-free Node analyzer
  fixtures/
    consistent-evidence-yields-proposal.yaml   # consistent -> proposal, ready_for_review
    ambiguous-evidence-withholds-proposal.yaml # ambiguous  -> unknowns, no proposal
    refuses-without-policy-or-evidence.yaml    # no policy/evidence -> governed stop (failure)
  harness/
    harness-output.json                        # local harness evidence: passed, 3/3
    receipts/                                  # the sealed receipts from that run
```

## Install and run

```bash
runx add epistemedeus/postmortem-maker@0.1.0
runx skill epistemedeus/postmortem-maker@0.1.0 \
  --input-json incident_timeline='[...]' \
  --input-json alerts='[...]' \
  --input-json deploy_events='[...]' \
  --input-json chat_notes='[...]' \
  --input-json postmortem_policy='{"allow_publish":true}' \
  --json
```

## Local harness

```bash
runx harness ./skills/postmortem-maker
```

Three cases: `consistent-evidence-yields-proposal` (seals, proposal),
`ambiguous-evidence-withholds-proposal` (seals, unknowns populated, no
proposal), and `refuses-without-policy-or-evidence` (the governed stop case).

The authoritative harness cases are declared inline in `X.yaml` under
`harness.cases` (what `runx harness <skill-dir>` runs); the files in `fixtures/`
mirror those same cases. Keep them in sync when editing.

## License

MIT — see [LICENSE](LICENSE).
