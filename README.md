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
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



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
