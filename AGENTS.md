# AGENTS.md

Last updated: 2026-10-01

Rules for AI coding agents working in this repo. Read all of it before doing anything. These rules override your defaults.

This repo is public. Read section 6 before writing any file.

---

## 1. What this repo is for

The question: **do language models encode which way a political statement leans, in a form that carries across languages?**

The previous attempt (Manifesto Project RILE data, in `~/Desktop/Projects/AI safety`) produced weeks of careful analysis on statements and labels the owner could not read or trust. Read `docs/lessons-from-rile.md` before proposing any dataset or experiment.

Priorities, in order:

1. **The owner understands the work** well enough to discuss it with working researchers.
2. **The data is right.** See section 2.
3. Outreach to researchers with specific questions.
4. A blog post or preprint only if it falls out naturally. Do not optimize for it.

## 2. Data first (the main rule)

Data is the hardest bar in this repo. No probing, model runs, or GPU work until the owner has signed off on the dataset.

Stages, each with a human check before the next starts:

1. **Sources.** Prefer official transcripts (debates, Hansard, Congressional Record, Lok Sabha) over speeches, and speeches over pundit transcripts. Every item keeps its source URL, date, speaker, and exact location in the source.
2. **Extraction.** One item is one claim, verbatim. **An item must make sense read on its own.** If it needs the previous sentence, skip it.
3. **Curation.** Balance positions within each topic. Many speakers, with a per-speaker cap.
4. **Proofreading.** Check each item word for word against the source. Remove transcription errors, list markers, and symbol junk.
5. **Labelling.** Label each item as a position on a concrete proposition (for example, "The government should reduce immigration": agree / disagree / neither), not as left or right. Mapping positions to left/right is separate, per-country metadata. Two people label independently without seeing the speaker. Items they disagree on are dropped, not forced. The owner sets the minimum agreement before labelling starts.
6. **Translation, later.** Prefer real native-language statements over translations. Anything translated is back-translated and checked by someone who reads that language. Only languages someone on the project can actually read.

Pilot every stage on a small batch (for example, one debate) before scaling.

Things that broke the last dataset, which must not come back: labels decided by a category code instead of what the sentence says; sentence fragments; topic that predicts the label; tone or negation that predicts the label. Every experiment needs baselines for these.

## 3. How to work with the owner

### Teach, don't ship, and chunk the code

- Never drop a large end-to-end pipeline. One concept, one small chunk (tens of lines), the owner runs it, discuss what they saw, then the next chunk.
- Boilerplate (imports, argparse, file I/O) can be written quickly. Conceptual code (labelling logic, probes, baselines, metrics) is chunked and explained before it runs.
- Prefer questions on conceptual pieces: "What do you expect if we remove the topic words?"

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
├── docs/            # lessons, data spec, labelling guide, bugs-squashed, writeups
├── data/            # git-ignored until each source's licence is checked
└── src/             # small scripts
```

## 9. TL;DR

1. Data first. No model work until the owner signs off on the data.
2. Items must make sense on their own; labels are positions on propositions, checked by two people.
3. Teach in small chunks. Ask before changing anything.
4. Public repo: no cloud IDs, credentials, or restricted text.
5. Read `~/Desktop/gcp-workspace/SKILL.md` before any cloud action.
