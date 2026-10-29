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

A DistilBERT classifier for [r/cscareerquestions](https://www.reddit.com/r/cscareerquestions/).
That subreddit mixes three different speech acts in one feed: people who still
have a decision in front of them, people who are reporting something that
already happened, and people arguing about the industry. The model sorts a
post into `advice_seeking`, `situation_report`, or `industry_take`. The point
of the split is auto-flairing — the replies a layoff story needs are not the
replies an offer-comparison needs.

---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `advice_seeking`

**Definition:** The writer wants help with a personal next step — a live decision about their own job, offer, team, or skills.

**Example 1:**
> Leave team after 1.5 years? I got a sub-par review on a deprecation team. A friend is starting an AI product in another division and wants me to join. Would leaving push my promotion timeline back?

**Example 2:**
> Should I learn Python or C++? I'm a frontend developer trying to pick which language to focus on first before I'm made obsolete.

### `situation_report`

**Definition:** A first-person account of something that already happened to the writer — informing, venting, or celebrating — with no ask about what they should do next.

**Example 1:**
> Got laid off by AI. Seventeen of us were told our roles were redundant on a Teams call during probation. I'm upset. I guess I'll take some time off and brush up DSA.

**Example 2:**
> Signed 300 TC Seattle offer. Hated my last job, took six months off, recruited for 1.5 months, got two offers. Reflection: strong AI-tool skill is red hot right now.

### `industry_take`

**Definition:** A general claim or discussion about tech careers, AI, hiring, companies, or the market, including questions about the world rather than the writer's own next step. A checkable fact counts if it carries the claim.

**Example 1:**
> Software-development employment has continued to increase since ChatGPT. CPS data, 2022 to now: employment up, unemployment also up, and the gain is stronger for ages 30–44 than for people in their 20s.

**Example 2:**
> AI is the worst invention to ever hit humanity. Mass layoffs, worthless degrees. If you are still pro-AI at this point, you are a fool or a shill.

### The hardest boundary

**Which two labels:** `advice_seeking` vs `situation_report`

**The decision rule I used every time:**
If the writer still has a live decision, or an explicit ask the comments can
answer, it is `advice_seeking` even if ninety percent of the post is story.
If the outcome already happened and they are informing, venting, or
celebrating, it is `situation_report`. A trailing "I guess I'll start
applying" is a plan, not an ask. "Got laid off, should I take this other
offer?" is `advice_seeking`.

---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** r/cscareerquestions, pulled from the public
Arctic Shift archive (self-posts plus comments). I kept English posts between
about 160 and 1,800 characters, dropped `[removed]` / `[deleted]` / AutoModerator,
and deduped on the text. Original posts carry the title plus body. Comments
are the body only. I downsampled `advice_seeking` and `industry_take` so the
file would not be 70% questions — that is the natural base rate on this
subreddit, and it is not a fair training set.

**How I labelled them:** Twenty-one rows were labelled cold, from reading,
before any heuristic. The rest were pre-sorted with a keyword heuristic
(question marks / "should I" vs layoff/offer language vs industry-claim
language), then **every keep row was read and corrected**. The heuristic was
noisy: the `situation_report` pile was mostly live asks, and the
`industry_take` pile mixed in offer-choice posts. Net effect of review was
moving a lot of rows *onto* `advice_seeking`, not off it.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `advice_seeking` | 86 | 43.0% |
| `industry_take` | 85 | 42.5% |
| `situation_report` | 29 | 14.5% |
| **Total** | 200 | 100% |

`situation_report` is the thin class on purpose. Genuine closed stories are
rare on this subreddit — most vents still ask "what do I do". I did not
relabel those as reports just to pad the class. Test F1 on ~4–5 rows will
jump between seeds; that is in `criteria.md` already.

The table above is the set that unit 5 and the unit 6 before-run trained on.
During unit 6 I moved 8 rows from `situation_report` to `industry_take`
because they broke the decision rule (questions about the world, general
claims, third-person anecdotes). After that move the file is
`advice_seeking` 86 (43.0%), `industry_take` 93 (46.5%),
`situation_report` 21 (10.5%). No label is over 70%.

**Three hard cases**

**1.**
> *The post:* Got placed as an Associate Full Stack Developer, then seventeen people were told on a Teams call that the roles were redundant because of AI. "I'm upset... I guess I'll take some time off and brush up DSA again and attempt at another job."
>
> *Could have been:* `advice_seeking` (they mention looking again) or `situation_report`.
>
> *I chose, because:* `situation_report`. The live event is the layoff. "I guess I'll..." is a plan they already stated, not a question the comments are being asked to answer.

**2.**
> *The post:* Recently laid off in a restructure, 10 YOE, senior title, passion gone, considering leaving tech. "At what point is it burnout/imposter syndrome vs a sign that the career isn't for me anymore?"
>
> *Could have been:* `situation_report` (the layoff already happened) or `advice_seeking`.
>
> *I chose, because:* `advice_seeking`. The layoff is backstory. The unanswered question is whether to stay in the career, and that is still in front of them.

**3.**
> *The post:* "If you think AI is gonna replace Software Devs, you're not a real SWE" was the mood here six months ago. "This sub, and everyone in this industry, is full of shit." Then: when I started trusting myself, my opportunities expanded.
>
> *Could have been:* `situation_report` (personal turnaround) or `industry_take`.
>
> *I chose, because:* `industry_take`. The load-bearing claim is about the subreddit and the industry. The personal sentence is the illustration, not the job of the post.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate `2e-5`, batch size 16, seed 42, max length 128. Device is whatever section 1 of the notebook prints — this machine is CPU.

**Anything I changed from the defaults, and why:** Nothing. Hyperparameters are vocabulary this unit, not decisions. The labels in the notebook were changed to match `labels.csv`: `advice_seeking`, `situation_report`, `industry_take`.

**Split sizes:** Stratified 70/15/15, seed 42: train 139 / val 31 / test 30.

| Label | train | val | test |
|---|---|---|---|
| `advice_seeking` | 60 | 13 | 13 |
| `industry_take` | 59 | 13 | 13 |
| `situation_report` | 20 | 5 | 4 |

`situation_report` has 4 examples in test — under 8. Its F1 next unit will jump between seeds, which is why criterion 3 is 0.40 rather than matching the other two labels.



---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* A first pass at a three-label taxonomy that would actually balance on r/cscareerquestions, not a copy of the running-forum `analysis` / `hot_take` / `reaction` example.
- *What came back:* Topic splits (`job_choice` / `skill_path` / `workplace`) and a speech-act split. The topic split would have been easier to balance.
- *What I changed:* Kept the speech-act split. The hard boundary I actually needed was "is this person still deciding, or are they reporting." Padding `situation_report` by weakening that rule would have made the criteria untestable.

**Moment 2**

- *What I asked for:* Help standing up the venv, pinning, and the notebook `LABELS` so section 2 would not die on a spelling mismatch.
- *What came back:* The install, `test.py` green except the expected baseline-model warning, and the three label strings in the notebook.
- *What I changed:* Left training settings at the starter defaults. Wrote `criteria.md` before looking at any trained numbers.

**Moment 3 — unit 6**

- *What I asked for:* Run the unit 6 test in order: baseline, three seeds, the staff-set labels, a verdict on each criterion, one fix, and the re-run. Write the baseline prediction before the comparison.
- *What came back:* The zero-shot run, the two three-seed files, and a pass over the 29 `situation_report` rows against the decision rule.
- *What I changed:* Kept the prediction as it was written, including the part that turned out wrong (I expected the fine-tune to beat the baseline on the two large classes). Moved 8 rows that were not closed first-person stories. Left the five targets in `criteria.md` where unit 5 put them.

**Pre-labelling disclosure:** After 21 cold labels, a keyword heuristic pre-sorted the rest into three piles. I then read every keep row and overrode the pile name whenever the decision rule said so. The heuristic systematically treated first-person stories with a question at the end as reports; review moved those to `advice_seeking`. True `industry_take` news/analysis posts were kept. Rows that were megathreads, Discord/product promos, or duplicates were dropped, not relabelled. Unit 6 found 8 rows in `situation_report` that this review still got wrong. Those moves are noted on the rows.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

Same 30 posts (`test_split.csv`). Baseline is `facebook/bart-large-mnli` with the definitions in `label_definitions.txt`. Trained is the unit 5 seed-42 DistilBERT run in `results.json` (CPU, torch 2.14.1). This machine's unit 6 re-runs are torch 2.15.0+cpu, also CPU.

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.700 | 0.667 | -0.033 |
| Macro F1 | 0.508 | 0.478 | -0.030 |
| F1 — `advice_seeking` | 0.750 | 0.706 | -0.044 |
| F1 — `industry_take` | 0.774 | 0.727 | -0.047 |
| F1 — `situation_report` | 0.000 | 0.000 | +0.000 |

**What I predicted before I looked:**

Written before `baseline.py` ran. The seed-42 trained file already existed from unit 5, so this is a prediction about the gap, not a first look at those numbers. Zero-shot only gets the three definitions. `advice_seeking` and `industry_take` are easy hypotheses to entail — a personal next step versus a claim about the industry — so I expect the baseline to beat a majority-class guess and to get a useful share of those two right. `situation_report` is defined by the absence of an ask, and a layoff post still narrates what the writer did next, so I expect the baseline to call almost all of them `advice_seeking`. I expect the fine-tune to beat the baseline on `advice_seeking` and `industry_take`, and I do not expect it to grow a `situation_report` detector out of about twenty training rows. If the two overall accuracies land close, fine-tuning mostly rediscovered the definitions.

**What the gap actually means:**

Fine-tuning added nothing. On the same 30 posts the zero-shot model is more accurate (0.700 vs 0.667) and has a higher macro F1 (0.508 vs 0.478). It is ahead on both large classes. Both models score 0.000 on `situation_report`: the baseline never got one of the four right, and the fine-tune called all four `advice_seeking`. The definitions already separate a personal ask from an industry claim. Three epochs on 139 rows did not beat that, and twenty `situation_report` training rows were not enough to teach a class the definitions also miss. That is the finding.



---

## Run Log — Before

Hardware for all three seeds: CPU, torch 2.15.0+cpu. Same code path as the notebook's section 6 (`distilbert-base-uncased`, 3 epochs, lr `2e-5`, batch 16, max length 128). Accuracy moved 0.100 across seeds. Seed 42 and seed 7 landed on the same hard-label counts; their confidence means differ (0.453 vs 0.441 when right), so the runs are not copies of one file.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.55 | 0.667 | 0.667 | 0.567 | MET |
| 2. Macro F1 | ≥ 0.50 | 0.478 | 0.478 | 0.379 | MISSED |
| 3. Worst-label F1 (`situation_report`) | ≥ 0.40 | 0.000 | 0.000 | 0.000 | MISSED |
| 4. Best F1 minus worst F1 | ≤ 0.35 | 0.727 | 0.727 | 0.667 | MISSED |
| 5. Confident-third gap | ≥ 0.10 | 0.400 | 0.500 | 0.500 | MET |

Criterion 3 is the minimum F1 across the three labels. On every seed that minimum is `situation_report`. The other two labels cleared 0.40 on every seed (`advice_seeking` 0.706 / 0.706 / 0.667, `industry_take` 0.727 / 0.727 / 0.471).

### Confusion matrix

Seed 42, the first of the three. Rows are the true label.

| true \ predicted | advice_seeking | situation_report | industry_take |
|---|---|---|---|
| **advice_seeking** | 12 | 0 | 1 |
| **situation_report** | 4 | 0 | 0 |
| **industry_take** | 5 | 0 | 8 |

**My biggest off-diagonal number, and what it means:**

5 real `industry_take` posts were called `advice_seeking`, and only 1 real `advice_seeking` post was called `industry_take`. The model leans toward `advice_seeking` on that pair. The `situation_report` column is all zeros: the model never predicted the label. All 4 real reports were called `advice_seeking`, and 0 went the other way, because the other way never happened. Seed 7 has the same matrix. Seed 2024 sends 9 of 13 `industry_take` posts and all 4 reports to `advice_seeking`.



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.55 | MET | 0.667, 0.667, and 0.567 all clear 0.55. Seed 2024's margin is 0.017, smaller than the 0.100 seed spread, so the call is fragile. |
| 2 | Macro F1 | ≥ 0.50 | MISSED | 0.478, 0.478, 0.379. Every seed is under 0.50. |
| 3 | Every label's F1 | ≥ 0.40 | MISSED | `situation_report` is 0.000 on all three seeds. |
| 4 | Best F1 minus worst F1 | ≤ 0.35 | MISSED | 0.727, 0.727, 0.667. The gap is the 0.000 on `situation_report`. |
| 5 | Confident-third gap | ≥ 0.10 | MET | 0.400, 0.500, 0.500. The sure third is right 0.90 of the time on every seed. |

**Diagnoses**

Criteria 2, 3, and 4 miss for the same reason. `situation_report` has F1 0.000 on every seed, the model never emits the label, and that zero pulls macro F1 under 0.50 and opens a gap well past 0.35.

Two causes, and they are separable.

**Too few examples.** The before-run file had 29 reports. Every seed put 4 in the test split. A label the model never predicts cannot score above 0 on a support of 4, and a support of 4 would jump between seeds even if the model were trying. The zero column says it is not trying. The plurality class is `advice_seeking`, and first-person career text is what that class looks like, so the model parks reports there.

**The boundary was applied inconsistently.** I re-read all 29 `situation_report` rows against the decision rule in this file. Eight are not closed first-person stories:

- "Are your lead devs always in meetings?" and "Do you guys ever see a disconnect…" are questions about the world.
- "I'm curious to know what others think" is an ask, and the writer has already resigned.
- The Meta-contractor comment, the "you are a number / take the money" comment, and the scrum-master comment are general claims. The personal sentence is the illustration. That is the hard-case rule I already wrote for `industry_take`.
- "Had a friend of a friend…" and "Knew a guy who…" are third-person anecdotes. The definition requires something that happened to the writer.

That re-read is the labelling evidence. `data/staff_labels.csv` is not in this repo — the starter says the TF sends it after `my_staff_labels.csv` exists — so `agreement.py` cannot yet score me against staff. I labelled the 30 staff posts anyway, under `data/staff_taxonomy.md`, and the same kind of miss shows up there as a risk: the staff rules turn on whether a fact is load-bearing, and I had already mis-filed posts on my own set where a personal sentence was decoration on a claim. Until the staff file arrives I cannot quote a rate. The evidence I do have is those 8 rows.

The pair itself is also hard. A layoff story and an advice post share first person, a workplace, and a next step. The rule that separates them is the absence of an ask. Zero-shot, given that definition in plain text, also scored 0.000 on `situation_report`. So this is not only a fine-tune failure. The class is small, some of its rows were the neighboring classes, and the boundary is a negative.

Criteria 1 and 5 held. Accuracy stays above a majority-class guess because `advice_seeking` and `industry_take` are learnable, and the confident third is actually more accurate than the unsure third. The model's confidence is usable on the two classes it knows. It is not evidence that `situation_report` works.



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** not computed. `my_staff_labels.csv` is done (30 rows, text pasted unchanged). `data/staff_labels.csv` is not in the repo. `agreement.py` stops without it and tells you to ask the TF, which matches `data/staff_taxonomy.md`. I am not filling in a rate or a staff label I do not have.

My labels, under the staff rules: `sp_001`–`sp_010` `analysis`, `sp_011`–`sp_020` `hot_take`, `sp_021`–`sp_030` `reaction`.

How the rules got applied on the posts that look like traps:

- `sp_014` ("times are down about 3%… so obviously the shoes are the whole story") is the taxonomy's own `hot_take` example. The 3% is a flourish.
- `sp_005` ("2% in the shoes and another 2% the following year… 35 to 50 miles a week") is `analysis`. The second number is what separates mileage from the shoe. Same shape as the practice-set posts that compare a runner's own logs.
- `sp_016` ("gel before 90 minutes") is `hot_take`. The practice set labels "Nobody needs a gel before 90 minutes" as `hot_take`. The time is the claim, not evidence for a separate claim.
- `sp_007` and `sp_010` stay `analysis` under rule 3: a personal run is still analysis when the measurement carries the argument.
- `sp_027` ("leaning at about 20 degrees… coming back felt like cheating") is `reaction`. The number is color, and the post is about how the run felt.
- `sp_028` ("Third DNF this year… I don't really want to talk about it") is `reaction`. The count does not carry an argument. There is no argument.

**Disagreements**

None I can adjudicate until the staff file is here. Inventing a staff label to argue with would be a fake disagreement.

**What the pattern in my disagreements tells me:**

I cannot use a staff rate I do not have to separate a labelling problem from a model problem. The substitute I used is the re-read of my own 29 reports, documented under Diagnoses. Eight of them broke a rule I had already written down. That is the pile-(a) pattern on my own set: the rule covered it, and the label did not follow it. It is weaker evidence than the staff set, because I am the one applying the rule and the one grading the application. When `data/staff_labels.csv` arrives, `python agreement.py` will list the real disagreements and this section should be replaced with those.



---

## The Improvement

**What I changed:**

I re-applied the unit 5 decision rule to the 29 `situation_report` rows and moved 8 of them to `industry_take`. Nothing else changed: same model, same epochs, same learning rate, same batch size, same three seeds. The rows and the reason on each are in the `note` column. Counts after the move: `advice_seeking` 86, `industry_take` 93, `situation_report` 21.

**Which diagnosis pointed at it:**

The miss on criteria 2, 3, and 4 is `situation_report` at F1 0.000, with every test report called `advice_seeking`. The re-read showed 8 of the 29 training labels were actually questions, general claims, or third-person anecdotes. Teaching the model those rows as reports would train the neighboring classes into the thin one. The fix the diagnosis points at is to take those rows off the label, then train again. I did not lower the targets in `criteria.md`.

### Run Log — After

Same hardware (CPU, torch 2.15.0+cpu) and the same training settings. The split is new, because the labels changed: each seed now has 3 `situation_report` rows in test, not 4. Before and after are not the same 30 posts.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.55 | 0.767 | 0.767 | 0.667 | MET |
| 2. Macro F1 | ≥ 0.50 | 0.536 | 0.540 | 0.463 | MISSED |
| 3. Worst-label F1 (`situation_report`) | ≥ 0.40 | 0.000 | 0.000 | 0.000 | MISSED |
| 4. Best F1 minus worst F1 | ≤ 0.35 | 0.839 | 0.846 | 0.774 | MISSED |
| 5. Confident-third gap | ≥ 0.10 | 0.300 | 0.400 | 0.500 | MET |

**Did it help, and how do I know:**

It helped the two classes the model already had, and it did not help `situation_report`. Accuracy rose on every seed (0.667→0.767, 0.667→0.767, 0.567→0.667). Macro F1 rose on every seed (0.478→0.536, 0.478→0.540, 0.379→0.463), enough for seeds 42 and 7 to clear 0.50 and not enough for seed 2024, so criterion 2 stays a miss. `situation_report` F1 stayed 0.000. The after matrices still have a zero column for that label. On seeds 42 and 7 the three real reports are called `advice_seeking` (2 and 3 of them) or `industry_take`. On seed 2024 all 3 are called `industry_take`. Taking the mislabeled rows off the class made `industry_take` sharper — seed 42 went from 8/13 correct to 13/14 — and left the model with no report class to emit. Criterion 4 got worse because the best F1 went up and the worst stayed at zero. Relabelling was the right correction to the file. It was not a way to make 21 examples of a negative class learnable.



---

## What's Still Broken

**Criterion 2, macro F1 ≥ 0.50.** Seed 2024 is 0.463 after the fix. The other two seeds clear it. The miss is the seed where `industry_take` F1 drops to 0.615 and `situation_report` is still zero, so the average falls under 0.50. What I would do next is add closed first-person stories until the test slice is large enough that one seed cannot swing the macro average by this much. I stopped because the posts I still have are the 21 that survived the rule, and inventing more by calling live asks "reports" is the padding unit 5 already refused.

**Criterion 3, every label F1 ≥ 0.40.** `situation_report` is 0.000 on every seed before and after. The model never predicts it. What I would do next is the same collection: more rows where the outcome already happened and there is no ask, on the order of the other two classes, and then train again. I stopped because 21 rows, with 3 in test, is the set the rule actually produces from this scrape. Lowering 0.40 would hide that.

**Criterion 4, best-minus-worst F1 ≤ 0.35.** The gap is 0.839, 0.846, 0.774. It widened after the fix because `industry_take` got better and the worst label stayed at 0. The gap closes when criterion 3 closes. Same stop.

**The gap between what I meant my labels to capture and what the model learned:**

I meant `situation_report` to be a closed first-person story with no ask. The model learned two classes, personal ask and industry claim. After the relabel the `situation_report` column is still all zeros, and the real reports are absorbed into `advice_seeking` on seeds 42 and 7 and into `industry_take` on seed 2024.



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
