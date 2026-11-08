# Labeling Codebook: Spotify Review Issue Taxonomy (v1)

Give every review **exactly one primary label**: the issue a product team would need to act on.
Label from the **review text only**. The star rating is hidden on purpose, so human and AI labelers see the same information.

## Labels

| Label | Use when the review's main point is... | Examples |
|---|---|---|
| `ads` | Advertisements: too many, too long, too loud, repetitive, interrupting music | "Too many ads", "an ad after every song" |
| `playback_issues` | **The audio** misbehaves: music stops, pauses or skips by itself, won't play, stops in the background, Bluetooth/car/Connect/smartwatch playback, sound quality or volume | "music stops when my screen locks", "won't connect to my car" |
| `app_stability` | **The app itself** fails: crashes, freezes, won't open, slow or laggy, black screen, battery or data drain, error messages, visual glitches | "app crashes on launch", "so laggy since the update" |
| `ui_update_change` | Design or layout changes, or features removed, moved or hidden by an update, plus requests for interface features | "bring back the old layout", "can't see full albums anymore", "need a search bar in playlists" |
| `free_tier_limits` | Restrictions on free users or the premium paywall: shuffle-only, skip limits, can't choose songs, premium too expensive | "I can't pick the song I want", "only 6 skips" |
| `offline_download` | Downloads disappearing, can't download, offline mode not working | "my downloaded songs keep vanishing" |
| `account_login` | Sign-in, logout, account access, password, family/duo plans, payment or billing errors, subscription not recognized | "keeps logging me out", "charged twice" |
| `content_recommend` | Catalog or content: missing songs, artists, lyrics or podcasts; recommendations; autoplay or Smart Shuffle adding unwanted songs; complaints about content or platform policy | "adds random songs to my playlist", "removed my favorite artist" |
| `praise_no_issue` | Positive or neutral review with **no** specific complaint | "Best music app ever!" |
| `other` | Negative but too vague to act on, off-topic, or not readable | "worst app", "not working properly" |

## Tie-break rules (apply in order)

1. **A complaint beats praise.** "Love it but too many ads" → `ads`, not `praise_no_issue`.
2. **Several complaints → pick the one the reviewer emphasizes most** (most words, or framed as the main reason for the rating). If they're equal, pick the one mentioned first.
3. **Audio vs. app:** if the *music* stops or fails while the app keeps working → `playback_issues`. If the *app* crashes or freezes, which stops the music → `app_stability`.
4. **Ads vs. free tier:** about the ads themselves → `ads`. About what free users *can't do* (shuffle, skips, choosing songs) → `free_tier_limits`.
5. **Vague negative → `other`.** Only pick a specific label if the review names or clearly describes that problem.
6. **Non-English:** label it if you can understand it, otherwise `other`.

## Process for the gold set

- Each partner labels the **same** reviews **independently**. Don't discuss them until both files are done, because the human–human kappa is only valid if you label independently.
- Use the `notes` column for anything you weren't sure about. Those notes become evidence for your AI use log and the governance memo.
- After both partners finish, the notebook computes kappa. Then the two of you resolve disagreements together to make the final `gold_label`.
