# Spotify App Reviews: AI-Assisted Issue Prioritization

Morgan Henderson and Subrahmanyam Kodiguddu
INFO 4340, Fall 2026, University of Denver

## Overview

Which product problems should Spotify's app team fix first, and can an AI label app reviews reliably enough to track this every week? Star ratings show how unhappy users are but not why, and 61,594 reviews are too many to read by hand. At the center of our workflow, Claude reads each review and assigns it one issue category (e.g., ads, playback problems, app crashes). We test those labels against human labels and a non-AI keyword baseline, then rank issues by how often they appear, how negative those reviews are, and how many users agree with them.

## Key findings

*TBD: filled in once the evaluation is complete.*

- [Claude vs. keyword baseline: macro-F1 with 95% bootstrap CI]
- [Human–human and human–AI agreement (Cohen's kappa)]
- [Top 3 issues to fix, with share of negative reviews]
- [Cost per 1,000 reviews labeled]

## Data

- **Source:** [Spotify App Reviews 2022](https://www.kaggle.com/datasets/mfaaris/spotify-app-reviews-2022/data) (public Kaggle dataset of Google Play Store reviews)
- **Size:** 61,594 reviews, January–July 2022
- **Unit of observation:** one user review (text, 1–5 star rating, thumbs-up count, timestamp, and Spotify's reply if any)
- **Access:** included in `data/raw/reviews.csv`. No usernames or personal identifiers are in the data.

## Workflow

1. **Cleaning:** *TBD.* Validate the raw file with assertions, normalize text, and document every decision in a decision log.
2. **AI step:** *TBD (model and settings finalized after the pilot).* Plan: Claude Haiku 5.5 labels a stratified sample of ~3,000 reviews into one of the 10 issue categories below, using structured JSON output. Claude Sonnet 5.5 labels the gold set only, as a model comparison. Current Claude models do not accept a temperature setting, so reproducibility comes from saving every output to CSV and logging the exact model ID and prompt.
3. **Analysis:** issue prevalence, monthly trends, and a priority score (volume × severity × user agreement).
4. **Evaluation:** precision/recall/F1 with bootstrap CIs, Cohen's kappa (human–human and human–AI), minimum detectable difference, and cost, against a hand-labeled gold set and a keyword baseline.
5. **Recommendation:** *TBD.* Which issues to fix first, plus governance rules for when a human must review the AI's labels.

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

- `data/raw/reviews.csv`: original Kaggle data, never edited
- `data/processed/`: cleaned sample and saved Claude outputs (so the notebook runs without the API)
- `data/labels/`: hand-labeled gold set from both partners
- `notebooks/`: main analysis notebook (*coming*)
- `docs/`: label codebook, written summary, governance memo, slides (*coming*)
- `outputs/figures/`: charts used in the summary and slides

## How to run it

1. Clone this repo and install packages:
   `pip install -r requirements.txt`
2. (Only needed to re-run the AI step) Copy `.env.example` to `.env` and add your Anthropic API key, or set an environment variable named `ANTHROPIC_API_KEY`.
3. Open the notebook in `notebooks/` and run all cells.
   By default (`RUN_API = False`), the notebook reads saved Claude outputs from `data/processed/` and **does not call the API**.

## Limitations

*TBD: finalized after evaluation.* Known so far:

- Reviews come from Google Play only (Android), Jan–Jul 2022, so they may not reflect iOS users or current app versions.
- Each review gets one primary label, so reviews that raise several issues are simplified.

## AI use

We used Claude (via Claude Code) to help plan the workflow, write and debug code, and draft documentation. We verified its work with assertions, by hand-labeling a gold set ourselves, and by reviewing each output. Our individual AI use logs are submitted on Canvas.
