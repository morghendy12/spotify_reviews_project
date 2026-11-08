# Spotify App Reviews: AI-Assisted Issue Prioritization

**Question:** Which product problems should Spotify's app team fix first, and can Claude label app reviews reliably enough to track this every week?

**Data:** [Spotify App Reviews 2022](https://www.kaggle.com/datasets/mfaaris/spotify-app-reviews-2022/data): 61,594 Google Play reviews, Jan–Jul 2022 (`Time_submitted`, `Review`, `Rating`, `Total_thumbsup`, `Reply`).

## Workflow

```
raw reviews → cleaning + validation → stratified sample → Claude issue labels (cached to CSV)
           → non-AI baseline (TF-IDF + logistic regression)
           → evaluation vs. human gold set (P/R/F1, bootstrap CIs, kappa, MDD, cost)
           → issue prioritization (volume × severity × thumbs-up) → governance rules
```

## Issue taxonomy (one primary label per review)

| Label | Covers |
|---|---|
| `ads` | Ad frequency, length, loudness, ads interrupting playback |
| `playback_issues` | Music stops/pauses/skips on its own, won't play, Bluetooth/car/Connect, audio quality |
| `app_stability` | Crashes, freezes, slow, won't open, battery drain, visual glitches |
| `ui_update_change` | Complaints about a redesign or update that removed/moved features |
| `free_tier_limits` | Shuffle-only, skip limits, can't pick songs, premium paywall |
| `offline_download` | Downloads lost, offline mode not working |
| `account_login` | Logged out, can't sign in, billing/subscription problems |
| `content_recommend` | Missing songs/podcasts, bad recommendations, unwanted songs added |
| `praise_no_issue` | Positive review with no actionable complaint |
| `other` | Off-topic, unclear, or non-English |

## Repo layout

```
data/raw/            original Kaggle CSV (never edited)
data/processed/      cleaned data + analysis sample
data/labels/         human gold-set labels (both partners)
outputs/llm/         cached Claude outputs: the notebook reads these instead of re-calling the API
outputs/figures/     charts used in the summary and slides
notebooks/           the reproducible notebook (.ipynb + rendered .html)
docs/                written summary, governance memo, slides (PDF)
docs/ai_use_logs/    individual AI use logs
```

## Reproducing

1. `pip install -r requirements.txt`
2. Copy `.env.example` to `.env` and add your Anthropic API key. **This is only needed if you set `RUN_API = True`.**
3. Run `notebooks/` top to bottom. By default `RUN_API = False`, so the notebook reads cached Claude outputs from `outputs/llm/` and makes **no** API calls.
