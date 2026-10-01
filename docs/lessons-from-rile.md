# Lessons from the RILE attempt

Last updated: 2026-10-01

The previous version of this project lives in the private local repo `~/Desktop/Projects/AI safety`, project folder `src/crosslingual-political-repr/`, last commit `9224961` (2026-10-01). Its full results record is `src/crosslingual-political-repr/docs/dataset_manifesto_rile.md` there. Nothing from that dataset is copied here: the Manifesto Project's terms forbid redistributing its text, including translations.

## What was built (2026-09 to 2026-10-01)

- About 6,100 statements from party manifestos (English, German, Spanish originals), translated into six languages (en, es, de, zh, hi, mr).
- Labels came from the Manifesto Project's RILE scheme: each statement's category code decides its side. For example, every "military positive" statement counts as right and every "welfare expansion" statement counts as left.
- Linear probes (simple classifiers on model hidden states) were trained in one language and tested in the others, for four open models, with character n-gram, word n-gram, and length/punctuation baselines.

## What the measurements showed

- On the development data, averaged over source/target language pairs, probes scored 53 to 64% balanced accuracy (50% is chance).
- On four categories held out from all training and layer selection, those averages fell to 49.6 to 57.4%.
- On held-out categories, one model was right about 37% of the time on the two left categories and 69% on the two right ones: it mostly guessed one side.
- Any two models disagreed on 45 to 49% of held-out statements, close to what two independent guessers would do.
- Within a single language, character n-grams (67% on development data) matched or beat the probes.
- Owner's blind check on 50 random statements (English text only, no labels shown): 17 could not be called left or right at all; on the other 33, agreement with the RILE label was 18/33 (55%). Most disagreements came from the category table, for example "tackling domestic violence" labelled right because it is coded law and order, and protectionist trade statements labelled left.

## What went wrong

1. **Labels came from category codes, not from what each sentence says.** Topic and label were the same variable, so a probe could score by recognising the topic.
2. **Many items were fragments.** The Manifesto Project splits text into "quasi-sentences"; about 11% started with a connecting word like "and", and many did not make sense alone.
3. **Weeks of careful analysis happened before anyone read the data.** The pipeline had strict provenance and statistics, but the input was never checked by a human for readability or label sense until the end.
4. **Baselines that could not compete across scripts.** N-gram baselines had no shared vocabulary between, for example, English and Hindi, so "probe beats baseline across languages" was a weak comparison.
5. **Documentation outgrew understanding.** The results record reached 1,273 lines with hash tables; the core finding fits in a paragraph.

## Rules that came out of it

- Read the data first. A human blind check of a random sample comes before any modelling.
- Every item must make sense on its own.
- Label positions on concrete propositions, per item, by two people; left/right mapping is separate per-country metadata.
- Hold topic fixed and vary only the position, so recognising the topic is worth nothing.
- Baselines for topic, tone, negation, and a multilingual sentence-embedding model.
- Prefer native-language statements over translations; only languages someone on the project can read.
- Keep documentation short enough that the owner can explain the result out loud.
