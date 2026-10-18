# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1.

Overall accuracy on the held-out test split is at least 0.65, and it holds on all three seeds.

**Why this target:**
With three labels a coin-flip guesser scores about 0.33, and always guessing the biggest label scores at most about 0.55 if criterion 4 holds. 0.65 asks for a clear lead over chance on a task where my own stress test left 2 of 8 boundary posts debatable. I wouldn't trust a classifier that is wrong on one post in three, and a test split of about 30–40 posts makes anything much above 0.70 a stretch.

---

## 2.

Each label's own F1, computed per label on the test split (`test_split.csv`), is at least 0.55, on all three seeds (42, 7, 2024). One label below 0.55 on one seed is a miss.

**Why this target:**
This stops a high accuracy from hiding one label the model never learned. 0.55 is looser than the 0.65 accuracy target because my smallest label will have only about 8 posts in test, so its F1 will swing between seeds. I'd rather set a floor I can honestly defend on 8 examples than a tight number I'd hit by luck.

---

## 3.

Read from the test-split confusion matrix: the count of true `analysis` posts predicted `hot_take`, divided by the total true `analysis` posts, is at most 0.25, and the count of true `hot_take` posts predicted `analysis`, divided by total true `hot_take` posts, is also at most 0.25. Both hold on all three seeds. The fraction is compared exactly with no rounding, so 3 of 10 (0.30) is a miss and 2 of 8 (0.25) is a pass.

**Why this target:**
`analysis` vs `hot_take` is the boundary the whole taxonomy rests on: does the post cite a specific, checkable fact or not. Posts like "Cyberpunk's launch reviews were glowing" sit right on that line, so I expect most errors there. A 25% ceiling is generous enough for the genuinely ambiguous posts but tight enough that the model has to be reading for specifics, not tone.

---

## 4.

In the final committed `labels.csv` (all rows, before any train/test split), no label accounts for more than 55% of the rows, and every label has at least 30 rows. The percentage is out of however many rows the file has, which must be at least 150.

**Why this target:**
The README caps any label at 70%, but at 70% a model that always says the biggest label gets 0.70 accuracy and would pass criterion 1 without learning anything. r/truegaming's top posts lean toward argued, fact-backed writing, so `analysis` is the label most likely to swamp the others. 30 posts per label puts about 5–6 of the smallest label in a 15% test split, which is already thin, so I won't go lower.

---

## 5.

When I label the 30 posts in `data/staff_posts.csv` using the staff definitions in `data/staff_taxonomy.md` (not my own taxonomy), `agreement.py` reports at least 21 of 30 (70%) matching the staff labels.

**Why this target:**
The staff set uses a different taxonomy and was picked to include ambiguous posts, so 100% isn't realistic and I don't expect to be right on every one. If I can't agree with staff on 7 of 10 posts using written rules, my own labels probably aren't consistent enough for a model to learn from. 70% is also about where my own boundary stress test landed on a first pass.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
