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

**Pre-labelling disclosure:** After 21 cold labels, a keyword heuristic pre-sorted the rest into three piles. I then read every keep row and overrode the pile name whenever the decision rule said so. The heuristic systematically treated first-person stories with a question at the end as reports; review moved those to `advice_seeking`. True `industry_take` news/analysis posts were kept. Rows that were megathreads, Discord/product promos, or duplicates were dropped, not relabelled.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



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
