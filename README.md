# Spotify App Reviews: AI-Assisted Issue Prioritization

Morgan Henderson and Subrahmanyam Kodiguddu
INFO 4340, Fall 2026, University of Denver

## Overview

Which product problems should Spotify's app team fix first, and can an AI label app reviews reliably enough to track this every week? Star ratings show how unhappy users are but not why, and 61,594 reviews are too many to read by hand. At the center of our workflow, Claude reads each review and assigns it one issue category (e.g., ads, playback problems, app crashes). We test those labels against human labels and a non-AI keyword baseline, then rank issues by how often they appear, how negative those reviews are, and how many users agree with them.

## Key findings

- **Claude labels reviews about as reliably as a second human.** Against our two sets of hand labels (100 reviews), Claude Haiku 5.5 scored accuracy 0.80, macro-F1 0.76, and Cohen's κ = 0.77 (95% bootstrap CI 0.65–0.89 across the two labelers). We agreed with each other at κ = 0.73, and the 0.04 gap is within the minimum detectable difference (0.14).
- **Claude beats the keyword baseline by a real margin.** Keyword rules scored accuracy 0.42 and macro-F1 0.37. Claude's lead (+0.39 accuracy, 95% CI 0.29–0.47; +0.38 macro-F1, CI 0.28–0.48) is far larger than the MDD (0.12 accuracy, 0.08 macro-F1). Claude gave the same label in 3 identical runs for 98% of reviews.
- **Fix app reliability first.** App crashes (11.8% of reviews, 69% of them 1–2★) and music stopping (11.3%, 71% 1–2★) rank #1–2 in 100% of 1,000 bootstrap resamples. Together, ~16% of all reviews are 1–2★ reliability complaints. #3 is a toss-up between UI/update changes (8.6%) and ads (10.0%).
- **April 2022 was a reliability incident.** Music-stopping complaints rose from 9.1% to 18.9% of reviews (+9.9 pts, 95% CI 6.7–13.0), and app-crash complaints from 9.9% to 18.6%.
- **Cheap enough to run monthly:** $0.14 per 1,000 reviews (~$1.41 per 10K). Total project API spend was $0.45. Human review of low-confidence labels (< 0.70) would catch 20 of 22 errors while flagging 38.5% of reviews.

## Data

- **Source:** [Spotify App Reviews 2022](https://www.kaggle.com/datasets/mfaaris/spotify-app-reviews-2022/data) (public Kaggle dataset of Google Play Store reviews)
- **Size:** 61,594 reviews, January–July 2022
- **Unit of observation:** one user review (text, 1–5 star rating, thumbs-up count, timestamp, and Spotify's reply if any)
- **Access:** included in `data/raw/reviews.csv`. No usernames or personal identifiers are in the data.

## Workflow

1. **Cleaning:** seven documented steps, each printing row counts before and after and checked with assertions (70 across the notebook). Parsed dates; normalized 81 reviews written in decorative Unicode fonts; confirmed 0 exact duplicates (354 repeated texts kept as different users); replaced the 99.65%-empty reply column with a flag; log-transformed skewed thumbs-up counts; added length, emoji, and non-English flags for subgroup tests. No rows were removed.
2. **AI step:** **Claude Haiku 5.5** (`claude-haiku-5-5`), effort `low`. Temperature is not set because current Claude models do not accept it. Reproducibility comes from saving every output to CSV with the model ID and prompt version, plus a 3-run consistency test. The prompt (built from our codebook and tie-break rules) asks Claude to read the review text only (star rating hidden) and return structured JSON: one of 10 labels, a 0–1 confidence, and a supporting quote. A 20-review pilot came first, then the full run on a 2,999-review sample stratified by month × rating. A $2.00 budget cap blocked any over-budget run.
3. **Analysis:** issue prevalence with 95% CIs; a priority score (share of 1–2★ complaints × thumbs-up agreement) with bootstrap ranking; monthly trends with two-proportion tests.
4. **Evaluation:** both partners independently labeled the same 100 reviews (human κ = 0.73). Claude, the keyword baseline, and a majority-class floor were scored against **each partner's labels separately**: accuracy, per-class and macro precision/recall/F1, Cohen's kappa, confusion matrices, 95% bootstrap CIs (1,000 resamples), MDD (mean rater disagreement + 1 SD), 3-run consistency, and cost.
5. **Recommendation and governance:** prioritize app reliability; investigate UI changes vs. ads. Use Claude as a supervised monitor. Every governance threshold (confidence < 0.70 review, weekly 50-review audit, macro-F1 ≥ 0.67, 1.25× subgroup disparity, +5-pt drift alarm, ≥ 95% consistency, cost and PII caps) is back-tested on our data in notebook Section 10.

### Issue taxonomy (one primary label per review)

| Label | Covers |
|---|---|
| `ads` | Ad frequency, length, loudness, ads interrupting music |
| `playback_issues` | The *audio* misbehaves: music stops/pauses/skips on its own, won't play, Bluetooth/car/Connect, sound quality |
| `app_stability` | The *app* fails: crashes, freezes, slow, won't open, battery drain, error messages, visual glitches |
| `ui_update_change` | Redesigns or updates that removed or moved features; interface feature requests |
| `free_tier_limits` | Shuffle-only, skip limits, can't pick songs, premium paywall or price |
| `offline_download` | Downloads lost, can't download, offline mode not working |
| `account_login` | Logged out, can't sign in, family plans, billing/subscription problems |
| `content_recommend` | Missing songs/artists/podcasts, bad recommendations, unwanted songs added |
| `praise_no_issue` | Positive or neutral review with no specific complaint |
| `other` | Negative but too vague to act on, off-topic, or unreadable |

## Repo contents

- `notebooks/spotify_review_triage.ipynb`: main analysis, runs top to bottom (cleaning → sampling → baseline → Claude labeling → evaluation → prioritization → governance back-tests)
- `notebooks/spotify_review_triage.html`: rendered copy of the notebook with all outputs
- `data/raw/reviews.csv`: original Kaggle data, never edited
- `data/processed/reviews_clean.csv`: cleaned data (all 61,594 reviews)
- `data/processed/analysis_sample.csv`: the 2,999-review stratified sample (`in_gold` marks the 100 gold reviews)
- `data/processed/claude_labels_haiku.csv`: **saved Claude outputs** (label, confidence, evidence, tokens, cost, model, prompt version)
- `data/processed/claude_pilot_haiku.csv`, `claude_consistency_run2.csv`, `claude_consistency_run3.csv`: pilot and consistency-test outputs
- `data/processed/baseline_labels.csv`: keyword-baseline labels
- `data/labels/`: both partners' hand labels for the 100 gold reviews
- `docs/codebook.md`: label definitions and tie-break rules used by humans and Claude
- `outputs/figures/`, `outputs/tables/`: charts and metric tables used in the write-ups

## How to run it

1. Clone this repo and install packages:
   `pip install -r requirements.txt`
2. (Only needed to re-run the AI step) Set an environment variable named `ANTHROPIC_API_KEY`, or copy `.env.example` to `.env` and add your key.
3. Open `notebooks/spotify_review_triage.ipynb` and run all cells (about 1–2 minutes).
   By default (`RUN_API = False`), the notebook reads saved Claude outputs from `data/processed/` and **does not call the API**.

## Limitations

- **Small gold set:** accuracy is measured on 100 reviews (CIs ≈ ±8 points), and `free_tier_limits`, `offline_download`, and `other` have too few examples (< 5) to judge per-class performance.
- **Humans disagree too:** the two of us agreed on only 77% of reviews. App crashes vs. music stopping is the fuzziest boundary for both people and Claude, so we report them together as one reliability workstream.
- **Low-confidence review is costly:** the < 0.70 human-review rule flags 38.5% of reviews (~21 hours/month at ~9,800 reviews/month). Confidence scores cluster at 0.6, which makes the threshold sensitive.
- **Scope:** Google Play (Android) reviews in mostly English from Jan–Jul 2022; results may not reflect iOS users or current app versions. One primary label per review simplifies reviews that raise several issues.
- **The #3 priority is a judgment call:** it depends on whether thumbs-up agreement or raw volume is weighted more.

## AI use

We used Claude (via Claude Code) to help plan the workflow, write and debug code, and draft documentation, and ChatGPT to draft initial governance rules, which we critiqued and rewrote. We verified the work with assertions and test cases, by hand-labeling the gold set ourselves, by back-testing every governance threshold, and by checking each output against the notebook. Our individual AI use logs are submitted on Canvas.
