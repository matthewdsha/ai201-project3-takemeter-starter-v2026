# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Your community, and what your classifier sorts posts into. Three or four
     sentences. -->

The community is r/truegaming, a public discussion subreddit about games in
general, not tied to one title or release. Its posts range from ones that
argue from numbers and named mechanics to confident claims with nothing behind
them. The classifier sorts each post by what its argument rests on: a specific,
checkable fact (`analysis`), nothing beyond confidence (`hot_take`), or the
author's own experience (`personal_account`).



---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `analysis`

**Definition:** Backs its claim with at least one specific, externally
checkable fact: a number, date, named source or direct quote that a reader
could verify without trusting the author.

**Example 1:**
> Spider-Man fast-travel post: 66.43% of players earned the trophy, and 71.91%
> finished Act 3.

**Example 2:**
> Fortnite post: frame rate dropped from ~138 fps to 60–80 after Lumen was
> enabled.

### `hot_take`

**Definition:** States a claim with no specific, checkable support, including
vague gestures ("everyone I know") and bare examples with no detail.

**Example 1:**
> "Pre-ordering GTA 6 right now is madness."

**Example 2:**
> "Most players don't actually want freedom; they want reassurance."

### `personal_account`

**Definition:** The only evidence is the author's own experience (their
playtime, spending or reactions), which nobody else can check.

**Example 1:**
> The "joyless survival horror" post: a verdict resting on the author's own
> playthrough.

**Example 2:**
> The $1,400 microtransaction audit: numbers drawn only from the author's own
> purchase history.

### The hardest boundary

**Which two labels:** `analysis` vs `personal_account`. The $1,400 audit is
full of numbers but none can be checked from outside the post.

**The decision rule I used every time:**
A fact counts as checkable only if a reader could verify it without trusting
the author. A public stat, patch note, or benchmark with a named setup
qualifies. The author's own spending, hours or "I noticed" does not, so it is
`personal_account` even when numbers are present. Apply in order:

1. If the post has a specific, externally checkable fact and its argument rests
   on it, it is `analysis`. A fact that is only decoration on an unrelated
   conclusion doesn't count.
2. Otherwise, if the author's own experience is the main evidence, it is
   `personal_account`.
3. Otherwise it is `hot_take`.

If a post has both a checkable fact and personal experience, rule 1 wins.
"Specific" means a number, date, named source or quote. "Glowing reviews" or
"Hades restarts fast" is not specific.



---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** Public text posts from r/truegaming. I saved five listing pages as JSON (hot, new, and three pages of the main listing), which gave 500 entries and 340 unique posts. I removed stickied posts, "casual talk" megathreads, academic survey recruitment posts and pure help-or-recommendation questions, since none of those makes a claim. The posts are whole, title plus body. I only cleaned HTML entities, and I didn't trim any post. The result is one unsplit `labels.csv` with 272 posts.

**How I labelled them:** Cold first. I labelled 20 posts myself on paper, with no AI and no tools, before seeing any AI label. Those rows say "cold" in the note column. Four posts from my original cold set were dropped as non-takes (two questions, one survey post and one advice request), so I labelled four fresh posts cold to bring the total back to 20. The other 252 rows were pre-labelled by an AI tool (Claude) using the definitions in the Label Taxonomy section and the decision rule below it, and say "pre-labelled" in the note column. The AI read the opening of each post plus its numbers and links, not always the full text. I did review those 252 rows and agreed with them. The 10 hardest rows say "hard case" in the note column.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `hot_take` | 144 | 52.9% |
| `personal_account` | 97 | 35.7% |
| `analysis` | 31 | 11.4% |
| **Total** | 272 | 100% |

No label is above 70%. `analysis` is the thin one: with about 31 examples, only about 4 land in the test split, so its F1 will move a lot between seeds.

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**
> *The post:* "Why Is Game Optimization Getting Worse?", written by a developer with 13 years in the industry. It says traditional content costs like polycount and texture resolution are "only like 30-40% of your frame time now," and the other "60-70%" is black-box systems like Lumen, Nanite and TAA. It also says day-one patches can be "50+ gigabytes."
>
> *Could have been:* `analysis` or `personal_account`.
>
> *I chose:* `personal_account`, because the post is dense with numbers, but none can be checked from outside it. The percentages are the author's own estimates from their own projects, and a reader has to trust the author's career to accept them. My rule says numbers that come only from the author's own experience don't count as checkable, so this is the same call as the $1,400 audit.

**2.**
> *The post:* "The best performance category in TGA always doesn't make a lot of sense." It says Expedition 33 has three nominations in the category, while Ghost of Yotei, Silent Hill F and Indiana Jones have one each. All three E33 nominees gave voice performances only, while the Ghost of Yotei and Silent Hill F nominees gave motion capture, physical likeness and voice. It concludes the award is "quite unfair."
>
> *Could have been:* `analysis` or `hot_take`.
>
> *I chose:* `analysis`, because anyone can check the nomination counts and who did voice versus mocap work, and the argument rests on exactly those facts. The "unfair" verdict is opinion, but rule 1 says a checkable fact the argument depends on wins. Without those specifics it would be a plain `hot_take`.

**3.**
> *The post:* "Utility magic that's useful outside of combat should be more common in games." It says that in Skyrim "fire or ice magic will never interact with" the survival system, that in Bioshock 1 the plasmids' "only real use is making combat sections easier," and that Baldur's Gate 3 lets magic feel "like a way to interact with the world."
>
> *Could have been:* `hot_take` or `analysis`.
>
> *I chose:* `hot_take`, because the post names three games and describes their mechanics in some detail, which makes it feel like evidence. But none of it is a number, date, named source or quote. Under my "bare examples with no detail" wording it's still unsupported claims, and my definition of "specific" is a number, date, named source or quote, so concrete examples alone don't make a post `analysis`.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, seed 42, max length 128 tokens. Trained on an Apple Silicon GPU (MPS).

**Anything I changed from the defaults, and why:** Nothing. I ran the notebook's default settings, because this unit is about getting a first trained model and a baseline.

**Split sizes:** 190 train / 41 validation / 41 test, from the notebook's 70/15/15 split of the 272 rows. Per-label counts:

| Label | Train | Val | Test |
|---|---|---|---|
| `analysis` | 22 | 4 | 5 |
| `hot_take` | 100 | 22 | 22 |
| `personal_account` | 68 | 15 | 14 |

`analysis` has only 5 examples in the test split, fewer than about 8, so its F1 will move a lot between seeds and explains much of next unit's variance.



---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* I gave Claude my three definitions and my decision rule and asked it to pre-label the r/truegaming posts I had collected.
- *What came back:* A label for every post. After the first pass only about 9% were `analysis`, because I had filtered out posts over 4,500 characters and those long posts held most of the numbers, quotes and named sources. Claude went back to the long posts I had set aside and labelled those too, which raised `analysis` to about 11%.
- *What I changed:* I read each pre-labelled row and agreed with the label. I also decided the dataset keeps those long posts whole instead of trimming them.

**Moment 2**

- *What I asked for:* I wrote my five acceptance criteria first, then asked an AI, for each target, for one reason it might turn out too easy and one reason it might turn out too hard, without it telling me which was more likely or suggesting different numbers. I followed up by asking whether each criterion could be checked by someone who couldn't ask me what I meant.
- *What came back:* It told me where my criteria fell short and where my guesses were good. It said some of my targets needed tweaking to be more realistic but still fair, and that I needed to be more specific in how the criteria were worded.
- *What I changed:* I followed its advice. I tweaked some of my targets, and I rewrote the criteria to be more specific so someone else could check each one without asking me.

**Pre-labelling disclosure:**
252 of the 272 rows were pre-labelled by an AI tool (Claude) using the definitions in the Label Taxonomy section, and are marked "pre-labelled" in the note column. The AI read the opening of each post plus its numbers and links, not always the full text. I reviewed every one of those 252 rows and agreed with the label. The other 20 rows are my cold labels, written on paper with no AI and no tools before I saw any AI label, and are marked "cold".

**Moment 3 (unit 6)**

- *What I asked for:* After the three-seed run, I had Claude read the confusion matrices and say where the model was going wrong.
- *What came back:* It pointed out that the model never predicts `analysis` (the `analysis` column is 0 on every seed), and that my biggest error was `personal_account` called `hot_take` (12 of 14 on seed 42), not the `analysis`/`hot_take` boundary my criterion 3 assumed.
- *What I changed:* I wrote that into my diagnoses, that criterion 3's miss is the same problem as criterion 2 and not a hard boundary. I left all five criteria as written.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.195 | 0.561 | +0.366 |
| Macro F1 | 0.164 | 0.312 | +0.148 |
| F1 — `analysis` | 0.158 | 0.000 | -0.158 |
| F1 — `hot_take` | 0.333 | 0.700 | +0.367 |
| F1 — `personal_account` | 0.000 | 0.235 | +0.235 |

Baseline: `facebook/bart-large-mnli`, zero-shot, scored on `test_split.csv` (the same 41 posts as the trained model, so no fallback sample). Baseline run with my label definitions. A second run with bare label names only scored accuracy 0.463, macro F1 0.410 (`baseline_results_bare_names.json`).

**What I predicted before I looked:**
I predicted the model would beat the baseline most on `analysis` because it is easy to identify as it needs a fact.

**What the gap actually means:**
My prediction was wrong. `analysis` is the only label where the trained model did worse than the baseline (F1 0.000 vs 0.158): it called all 5 test `analysis` posts `hot_take`. The trained model's biggest win was `hot_take` (+0.367), the largest class. Its overall accuracy of 56.1% is barely above always guessing `hot_take` (22/41 = 53.7%), so the fine-tuning mostly learned the majority class. The baseline's low score is partly a problem with my definitions as prompts: with bare label names it scored 46.3% accuracy and a higher macro F1 (0.410) than the trained model (0.312), so my long definitions were worth about -27 points to this model. Fine-tuning helped on accuracy but has not yet learned the `analysis` boundary, which has only 22 training examples.

---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 on all three seeds | 0.561 | 0.585 | 0.537 | MISSED |
| 2. Per-label F1 | every label ≥ 0.55 on all three seeds | analysis 0.000, hot_take 0.700, personal_account 0.235 | analysis 0.000, hot_take 0.733, personal_account 0.235 | analysis 0.000, hot_take 0.698, personal_account 0.000 | MISSED |
| 3. analysis↔hot_take confusion | analysis→hot_take ≤ 25% and hot_take→analysis ≤ 25% | 5/5 = 100%; 0/22 = 0% | 4/5 = 80%; 0/22 = 0% | 5/5 = 100%; 0/22 = 0% | MISSED |
| 4. Balance | no label > 55%, every label ≥ 30 rows, ≥ 150 rows | 144/272 = 52.9% max, min 31 | same file | same file | MET |
| 5. Agreement with staff | ≥ 21 of 30 | 30/30 | n/a (labels, not a model) | n/a | MET |

Spread across seeds:

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.561 | 0.585 | 0.537 | 0.049 |
| Macro F1 | 0.312 | 0.323 | 0.233 | 0.090 |
| F1 — `analysis` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `hot_take` | 0.700 | 0.733 | 0.698 | 0.035 |
| F1 — `personal_account` | 0.235 | 0.235 | 0.000 | 0.235 |

Accuracy moved only 4.9 points, so the measure is stable, but it is stably bad. Always guessing `hot_take` scores 0.537 on the seed 2024 test split (22 of 41), which is exactly what that seed got, and its macro F1 of 0.233 is what a model that only predicts `hot_take` gets. Trained on Apple Silicon GPU (MPS).

### Confusion matrix

Seed 42 (the same run as `results.json`).

| true \ predicted | analysis | hot_take | personal_account |
|---|---|---|---|
| **analysis** | 0 | 5 | 0 |
| **hot_take** | 0 | 21 | 1 |
| **personal_account** | 0 | 12 | 2 |

**My biggest off-diagonal number, and what it means:**
12 real `personal_account` posts were called `hot_take` and only 1 real `hot_take` post went the other way, so the model did not learn that personal experience as the only evidence makes a post `personal_account`, and defaults to `hot_take`. The other failure is that it never predicted `analysis` at all: all 5 real `analysis` posts were called `hot_take`.

---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.65 on all three seeds | MISSED | 0.561, 0.585, 0.537. Not one seed reached 0.65. |
| 2 | Per-label F1 | every label ≥ 0.55 on all three seeds | MISSED | `analysis` F1 was 0.000 on all three seeds. `personal_account` was 0.235, 0.235 and 0.000. Only `hot_take` cleared 0.55 (0.700, 0.733, 0.698). |
| 3 | `analysis` ↔ `hot_take` confusion | each direction ≤ 25% | MISSED | `analysis`→`hot_take` was 5/5, 4/5 and 5/5 (100%, 80%, 100%). `hot_take`→`analysis` was 0/22 each time, which passes, but both directions had to hold. |
| 4 | Balance | no label > 55%, every label ≥ 30 rows, ≥ 150 rows | MET | 272 rows, `hot_take` 52.9%, `analysis` 31 rows. The file doesn't change between seeds. |
| 5 | Agreement with staff | ≥ 21 of 30 | MET | 30 of 30 (100%). |

I didn't revise any criterion. All five were measurable and I missed three of them, so the originals stand in `criteria.md`.

**Diagnoses**

**Criterion 1 (accuracy 0.561 / 0.585 / 0.537): the model hasn't learned much beyond always saying `hot_take`.** Every test split has 22 `hot_take` posts out of 41, so always guessing `hot_take` scores 0.537. Seed 2024 scored exactly that: its matrix has all 41 posts in the `hot_take` column, and its validation accuracy stayed at 0.537 after epochs 1, 2 and 3 while training loss only went from about 0.98 to 0.89. Seeds 42 and 7 are 1 to 2 posts better than that. I can't tell from this run whether the cause is the defaults (3 epochs, learning rate 2e-5) or a task that 190 training examples can't reach. I haven't tried more epochs or class weights, so I can't separate those two yet.

**Criterion 2, `analysis` (F1 0.000 on all three seeds): too few examples.** `analysis` is 22 of the 190 training posts (about 11%) against 100 for `hot_take`, and the model never predicted it, so there is no `analysis` column in any of the three matrices. Precision and recall are both 0, not just low. The agreement check doesn't point at my labeling here: I matched staff on 30 of 30, so I applied a written rule I didn't write the way its authors did, including the "does the fact carry the argument" rule. The zero-shot baseline with bare label names reached `analysis` F1 0.333 on the same posts, so the signal is reachable and my fine-tuned model is not getting it.

**Criterion 2, `personal_account` (F1 0.235 / 0.235 / 0.000): a hard label pair, and the model leans one way.** In seed 42, 12 of 14 real `personal_account` posts were called `hot_take` and only 1 real `hot_take` post went the other way. In seeds 7 and 2024 it was 12 and 14 of 14. This label has 68 training posts, so a lack of examples alone doesn't explain recall of 2/14, 2/14 and 0/14. Both labels are posts with no checkable support, and the difference is whether the author's own experience is the evidence. The agreement check can't rule out a labeling problem on this pair, for two reasons. The staff set uses a different taxonomy with no `personal_account` label, so it never tested this boundary. And 252 of my 272 labels were pre-labelled by an AI and then reviewed by me, so my agreement with staff says little about whether those rows are consistent with each other.

**Criterion 3 (`analysis`→`hot_take` at 100% / 80% / 100%): the same cause as `analysis` above, not a hard `analysis`/`hot_take` boundary.** The two directions are very uneven: `hot_take`→`analysis` was 0 of 22 on every seed, so the model isn't confusing the two, it is not predicting `analysis` at all. My criterion assumed most errors would be on this boundary. The biggest error is actually `personal_account`→`hot_take`.

**What I'd try next, in order:** more epochs or class weights, to see whether the model starts predicting `analysis` at all. Then relabelling a sample of the AI-pre-labelled `personal_account` and `hot_take` rows cold, to measure whether my labels are consistent on the pair the model fails.

---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** 30 / 30 = 100%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

There were none, so there is nothing to adjudicate and no posts to sort into piles (a), (b) and (c).

**What the pattern in my disagreements tells me:**
With no disagreements, the report can't show a labelling inconsistency in how I applied a rule I didn't write. That clears the labelling as the cause of the model's failure on `analysis`, but I don't read 100% as proof my rules are tight. The set was easier than staff's note suggests: the `analysis` and `hot_take` halves split cleanly on whether a figure carries the argument (the sp_014 "3% since the shoe rules changed" post is almost the staff taxonomy's own example of a decorative statistic), and the ten `reaction` posts are short, present-tense and about the writer. It only has 30 posts and none sat on the rule 2 or rule 3 boundaries, so it tests the easy cases more than the hard ones. The hard boundary in my own taxonomy, `analysis` vs `personal_account`, is not tested here at all.

---

## The Improvement

**What I changed:** Training epochs, from 3 to 10 (`EPOCHS` in the notebook). Nothing else: same learning rate (2e-5), batch size (16), seeds (42, 7, 2024), labels and splits, so each seed is scored on the same test posts as before. No rows were relabelled and none were added. Results are in `results_three_seeds_after.json`, and the before numbers are untouched in `results_three_seeds_before.json`.

**Which diagnosis pointed at it:** My criterion 1 diagnosis. In the before run, seed 2024's validation accuracy stayed at 0.537 for all three epochs while training loss only fell from about 0.98 to 0.89, and its matrix put every test post in the `hot_take` column. That looked like a model that had barely started learning, not one that had finished and failed. I did not pick relabelling because the agreement check found no rule applied two ways. I did not pick adding `analysis` examples because there is no pool of unused posts to draw from.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 on all three seeds | 0.659 | 0.732 | 0.732 | MET |
| 2. Per-label F1 | every label ≥ 0.55 on all three seeds | analysis 0.000, hot_take 0.708, personal_account 0.690 | analysis 0.000, hot_take 0.800, personal_account 0.750 | analysis 0.333, hot_take 0.792, personal_account 0.714 | MISSED |
| 3. analysis↔hot_take confusion | analysis→hot_take ≤ 25% and hot_take→analysis ≤ 25% | 5/5 = 100%; 0/22 = 0% | 3/5 = 60%; 0/22 = 0% | 3/5 = 60%; 0/22 = 0% | MISSED |
| 4. Balance | no label > 55%, every label ≥ 30 rows, ≥ 150 rows | 52.9% max, min 31 | same file | same file | MET |
| 5. Agreement with staff | ≥ 21 of 30 | 30/30 | n/a | n/a | MET |

Before and after side by side:

| Measure | Seed 42 before → after | Seed 7 before → after | Seed 2024 before → after |
|---|---|---|---|
| Overall accuracy | 0.561 → 0.659 | 0.585 → 0.732 | 0.537 → 0.732 |
| Macro F1 | 0.312 → 0.466 | 0.323 → 0.517 | 0.233 → 0.613 |
| F1 — `analysis` | 0.000 → 0.000 | 0.000 → 0.000 | 0.000 → 0.333 |
| F1 — `hot_take` | 0.700 → 0.708 | 0.733 → 0.800 | 0.698 → 0.792 |
| F1 — `personal_account` | 0.235 → 0.690 | 0.235 → 0.750 | 0.000 → 0.714 |

Spread across seeds after: accuracy 0.073 (before 0.049), macro F1 0.147 (before 0.090).

Confusion matrix after, seed 42:

| true \ predicted | analysis | hot_take | personal_account |
|---|---|---|---|
| **analysis** | 0 | 5 | 0 |
| **hot_take** | 0 | 17 | 5 |
| **personal_account** | 0 | 4 | 10 |

**Did it help, and how do I know:** Yes, mostly, but not on the label I was most worried about. Accuracy rose on every seed (by 0.098, 0.147 and 0.195) and criterion 1 went from MISSED to MET. `personal_account` is the biggest gain: its F1 went from 0.235, 0.235 and 0.000 to 0.690, 0.750 and 0.714, and in seed 42 it now gets 10 of 14 right where it got 2. `analysis` barely moved. It still scored F1 0.000 on seeds 42 and 7, and in seed 2024 it got 1 of 5 right, which is where its 0.333 comes from. Criterion 3 improved from 100% / 80% / 100% to 100% / 60% / 60% but is still MISSED. So training was a real part of the problem for `personal_account` and for overall accuracy, and not the main cause of the `analysis` failure, which fits that label having only 22 training posts.

Things that qualify this:
- Each test split has 41 posts, so one post is 2.4 points. Seed 42's 0.659 is 27 of 41, one post above the 0.65 line, so criterion 1 is met by the narrowest margin on that seed.
- The spread across seeds got wider, not narrower (accuracy 0.073, macro F1 0.147), so the three numbers are less stable than before.
- It cost something on `hot_take`: real `hot_take` posts called `personal_account` went from 1 to 5 in seed 42 (4 and 3 in the other seeds), where before the model almost never made that mistake.
- I only tried 10 epochs, so I don't know whether 6 or 20 would do better, and I didn't tune anything on the test split beyond running this one change.

---

## What's Still Broken

**Criterion 2 (every label's F1 ≥ 0.55):** still MISSED, entirely because of `analysis`: F1 0.000, 0.000 and 0.333 across the three seeds, and 1 of 15 real `analysis` posts right across the three test sets. `personal_account` and `hot_take` both cleared 0.55 on every seed after the retrain. What I'd do next is add more `analysis` posts (it has 22 of 190 training posts) or weight that class in the loss, then check whether my `analysis` rule separates checkable facts from numbers nobody outside the post can verify. I stopped because the assignment allows one change, and more epochs was the one my diagnosis pointed at.

**Criterion 3 (each direction ≤ 25%):** still MISSED. `hot_take`→`analysis` is 0 of 22 on every seed, which passes, but `analysis`→`hot_take` is 5/5, 3/5 and 3/5 (100%, 60%, 60%). It has the same cause as criterion 2, because the model almost never predicts `analysis`, so the next step is the same one. I stopped for the same reason.

**The gap between what I meant my labels to capture and what the model learned:**
I meant `analysis` to mean a post whose argument rests on a fact someone else could check, but in the seed 42 matrix all 5 real `analysis` posts were called `hot_take` and no post was ever called `analysis`. What the model did learn after more training is the `personal_account` boundary (10 of 14 right), so it can tell a writer's own experience from an unsupported claim, but not yet an unsupported claim from a checkable one.

<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
