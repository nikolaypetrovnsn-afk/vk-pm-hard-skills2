# VK Feed — Hypotheses for Growing timespent/DAU

## Overview

Structured hypothesis map for growing timespent/DAU in VK Feed. Covers both components of the target metric — sessions_per_DAU (frequency) and avg_session_length (depth) — with proxy metrics, guard metrics, experiment designs, and prioritization.

**Product:** VK Feed (main surface)

**NSM:** timespent/DAU

**Decomposition:** `timespent/DAU = sessions_per_DAU × avg_session_length`

**Scope:** Algorithm + UI/UX + cross-product (clips). Push notifications excluded (already overloaded).

---

## Proxy Metrics Map

### sessions_per_DAU (frequency)
- **push_open_rate** — share of DAU entering via push
- **organic_return_rate** — share of DAU returning organically
- **inter_session_gap** — average time between sessions
- **daily_opens** — average feed opens per day

### avg_session_length (depth)
- **scroll_depth** — average posts viewed per session
- **dwell_time_per_post** — average time spent per post
- **content_completion_rate** — consumption completeness (for video/articles)
- **session_end_trigger** — why session ended (scrolled to end / navigated away / closed)

### Guard Metrics
- **unlike_rate / hide_rate** — negative reactions
- **feed_exit_rate** — exits after 1-2 posts
- **D7/D30 retention** — don't sacrifice retention for timespent
- **ad_fatigue_rate / ad_ctr** — ad effectiveness must not degrade
- **privacy_complaint_rate** — for social features (A6)

---

## Full Hypothesis Map

| # | Hypothesis | Direction | ICE | Target Proxy | Cohort |
|---|-----------|-----------|-----|-------------|--------|
| Б3 | Freshness-boost | Algo | 448 | sessions_per_DAU | subscription-driven |
| В2 | Clip carousel | UI | 343 | clips_per_session | reco-driven |
| Б5 | Content contrast | Algo | 294 | scroll_depth + session_length | reco-driven |
| А6 | Friend badges | UI | 294 | sessions_per_DAU + session_length | social-driven |
| А2 | Post series | UI | 294 | content_completion_rate | reco-driven |
| Б1 | Dwell time in model | Algo | 288 | dwell_time + session_length | all |
| А3 | Interactive formats | UI | 240 | interaction_rate | newcomers |
| В1 | Seamless clips | UI+Algo | 240 | session_length | clip consumers |
| В3 | Next-clip bridge | Cross-product | 200 | total_timespent | clip consumers |
| А1 | Scroll checkpoints | UI | 168 | scroll_depth | whales |
| Б2 | Niche clusters | Algo | 120 | niche_affinity | newcomers |

**Excluded:** А4 (push triggers — already overloaded), А5 (comments — already shown in feed, not our domain), Б4 (re-rank on return — already implemented), В2 (clip carousel — too complex in design)

**Top-4 for deep dive:** Б3, А6, Б5, Б1

---

## Deep Dive 1: Б3 — Freshness-boost

### Core Hypothesis

Fresh content from subscriptions receives a temporary ranking boost for the first 30 minutes after publication. Users see "hot" content from their subscriptions more often → return more frequently to check updates → sessions_per_DAU increases.

### Mechanism

- Ranking score multiplied by `f(age) = max(0, 1 - age/30min)` for subscription content
- Linear decay: at publish = 1x, at 15 min = 0.5x, at 30 min = 0x
- Applied only to subscription content (not recommendation channel)
- User sees "new from subscriptions" at the top of feed on entry

### Impact Decomposition

```
timespent/DAU = sessions_per_DAU × avg_session_length

Б3 → sessions_per_DAU ↑
  └─ organic_return_rate ↑ (check "what's new" more often)
  └─ inter_session_gap ↓ (less time between sessions)
  └─ fresh_content_ctr ↑ (new content gets more clicks)
```

### Proxy Metrics

| Metric | Measures | Expected Effect |
|--------|----------|----------------|
| organic_return_rate | % DAU returning organically | +3-5pp |
| inter_session_gap | Average interval between sessions | -10-15% |
| fresh_content_ctr | CTR of content <30 min old | +15-25% |
| sessions_per_DAU | Average sessions per day | +5-8% |
| subscription_content_share | Share of subscription content in feed | +3-5pp |

### Guard Metrics

| Metric | Threshold | Reason |
|--------|-----------|--------|
| overall_ctr | not below -1pp | Boost may displace relevant older content |
| hide_rate | not above +0.5pp | Fresh ≠ good, users may hide |
| recommendation_content_ctr | not below -2pp | Don't displace quality reco content |
| D7 retention | not below -0.5pp | Don't reduce retention with bad content |

### Experiment Design

- **Split:** by user_id, 50/50
- **Duration:** 14 days (need to capture delayed effect on sessions_per_DAU)
- **MDE:** +3% sessions_per_DAU at α=0.05, β=0.2
- **Sample size:** ~200K users per group (at DAU ~60M, σ sessions_per_DAU ~1.2)
- **Ramp:** 1% → 5% → 50% → 100%

### Risks

1. **Fresh but weak** — boost gives visibility to low-quality content → guard: hide_rate
2. **"Empty feed" effect** — users with few subscriptions get no boost → analyze cohort with <20 subscriptions separately
3. **Recommendation cannibalization** — subscription content displaces reco → guard: recommendation_content_ctr

---

## Deep Dive 2: А6 — Friend Badges (Micro-notifications)

### Core Hypothesis

Activity indicators on friend avatars and subscription content cards ("online", "just posted", "discussing now") create a sense of liveness and FOMO, increasing both return frequency and session length.

### Mechanism

- On friend avatar in feed: dot indicator "online" (green) or "just posted" (orange)
- On subscription post card: badge "5 min ago" or "hot discussion" (>N comments in last hour)
- Indicators update in real-time via WebSocket when feed is open
- Visually subtle — 4px dot, does not overlap content
- Shown only on friend content (not communities)

### Impact Decomposition

```
timespent/DAU = sessions_per_DAU × avg_session_length

А6 → BOTH components:
  sessions_per_DAU ↑
    └─ organic_return_rate ↑ (FOMO — "what's happening right now")
    └─ inter_session_gap ↓
  avg_session_length ↑
    └─ friend_content_ctr ↑ (badged content gets more clicks)
    └─ social_session_rate ↑ (sessions with social content are longer)
```

### Proxy Metrics

| Metric | Measures | Expected Effect |
|--------|----------|----------------|
| friend_content_ctr | CTR of friend posts with badge vs without | +10-20% |
| organic_return_rate | Organic returns to feed | +2-4pp |
| social_session_rate | Share of sessions with 1+ friend interaction | +5pp |
| inter_session_gap | Interval between sessions | -8-12% |
| sessions_per_DAU | Sessions per day | +3-5% |

### Guard Metrics

| Metric | Threshold | Reason |
|--------|-----------|--------|
| privacy_complaint_rate | not above +0.1pp | Users may not want their activity visible |
| feed_exit_rate | not above +1pp | Badges shouldn't irritate and drive away |
| non_friend_session_length | neutral | Don't worsen experience for users with few friends |
| D7 retention | neutral or ↑ | Don't harm retention |

### Experiment Design

- **Split:** by user_id, 50/50
- **Duration:** 14 days (FOMO effect may be delayed)
- **MDE:** +2% sessions_per_DAU at α=0.05, β=0.2
- **Ramp:** 5% → 25% → 50% → 100% (start at 5% due to privacy risk)
- **Cohort slice:** measure separately for users with <10 friends vs >50 friends

### Risks

1. **Privacy** — "people can see I'm online" may concern users. Must respect existing visibility settings
2. **Badge blindness** — if badges are everywhere, they stop working. Limit to friend content only
3. **No friends** — for users without friends, badges don't work → no effect. OK — hypothesis targets social-driven cohort

---

## Deep Dive 3: Б5 — Content Contrast (Format Diversity)

### Core Hypothesis

Adding a diversity penalty in ranking for showing same-format content consecutively reduces "feed blindness" and fatigue. Alternating formats (post → clip → poll → article) leads to deeper scrolling and longer sessions.

### Mechanism

- Diversity penalty added to ranking: if 2 previous posts in feed are same type, score of next same-type post is multiplied by `1 - diversity_penalty`
- `diversity_penalty` = 0.3 (tuned offline) — third text post in a row gets -30% to score
- Content types: text post, photo post, clip, article, poll, community repost, ad
- Exception: ads don't receive penalty and don't count in consecutive-type calculation

### Impact Decomposition

```
timespent/DAU = sessions_per_DAU × avg_session_length

Б5 → avg_session_length ↑
  └─ scroll_depth ↑ (scroll deeper without fatigue)
  └─ skip_rate ↓ (fewer posts scrolled past without looking)
  └─ dwell_time_per_post ↑ (variety = more interest)
```

### Proxy Metrics

| Metric | Measures | Expected Effect |
|--------|----------|----------------|
| scroll_depth | Average posts viewed before session end | +10-15% |
| skip_rate | Share of posts scrolled past in <0.5 sec | -5-10pp |
| format_diversity_per_session | Number of unique formats in session | +1-2 formats |
| dwell_time_per_post | Average time per post | +5-10% |
| avg_session_length | Session length | +3-5% |

### Guard Metrics

| Metric | Threshold | Reason |
|--------|-----------|--------|
| overall_ctr | not below -1.5pp | Diversity may show less relevant content |
| unlike_rate | not above +0.3pp | Users may resent "forced" format |
| ad_ctr | not below -1pp | Ad effectiveness must not suffer |
| D7 retention | neutral or ↑ | Don't worsen retention |

### Experiment Design

- **Split:** by user_id, 50/50
- **Duration:** 14 days
- **MDE:** +3% avg_session_length at α=0.05, β=0.2
- **Ramp:** 1% → 5% → 50% → 100%
- **Pre-step:** offline evaluation on historical data — tune `diversity_penalty` (0.1, 0.2, 0.3, 0.5)

### Risks

1. **Forced format** — user wants only clips but gets articles → need segmentation by format preference
2. **Relevance reduction** — diversity penalty may displace best content → balance via penalty tuning
3. **Cold start for narrow profiles** — if user has no format diversity in subscriptions, nothing to alternate → check on narrow profiles

---

## Deep Dive 4: Б1 — Dwell Time in Ranking Model

### Core Hypothesis

Adding dwell_time_per_post (normalized by content type) as a target signal in the ranking model makes the model prefer content that users "fall into" — not just click but spend time on. This increases avg_session_length.

### Mechanism

- Training target becomes weighted sum: `target = α × CTR + β × dwell_time_norm + γ × like_rate`
- `dwell_time_norm` = dwell_time / median_dwell_time(content_type) — normalization: 10 sec on clip ≠ 10 sec on article
- Initial weights: α=0.5, β=0.3, γ=0.2 (tuned offline)
- Model retrained with new target, then A/B tested against current production model

### Impact Decomposition

```
timespent/DAU = sessions_per_DAU × avg_session_length

Б1 → avg_session_length ↑
  └─ dwell_time_per_post ↑ (model shows "engaging" content)
  └─ content_completion_rate ↑ (longer per post = more likely to finish)
  └─ scroll_depth ↑ (better content → scroll further)

Secondary effect on sessions_per_DAU:
  └─ better content → return rate ↑ (quality sessions form habit)
```

### Proxy Metrics

| Metric | Measures | Expected Effect |
|--------|----------|----------------|
| dwell_time_per_post | Average time per post | +10-15% |
| content_completion_rate | Consumption completeness | +5-10% |
| scroll_depth | Posts viewed per session | +8-12% |
| avg_session_length | Session length | +5-8% |
| long_dwell_share | Share of posts with dwell >2×median | +5pp |

### Guard Metrics

| Metric | Threshold | Reason |
|--------|-----------|--------|
| overall_ctr | not below -2pp | Dwell optimization may shift CTR |
| unlike_rate | not above +0.5pp | "Engaging" may be clickbait |
| hide_rate | not above +0.5pp | Users don't hide "engaging but useless" content |
| ad_ctr | not below -1pp | Ad CTR doesn't drop from context shift |
| D7 retention | neutral or ↑ | Don't worsen retention |
| diversity_score | not below -5% | Model doesn't collapse into one "engaging" content type. diversity_score = entropy of content type distribution per session |

### Experiment Design

- **Split:** cluster-based randomization (due to network effects — one user's content is shown to others)
- **Duration:** 21 days (algorithmic effects accumulate slowly + model stabilization period)
- **MDE:** +3% avg_session_length at α=0.05, β=0.2
- **Ramp:** 1% → 5% → 25% → 50% → 100% (conservative due to algorithmic risk)
- **Pre-steps:**
  1. Offline evaluation: A/B replay on historical data, tune weights α/β/γ
  2. Correlation analysis: which content types have high dwell but low CTR — these benefit most from new model
  3. Pipeline check: is dwell_time_per_post already logged? If not, add logging first

### Risks

1. **Clickbait loop** — "engaging" content is often low-quality. Model promotes it → guard: unlike_rate. Solution: add explicit signals (like, share) as counterweight
2. **Long test cycle** — algorithmic A/B tests take longer than UI tests. Need minimum 21 days
3. **Format bias** — articles and videos are longer by nature. Normalization addresses this, but verify short formats (memes, quotes) aren't pushed out
4. **Network effects** — ranking changes affect authors (fewer impressions → less motivation). Cluster randomization partially addresses this

---

## Interaction Map

```
                    timespent/DAU
                   /              \
      sessions_per_DAU       avg_session_length
         /      \               /          \
    ┌───┴───┐  ┌─┴──┐    ┌───┴────┐    ┌──┴─────┐
    │ Б3    │  │ А6 │    │ Б5     │    │ Б1     │
    │Fresh- │  │Bad- │    │Con-   │    │ Dwell  │
    │ness   │  │ges │    │trast  │    │ time   │
    └───────┘  └────┘    └────────┘    └────────┘

    Б3: ↑ frequency (subscription users)
    А6: ↑ frequency + ↑ length (social users)
    Б5: ↑ depth (reco-driven users)
    Б1: ↑ length (all users)
```

All four hypotheses are orthogonal — can be tested in parallel on different splits and effects can be summed.

### Recommended Testing Order

1. **Б3 (Freshness-boost)** — highest ICE, moderate complexity, fast to implement and test
2. **Б5 (Content contrast)** — low complexity, quick win, can be tuned offline first
3. **А6 (Friend badges)** — medium complexity, privacy risk requires careful ramp
4. **Б1 (Dwell time in model)** — highest impact potential but longest cycle, start offline work in parallel with Б3/Б5 testing

### Combined Effect Estimation

If all four hypotheses deliver expected effects:

- Б3: +5-8% sessions_per_DAU
- А6: +3-5% sessions_per_DAU, +2-3% session_length
- Б5: +3-5% session_length
- Б1: +5-8% session_length

Combined (conservative, accounting for overlap): **+8-12% sessions_per_DAU × +8-12% session_length = +17-26% timespent/DAU**

Realistically, not all will deliver full effect — target **+10-15% timespent/DAU** with 2-3 successful hypotheses.
