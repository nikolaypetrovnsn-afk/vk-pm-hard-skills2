# PM Hard Skills Textbook Rebuild — Design Spec

**Date:** 2026-08-05
**Author:** Nikolay Petrov (with Claude)
**Status:** Awaiting user review

## Context

The existing textbook at https://nikolaypetrovnsn-afk.github.io/vk-pm-hard-skills2/index.html is VK-only: 3 blocks (Reference / Cases / Practice) totaling 5 130 lines, all examples drawn from VK products (Feed, Clips, Messenger, Video). It serves VK interview prep well but doesn't help candidates preparing for interviews at other Russian tech companies — Yandex, Ozon, Tinkoff, Avito, Wildberries, or middle-tier (Kontur, Skyeng, 2GIS).

## Goal

Rebuild the textbook to be a practical interview-preparation guide oriented around the Russian tech market, with verified real interview questions from target companies and bidirectional cross-references between cases and the formulas they use.

## Non-goals

- Not targeting global / FAANG companies (Russian market only)
- Not adding interactive features (search, filters) — static HTML
- Not translating to English
- Not including interview questions older than 2023
- Not rewriting Part III formulas wholesale — only where needed (see §Migration)

## Architecture

Three parts. Top-level navigation switches between them.

### Part I. Universal frameworks + cross-company analytical chapter

New content, ~2 500-3 000 lines. Five chapters:

- **1.1 Product Sense framework** — CIRCLES, problem-structuring methods, walked example
- **1.2 Analytical methods** — the cross-company analytical chapter
  - Metrics and metric trees (North Star → input/output)
  - A/B testing methodology, traps, interpretation
  - SQL patterns: cohort, retention, funnel
  - Cross-company examples: how Yandex vs Ozon vs Tinkoff ask about metrics/A/B/SQL, with concrete questions and walkthroughs
- **1.3 Estimation methods** — top-down, bottom-up, Fermi, replacement theory
- **1.4 System design for PM** — trade-offs, scaling, architecture decisions
- **1.5 Behavioral** — STAR, leadership principles, common signals

Each chapter links to Part III for formulas and to Part II chapters for company-specific applications.

### Part II. Company guides

Seven chapters, each self-sufficient for preparation. Order:

1. ВКонтакте (migrated from existing `block2-cases.html`)
2. Яндекс
3. Ozon
4. Тинькофф
5. Avito
6. Wildberries
7. Middle-tier (Контур, Skyeng, 2ГИС — single summary chapter)

#### Chapter template (all 7 chapters follow this)

```
§N.1 Профиль компании (1 экран)
    • Бизнес-модель и основные продукты
    • Культура и что ценят в PM
    • Уровни: middle / senior / lead / principal — отличия

§N.2 Формат собеседования (1-2 экрана)
    • Сколько раундов, какие типы
    • Типичная последовательность
    • На каких раундах отсеивают чаще всего

§N.3 Реальные вопросы с собеседований (центральная часть)
    Сгруппированы по 5 форматам:
    • Product Sense (5-8 вопросов)
    • Analytical (5-8 вопросов)
    • Estimation (3-5 вопросов)
    • System Design (3-5 вопросов)
    • Behavioral (3-5 вопросов)
    Каждый вопрос: формулировка → что проверяют → на что
    обратить внимание → ссылка на метод в Части I
    Фильтр: только 2023-2026, без устаревших

§N.4 Разборы кейсов (2-3 шт.)
    Полные walked examples: формулировка → структура ответа →
    пример ответа (8-12 минут устного разбора) → что часто
    упускают → красные флаги
    Каждый разбор использует конкретные формулы из Части III
    с inline-аннотациями (см. Cross-references)

§N.5 Что отличает [компанию] (1 экран)
    • Специфика expectations
    • Типичные ошибки кандидатов
    • Сигналы, что вы прошли раунд

§N.6 Чек-лист подготовки (1 экран)
    • Что повторить в Части I (конкретные разделы)
    • Что прорешать в Части III (конкретные задачи)
    • Что почитать/посмотреть (публичные материалы)
    • Self-check: 5 вопросов для самопроверки
```

**Volume per chapter:** ~600-900 lines HTML, similar to existing `block2-cases.html` but deeper in §N.3 and §N.4.

**Middle-tier chapter (II.7)** is more compact: three company profiles (Контур / Skyeng / 2ГИС) with 2-3 questions per format each.

### Part III. Reference (existing blocks, revised)

Two files:

- **`part3-reference.html`** (renamed from `block1-reference.html`) — formulas and methods, 18 sections
- **`part3-practice.html`** (renamed from `block3-practice.html`) — 24 practice tasks, unchanged

#### Formula revision pass

Part III is not just renamed. Each formula section gets a revision pass:

- **VK-only examples → cross-company examples** where the formula applies to multiple companies
- **Missing derivations → added** where the formula is stated without showing how it's derived
- **"Используется в" section** added to each formula — back-references to all Part II cases that apply it (see Cross-references)
The intent is targeted improvement, not a full rewrite. Formulas that are already clear and well-connected stay as-is.

## Cross-references (bidirectional, annotated)

This is a hard requirement, not a nice-to-have. Every link between a case and a formula must be bidirectional and annotated with a description of the relationship.

### Case → Formula direction

In Part II case walkthroughs, whenever a formula is used, include an inline annotation:

> Применяем rolling retention (см. [§III.1.1 Rolling Retention](part3-reference.html#rolling-retention)) — выбираем 30-дневное окно вместо 7-дневного, потому что для рекомендательной Ленты короткое окно скрывает отложенное возвращение.

The annotation includes:
- Formula name + link
- Why this variant was chosen
- What the reader should extract from the formula page

### Formula → Case direction

In Part III, each formula gets a "Используется в" section at the bottom:

> **Используется в:**
> - [Лента ВК §II.1.4](part2-vk.html#feed-cohort) — retention рекомендательной vs подписочной Ленты, 30-дневное окно
> - [Яндекс Поиск §II.2.4](part2-yandex.html#search-retention) — DAU-impact оценки для A/B-теста
> - [Ozon Маркетплейс §II.3.4](part2-ozon.html#marketplace-retention) — retention покупателей vs продавцов

Each entry includes the case name, link, and one-line context.

### Bidirectionality rule

If case A links to formula F, then formula F's "Используется в" section must list case A. Verified during the polish phase.

## Research strategy

### Sources (priority order)

1. **Habr** — category "Управление продуктами", tag "собеседования", company blogs (Яндекс / Ozon Tech / Тинькофф / Avito)
2. **Medium** — Russian PM authors, framework breakdowns
3. **YouTube** — Школа менеджеров Яндекса, Ozon Tech, Avito Tech, conference talks (HighLoad, Podlodka, Conf(Rate), PM Conf)
4. **Telegram** (via public search) — channels: PM Online, Product Marketplace, Senior PM, Сеньор-помидор, Дистанция; chats: Product Manager RU, Avito PM
5. **Company product/engineering blogs** — tech.yandex.ru, habr.com/ru/companies/ozontech, /tinkoff, /avito
6. **Glassdoor analogs** — joblab.ru, fantasy.jobs (interview reviews)

### Method

Six parallel agents, one per company (Яндекс / Ozon / Tinkoff / Avito / Wildberries / middle-tier). Each agent:

- Finds 15-25 interview questions for PM positions (middle/senior)
- Distributes across 5 formats (Product Sense / Analytical / Estimation / System Design / Behavioral)
- For each question: formulation + what it tests + source URL + year
- Triangulation: 2+ independent sources mentioning similar question → marked as "частый"
- **Year filter: only 2023-2026. Questions older than 2023 are excluded, not marked.**
- Saves results to `docs/research/{company}.md`

Main agent compiles results into Part II chapters per the template.

### Quality and verification

- Source URL for every question — mandatory
- Year of source next to formulation
- Patterns (3+ sources) listed first in each §N.3
- Unique/rare questions at the end with "был один раз" note
- No fabricated questions — if no source, don't include

### Limitations

- Telegram channels may not be indexed — only publicly searchable content
- Wildberries and middle-tier have less public info — reconstruction from indirect signals (PM talks, hh.ru job descriptions) allowed, marked as "по косвенным сигналам"

## Migration plan

### File changes

```
Current                                  →  New
─────────────────────────────────────────┼────────────────────────────────
index.html (landing)                     →  Updated: 3 parts, new nav
                                         │
block1-reference.html (3 321 lines)      →  part3-reference.html
  18 formula sections                    │  (renamed + revision pass:
                                         │   cross-company examples,
                                         │   derivations, "Используется в"
                                         │   back-references)
                                         │
block2-cases.html (595 lines)            →  part2-vk.html
  4 VK cases                             │  (migrated to company chapter
                                         │   template, expanded with
                                         │   §1/§2/§5/§6 + research)
                                         │
block3-practice.html (1 214 lines)       →  part3-practice.html
  24 tasks                               │  (renamed, unchanged content)
                                         │
styles.css                               →  Extended: new classes for
                                         │  Part I and II sections
                                         │
.github/workflows/pages.yml              →  Unchanged
                                         │
vk-pm-hard-skills-guide.html (600 KB)    →  Deleted (single-page version,
                                         │  superseded by multi-page site)
vk-pm-hard-skills-guide.pdf              →  Deleted
```

### New files

```
part1-frameworks.html       ~2 500-3 000 lines
part2-vk.html               ~700-900 lines (migrated + expanded)
part2-yandex.html           ~700-900 lines
part2-ozon.html             ~700-900 lines
part2-tinkoff.html          ~700-900 lines
part2-avito.html            ~700-900 lines
part2-wildberries.html      ~700-900 lines
part2-middle.html           ~500-700 lines
docs/research/*.md          6 research files (not deployed)
```

## Execution plan

Six phases. After each phase: commit + push to origin/main (GitHub Pages deploys on push).

### Phase 1 — Skeleton

- Rename `block1-reference.html` → `part3-reference.html`
- Rename `block3-practice.html` → `part3-practice.html`
- Update `index.html` for 3-part navigation
- Update sidebar in all files
- Create empty stub files: `part1-frameworks.html`, `part2-*.html`
- Delete obsolete: `vk-pm-hard-skills-guide.html`, `vk-pm-hard-skills-guide.pdf`
- Verify deploy works
- **Commit + push**

### Phase 2 — VK as model chapter

- Migrate `block2-cases.html` content into `part2-vk.html` per company chapter template
- Add §1 Профиль, §2 Формат, §5 Отличия, §6 Чек-лист
- Light research on VK interview questions (own materials are extensive)
- Delete `block2-cases.html`
- **Commit + push**

### Phase 3 — Parallel research

- Six parallel agents: Яндекс / Ozon / Tinkoff / Avito / Wildberries / middle-tier
- Each returns 15-25 questions with sources, year-filtered to 2023-2026
- Save to `docs/research/{company}.md`
- **Commit** (research files don't deploy but live in repo)

### Phase 4 — Part II chapters

- One chapter per session, in order: Яндекс → Ozon → Tinkoff → Avito → Wildberries → middle
- Built from research outputs per the chapter template
- Each case walkthrough (§N.4) includes inline formula annotations
- **Commit + push after each chapter**

### Phase 5 — Part I (frameworks)

- Five chapters with cross-company examples
- Analytical chapter (1.2) is central — includes examples from all 5 companies
- **Commit + push**

### Phase 6 — Part III formula revision

- For each of the 18 formula sections in `part3-reference.html`:
  - Add cross-company examples where currently VK-only
  - Add derivations where the formula is stated without showing how it's derived
  - Add "Используется в" back-reference section listing all Part II cases that apply it
- Sanity-check each rewritten formula against the original
- Cross-check derivations with standard references
- **Commit + push**

### Phase 7 — Polish and verification

- Cross-reference verification pass:
  - Every case → formula link has a matching formula → case link
  - Every annotation explains the relationship (not a bare link)
  - Build a verification checklist from all `part2-*.html` files and walk through it
- Final sidebar navigation
- Mobile check
- Link check (no broken anchors)
- **Commit + push** (final deploy)

## Risks and mitigations

| Risk | Mitigation |
|------|------------|
| Little public info on Wildberries / middle-tier | Reconstruct from hh.ru vacancies, PM talks, press releases; mark as "по косвенным сигналам" |
| Work volume larger than expected | Phases 4 and 5 can iterate: chapter skeleton with 3-5 questions first, expand later |
| Telegram sources not indexed | Focus on Habr / Medium / YouTube / company blogs (publicly accessible) |
| Cross-reference drift (case links to formula, formula doesn't link back) | Verification pass in Phase 7; bidirectionality is a checklist item |
| Formula rewrite introduces errors | Each rewritten formula gets a sanity check against original; derivations cross-checked with standard references |

## Out of scope for this spec

- Generating new practice tasks beyond existing 24 in Part III
- Adding company-specific practice tasks (could be a follow-up project)
- English translation
- Interactive features (search, filters, progress tracking)
- PDF generation of the new structure

## Success criteria

- All 7 company chapters published with real interview questions (2023-2026, sourced)
- Part I frameworks chapter live with cross-company analytical examples
- Part III formulas revised with cross-company examples and "Используется в" sections
- Bidirectional cross-references verified between all cases and formulas
- Site deploys cleanly on GitHub Pages
- Candidate can prepare for an interview at any of the 7 target companies using only this guide + referenced public materials
