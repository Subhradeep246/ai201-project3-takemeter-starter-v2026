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

## 1. Overall accuracy of at least 0.55 on the held-out set

**Why this target:** Three labels, so a coin-flip is ~0.33 and always guessing
the biggest class would sit around 0.40 if I keep the set from getting
lopsided. 0.55 is the lowest number I would still call useful for
auto-flairing r/cscareerquestions — above a majority-class dummy, below a
number that one friendly 30-row test split could luck into. Unit 6 has to
hold this on all three seeds, and ~30 test rows move around, so I am not
writing 0.65.

---

## 2. Macro F1 of at least 0.50 on the held-out set

**Why this target:** Accuracy can look fine if the model sits on
`advice_seeking` and ignores the other two. Macro F1 averages the three
labels equally, so it is the number that would actually tell me the
classifier is sorting the taxonomy, not just the majority speech act. 0.50
is a hair under the accuracy target because one thin class on ~8–12 test
rows can drag the mean down without the model being useless.

---

## 3. Every label has an F1 of at least 0.40 on the held-out set

**Why this target:** `situation_report` is the smallest class I expect to
end up with, around 50–70 of 200, so its test slice is about 8–12 posts. An
F1 on 8 rows jumps if two of them flip. 0.40 is loose on purpose for that
class, and I am applying it to all three rather than giving
`advice_seeking` a vanity number I already know it will beat. If any label
comes back at 0.00, the taxonomy did not survive contact with DistilBERT.

---

## 4. The gap between the best-label F1 and the worst-label F1 is at most 0.35

**Why this target:** If `advice_seeking` lands at 0.80 and `industry_take`
lands at 0.20, the model learned "question mark vs. claim" and nothing
about the harder boundary. A 0.35 spread is the most imbalance I will
still call a working three-way classifier. Tighter than that is asking the
smallest test slice to match the biggest one, which the seed-spread warning
says it will not.

---

## 5. The most-confident third of test predictions is at least 0.10 more accurate than the least-confident third

**Why this target:** The notebook already writes this number, and it is the
one I would actually use: auto-flair the sure posts, leave the unsure ones
for a human. A 10-point gap is the smallest gap I would trust enough to
act on. If the two thirds score about the same, the model's confidence is
noise, and that is a miss even if accuracy is fine.

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
