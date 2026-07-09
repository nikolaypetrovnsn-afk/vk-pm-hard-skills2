# VK Feed — Geo-Signal Discovery: Structured Research Plan

## Overview

Structured discovery process for evaluating geo as a ranking signal in VK Feed. Goal: determine whether geographic proximity between viewer and content meaningfully improves recommendation quality, and identify the highest-leverage cohort-signal combinations.

**Product:** VK Feed

**NSM:** timespent/DAU

**Research question:** Does geo-proximity between viewer and content correlate with higher engagement, and can we leverage it to improve feed ranking?

**Scope:** All geo-signal sources (community settings, YClients, content geo-tags, IP-based viewer location). Excludes: push notifications, Maps product.

---

## Framework

Discovery proceeds in three layers, each answering a distinct question:

| Layer | Question | Method |
|-------|----------|--------|
| **Inventory** | How much geo data do we have and how good is it? | Analytics (logs, registries) |
| **Behavior** | Do users respond better to geo-matched content? Is there a signal? | Analytics + user surveys |
| **Leverage** | If we boost geo-matched content in ranking, what's the effect? | Offline evaluation + A/B |

Each layer depends on the previous. If inventory is sparse → no leverage. If behavior shows no signal → signal is dead.

**Go/no-go criteria** at the end of Layer 2 determine whether to proceed to Layer 3.

---

## Layer 1: Inventory — What We Have

### 1.1 Geo-Signal Map by Source

| Source | Where it lives | Completeness | Accuracy | Update frequency |
|--------|---------------|-------------|----------|-----------------|
| Geo-local community flag | Community settings | ? | High (explicit) | Rare |
| Address (city) in community settings | Community settings | ? | High | Rare |
| YClients coordinates | YClients integration | ? | Very high (GPS) | On address change |
| Geo-tag on post/clip | Content | ? | Medium (can be fake/spoofed) | On publication |
| IP-based viewer geo | Request logs | High | Medium (VPN, iOS permissions) | Per request |

### 1.2 Analytics Tasks — Inventory

#### З1: Geo-flagged community inventory

Build a cross-tabulation matrix of community geo-markers:

```
                        | geo-local flag | NOT geo-local | total
address specified       |      A         |       B       | A+B
address NOT specified   |      C         |       D       | C+D
total                   |     A+C        |      B+D      |
```

Separately: how many YClients-enabled communities fall into each cell.

**Key questions:**
- How many communities are geo-local? (A+C)
- How many have an address? (A+B)
- What's the intersection? (A)
- How many YClients communities overlap with each cell?
- What % of total active communities does each cell represent?

**Breakdown by author type** (overlaid on the matrix):
- Government communities (admin, ministries, schools, hospitals)
- Business communities (shops, restaurants, salons, fitness)
- Business with YClients
- Content communities with geo (media, blogs, events)

#### З2: Geo-tags on content

- How many posts/clips per day are published with a geo-tag?
- What % of all feed content does this represent?
- How are geo-tags distributed by city? (Top-20 cities + long tail)
- What % of geo-tags match the community's address/locality? (check for fake/spoofed tags)
- Geo-tag adoption rate: what % of posts from geo-local communities include a geo-tag?

#### З3: IP-geo quality

- For users who have a city specified in their profile: how often does IP-geo match? (% match by Russian cities, by countries)
- Break down by platform: iOS vs Android (iOS has worse permission-based accuracy)
- VPN traffic share: IP-geo ≠ profile city AND city ≠ Moscow/SPb (i.e. clearly not relocation)
- How does IP-geo accuracy vary by time of day? (proxy for mobile vs desktop usage)

#### З4: Coverage — how much feed content is geo-identifiable

Percentage of impressions/views from:
- Geo-local communities
- Communities with a specified address
- Posts/clips with geo-tags
- YClients communities
- **Aggregate: what % of feed impressions have ANY geo-signal we can match against viewer location?**

This is the critical number — it defines the upper bound of geo-boost's potential impact.

---

## Layer 2: Behavior — Is There a Signal

### 2.1 Analytics Tasks — Behavior

#### З5: Geo-match → engagement

For each impression in feed: does viewer geo match content geo (community/tag)?

Compare engagement metrics:
- CTR of geo-matched posts vs non-matched
- Dwell time of geo-matched vs non-matched
- Like/share rate of geo-matched vs non-matched
- Completion rate (for clips) of geo-matched vs non-matched

**Critical control:** normalize by content quality. Geo-matched content may just be better content. Control by comparing the SAME content shown to geo-matched vs non-matched viewers.

Break down by content source:
- Geo-local communities
- Communities with address
- YClients communities
- Posts with geo-tag

#### З6: Geo-signal by viewer cohorts

- By city tier: Moscow, SPb, 500K+, 100-500K, <100K
- By user type: "local" (IP = profile) vs "displaced" (IP ≠ profile)
- By app tenure: newcomers (0-7 days) vs established
- By age group: <25, 25-35, 35+

**Hypothesis:** Geo-signal is stronger in smaller cities (less content overall → local relevance higher) and for "displaced" users (new city → need local info).

#### З7: Geo-signal by author type

Break communities into:
- **Government** (administrations, ministries, schools, hospitals) — geo is critical (school in another city = irrelevant)
- **Business** (shops, restaurants, salons, fitness) — geo is important (salon booking in another city = useless)
- **Business with YClients** — geo is critical by definition (booking service)
- **Content with geo** (media, blogs, events) — geo is nice-to-have (reading about Moscow restaurant from Novosibirsk is OK)

For each type: CTR/dwell/like at geo-match vs no-match.

**Hypothesis:** Government and YClients have the strongest geo-match signal. Content communities have the weakest.

#### З8: Geo-cannibalization

If we show more geo-relevant content, does it displace quality non-geo content?

- Correlation between geo-content share in session and overall engagement
- Is there a threshold where "too much" geo-content reduces timespent?
- By cohort: does the threshold differ for small vs large cities?

This sets the upper bound on how aggressively we can boost geo-content.

### 2.2 User Surveys

#### О1: Value of local content (quantitative signal)

**Format:** In-app survey (n=2000-5000), shown after 3+ minutes in feed

**Questions:**

1. "Have you noticed posts about events/places in your city in the feed?" (Often / Sometimes / No / Don't remember)

2. "Would you like to see more content about your city in the feed?" (Yes / No / Don't care)

3. "What local content interests you most?" (multi-select):
   - City news and events
   - Restaurants and cafes
   - Sales and promotions from local shops
   - Events (concerts, exhibitions)
   - Jobs/services
   - Not interested in local content

4. "Have you ever seen content from another city that was irrelevant to you? How often?" (Often / Sometimes / Rarely / Never)

**Analysis:** Break responses by cohort (city tier, age, DAU segment). Identify which cohorts value local content most.

#### О2: Pain points with non-relevant geo-content (qualitative signal)

**Format:** Open-ended question (in survey or feedback analysis)

**Question:** "Have you come across content from another city in the feed that was irrelevant? What kind of content was it?"

**Goal:** Determine if there's a "negative signal" — if users don't complain, the problem may be overstated.

**Secondary method:** Analyze hide/unlike reasons for geo-mismatched content (if reason tags exist).

#### О3: Local-intent search behavior (behavioral proxy)

**Format:** Behavioral analysis (no survey needed)

- What % of DAU uses search with local intent? (queries like "cafe moscow", "salon spb", "events kazan")
- What % of DAU visits the "Recommendations" tab → city filter?
- What % of DAU follows geo-local communities?
- Cross-reference: do users who search locally also engage more with geo-matched feed content?

This is a proxy for demand for geo-content in the feed.

---

## Layer 3: Leverage — What We Do With the Signal

### 3.1 Approaches to Using Geo in Ranking

| Approach | Mechanism | Complexity | Hypothesis |
|----------|-----------|------------|------------|
| A. Geo-boost in ranking | When viewer geo matches content geo → +Δ to ranking score | Low | Local content is more relevant |
| B. Geo-slot in feed | Reserve 1-2 feed positions for geo-content | Medium | Guaranteed local content exposure |
| C. Geo-topic cluster | Treat "local feed" as a separate topic cluster in the model | High | Local content is better consumed as a cluster |
| D. Geo-penalty (negative) | Penalize content from a DIFFERENT city (for gov/YClients) | Low | Non-relevant geo-content is noise |

**Recommended starting point:** A (geo-boost) + D (geo-penalty for gov/YClients). These are the simplest to implement and test, and address both the positive and negative sides of the geo-signal.

### 3.2 Priority Cohorts for Testing

**Where geo-signal is most likely to work:**

1. **Small cities (<100K)** — less content overall, local = relevant, high need
2. **YClients consumers** — they seek local services, geo = conversion
3. **New-to-city users** (IP changed recently) — need "what's around me"
4. **Government content consumers** — municipal news from another city is irrelevant

**Where geo-signal is less likely to work:**

- Moscow/SPb — content is already relevant, geo adds little
- Content communities — geo is not critical for consumption
- Users with VPN — IP-geo is unreliable

### 3.3 Offline Evaluation (Pre-A/B)

#### З9: Offline simulation of geo-boost

- Take impression logs for 30 days
- For each impression: does viewer geo match content geo?
- Simulate: if we added +X% to score for geo-matched content, how would it change:
  - NDCG of the feed
  - Average CTR
  - Share of geo-content in top-K positions
  - Share of geo-content per session
- Sweep X = 5%, 10%, 20%, 30%

This gives an upper-bound estimate of the effect without running an A/B test.

**Additionally:** simulate geo-penalty (approach D) for gov/YClients content shown to non-matching geo viewers. What % of impressions would be affected? What content replaces them?

---

## Summary: Task Plan

### Analytics Tasks (Priority Order)

| # | Task | Answers | Est. Time |
|---|------|---------|-----------|
| З1 | Geo-flagged community cross-tab matrix | How much inventory? | 2 days |
| З2 | Geo-tags on content | Volume of geo-content? | 2 days |
| З3 | IP-geo quality | Can we trust viewer geo? | 3 days |
| З4 | Geo-identifiable content coverage | What % of feed can we geo-boost? | 2 days |
| З5 | Geo-match → engagement correlation | Is there a behavioral signal? | 3 days |
| З6 | Geo-signal by viewer cohort | Where is the signal strongest? | 3 days |
| З7 | Geo-signal by author type | For whom is geo most critical? | 3 days |
| З8 | Geo-cannibalization analysis | Does geo displace good content? | 2 days |
| З9 | Offline simulation of geo-boost | Upper bound of effect? | 5 days |

### Surveys

| # | Survey | What we learn | Est. Time |
|---|--------|--------------|-----------|
| О1 | Value of local content | Is there demand? Which content? | 1 week |
| О2 | Pain with non-relevant geo | Is there a negative signal? | 1 week |
| О3 | Local-intent search behavior | Proxy for demand | 2 days |

### Timeline

```
Week 1: З1-З4 (inventory) → Go/no-go checkpoint 1
Week 2: З5-З7 (behavior) + О3 (search proxy) → Go/no-go checkpoint 2
Week 3: З8-З9 (cannibalization + offline sim) + О1-О2 (surveys)
Week 4: Decision → A/B test design or pivot
```

### Go/No-Go Criteria

Proceed to A/B testing if ALL of the following are met:

| Criterion | Threshold | Rationale |
|-----------|-----------|-----------|
| З5: Geo-match CTR lift | ≥ +15% vs no-match | Signal must be meaningfully strong |
| З4: Geo-identifiable content share | ≥ 10% of feed impressions | Must be enough inventory to boost |
| З3: IP-geo accuracy | ≥ 70% match with profile city | Can't boost on unreliable signal |
| О1: Demand for local content | ≥ 30% of respondents want more | Must have user demand |

If any criterion fails → geo-signal is too weak, redirect resources to other hypotheses.

If criteria pass → design A/B test for approaches A (geo-boost) + D (geo-penalty), targeting highest-signal cohorts from З6/З7.
