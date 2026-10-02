# AGENTS.md

Last updated: 2026-10-01

Rules for AI coding agents working in this repo. Read all of it before doing anything. These rules override your defaults.

This repo is public. Read section 6 before writing any file.

---

## 1. What this repo is for

The question: **what do Pokémon battle agents know internally, and do they use it?**

We study existing trained agents (Metamon, UT Austin) instead of training our own. Each turn the agent sees raw facts (names, types, base stats, base power), never computed results like type effectiveness or damage. So anything it "knows" beyond that, it worked out itself.

First targets:

1. **Type chart.** Does the agent compute type effectiveness internally, how closely does it match the real 18×18 chart, and does editing it change move choice?
2. **Opponent-team memory.** Does the agent remember opponent Pokémon it has seen but that are no longer on the field?

The previous attempt (cross-lingual political probing) failed because its labels were opinions. Read `docs/lessons-from-rile.md` first.

Priorities, in order:

1. **The owner understands the work** well enough to discuss it with working researchers.
2. **Every claim is backed by evidence a sceptic would accept.** See section 2.
3. Outreach to researchers with specific questions.
4. A LessWrong post or preprint once a claim is conclusive. A clear negative result counts.

## 2. Evidence first (the main rule)

No large runs and no claims until each step passes, with an owner check before the next step starts.

1. **Behaviour test.** Before looking inside, check that the agent acts on the concept (for example, it picks super-effective moves far above chance). If it doesn't, there is nothing to probe.
2. **Input check.** Confirm from the code that the concept is not given to the agent directly. If it is, a probe proves nothing.
3. **Labels from the simulator.** Every label comes from exact game rules or the battle simulator's full state, never from human judgement or guessed data. Human replays only partly reveal teams; don't use them for hidden-information labels.
4. **Small probe with baselines.** On a few hundred examples, compare the trained agent against: an untrained copy of the same model, a probe on the raw inputs, and the same probe trained on shuffled labels (control task). If any baseline matches the trained agent, stop.
5. **Confounds.** Check the label is not predictable from something simpler (move identity, species, turn number). Use matched comparisons.
6. **Causal test before any claim.** Edit or remove the internal signal and check behaviour changes as predicted. Without this, report information as "present", never as "used".
7. **Replicate** on at least one other checkpoint or seed.

Pilot every step on a tiny batch first. Before each experiment, write down what result would count as yes, no, or inconclusive.

## 3. How to work with the owner

### Teach, don't ship, and chunk the code

- Never drop a large end-to-end pipeline. One concept, one small chunk (tens of lines), the owner runs it, discuss what they saw, then the next chunk.
- Boilerplate (imports, argparse, file I/O) can be written quickly. Conceptual code (label generation, probes, baselines, metrics) is chunked and explained before it runs.
- Prefer questions on conceptual pieces: "What do you expect a probe to find on an untrained copy of the model?"

### Approval

- Do not create, modify, move, or delete code, data, notes, or configuration without the owner's explicit approval of that change.
- Do not create new Markdown files unprompted. Suggest the idea in chat.
- Never start anything that costs money (API calls, GPU or VM time) without stating what it does, how much, and who pays, then waiting for a yes.

### Language

- Plain, everyday words. Define any technical term the first time it is used in a given explanation. Assume no prior knowledge of the term.
- Short, bite-sized explanations. One idea, then stop.
- No LaTeX. Use plain text, Unicode symbols, or inline code for math.
- Say what was measured before naming a technique. Separate what the data shows from interpretation.

### Commits

- Never add AI credit to commits or PRs: no `Co-Authored-By` trailers, no "Generated with" lines.
- The owner stages and commits. Agents hand over the commands.

## 4. Documentation

Keep it lean. The last repo's results document grew to 1,273 lines with hash tables, and that did not help anyone understand anything.

- Every Markdown file has `Last updated: YYYY-MM-DD` under its title.
- One document per topic. Date each section.
- Every run record states: date, settings, seeds, code version (commit), output location, status.
- Report numbers neutrally: what was measured, on what data, how many items. Then interpretation, clearly labelled as such.
- Mark anything unfinished as incomplete. Never present a partial or aborted run as a result.
- Keep a dated entry in `docs/bugs-squashed.md` for each data, evaluation, or environment issue: what happened, impact, fix, how it was checked. Open issues are labelled open.

### Before any substantial run

Share a short pre-run card and get the owner's go-ahead:

- the metric and every saved field;
- the dataset, exact split, controls, and models;
- what the run can and cannot show;
- expected runtime and cost, and known risks.

For long detached runs, do not poll unless asked.

## 5. Code and tooling

- Python with `uv` only: `uv venv`, `uv pip install`, `uv run python ...`. Never bare `pip` or `python`.
- Small, readable, directly runnable scripts. No frameworks, classes, or `utils/` until something actually needs reuse.
- Smoke-test any measurement on a small batch before a real run.
- Keep local repo disk usage under 2 GB. Large activations, checkpoints, and datasets live on the VM or cloud storage.
- Guard any `CUDA_VISIBLE_DEVICES` override in scripts.

## 6. Public repo: never commit these

- Raw or derived text from sources whose licence forbids redistribution (including anything from the Manifesto Project).
- Cloud identifiers: GCP project, VM, zone, disk, bucket, or service-account names; IP addresses; private URLs.
- Credentials: API keys, tokens, passwords, SSH keys, `.env` contents.
- Personal data: names of private individuals, emails, phone numbers.

Public figures' public statements are fine to store with their source. Redistribution terms of each source must be checked before its text is committed; until checked, keep it under the ignored `data/` folder.

Metamon code, weights, and datasets each have their own licence. Check the licence of each before committing any battle data, logs, or derived activations; until checked, keep them under `data/`.

## 7. Google Cloud

The GCP operating pack lives **outside this repo** at `~/Desktop/gcp-workspace/`, shared with the old repo. It is never copied into this repo.

Before any cloud, VM, GPU, storage, backup, startup, shutdown, or cleanup operation:

1. Read `~/Desktop/gcp-workspace/SKILL.md` in full. It is mandatory and governs discovery, reuse, approval, backups, migration, startup, shutdown, cleanup, security, and cost tracking.
2. Read `~/Desktop/gcp-workspace/infrastructure_manifest.json`.
3. Read `~/Desktop/gcp-workspace/workspace_state.md`.
4. Read the most recent entries in `~/Desktop/gcp-workspace/workspace_log.jsonl`.
5. Read open or recent rows in `~/Desktop/gcp-workspace/workspace_sessions.csv` and `~/Desktop/gcp-workspace/workspace_cost_ledger.csv`.
6. Compare recorded state with live GCP before changing anything.

Rules: do not create a VM before checking existing resources. No more than one GPU VM without explicit approval. Never delete or overwrite cloud resources without explicit approval. Do not repeat timed-out lifecycle commands blindly. Never leave a GPU VM running after work is done unless the owner says so.

After every verified operation: update `workspace_state.md`, append one JSON record to `workspace_log.jsonl`, update the session and cost ledgers, and report running resources and ongoing costs.

**Who does GCP work:**

- Read-only checks (VM status, disk usage, `nvidia-smi`, logs, cost review) go to `escalation-worker` (Sonnet).
- Actions (start or stop VMs, copying files, launching jobs, updating the ledgers) go to `builder` (Sonnet). One at a time.
- The root session plans the run, writes the pre-run card, and gets the owner's approval. Every handoff includes this section, `SKILL.md`, the exact commands, the owner's approval for any start/stop/create/delete, and completion criteria.
- The Sonnet agent reports back running resources and cost, what changed, and the ledger entries it wrote.

## 8. Layout

```
.
├── AGENTS.md
├── docs/            # lessons, experiment notes, bugs-squashed, writeups
├── data/            # git-ignored: battles, activations, labels until licences are checked
└── src/             # small scripts
```

## 9. TL;DR

1. Evidence first: behaviour test, input check, baselines, causal test, then a claim.
2. Labels come from the simulator or exact game rules, never from guesses.
3. Teach in small chunks. Ask before changing anything.
4. Public repo: no cloud IDs, credentials, or unlicensed data.
5. Read `~/Desktop/gcp-workspace/SKILL.md` before any cloud action.
