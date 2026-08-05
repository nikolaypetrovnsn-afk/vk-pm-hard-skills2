# PM Hard Skills Textbook Rebuild — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild the VK-only PM hard skills textbook into a Russian-market interview preparation guide with 7 company chapters, universal frameworks, revised formula reference, and bidirectional case↔formula cross-references.

**Architecture:** Three parts — (I) universal frameworks + cross-company analytical chapter, (II) 7 company guides following a standard template, (III) revised reference (formulas + practice). Static HTML, GitHub Pages deployment on push to main. Bidirectional cross-references between Part II cases and Part III formulas, annotated with context.

**Tech Stack:** Static HTML, CSS (existing `styles.css`), GitHub Pages. No build system, no JS framework.

**Spec:** `docs/superpowers/specs/2026-08-05-textbook-rebuild-design.md`

---

## File Structure

### Files to rename
- `block1-reference.html` → `part3-reference.html` (3321 lines, 18 formula sections)
- `block3-practice.html` → `part3-practice.html` (1214 lines, 24 tasks)

### Files to delete
- `vk-pm-hard-skills-guide.html` (600 KB single-page version, superseded)
- `vk-pm-hard-skills-guide.pdf` (PDF export of single-page version)
- `block2-cases.html` (after content migrated to `part2-vk.html`)

### Files to create
- `part1-frameworks.html` (~2500-3000 lines) — 5 chapters: Product Sense, Analytical, Estimation, System Design, Behavioral
- `part2-vk.html` (~700-900 lines) — VK company guide (migrated + expanded)
- `part2-yandex.html` (~700-900 lines)
- `part2-ozon.html` (~700-900 lines)
- `part2-tinkoff.html` (~700-900 lines)
- `part2-avito.html` (~700-900 lines)
- `part2-wildberries.html` (~700-900 lines)
- `part2-middle.html` (~500-700 lines) — Kontur, Skyeng, 2GIS summary
- `docs/research/yandex.md`, `docs/research/ozon.md`, `docs/research/tinkoff.md`, `docs/research/avito.md`, `docs/research/wildberries.md`, `docs/research/middle.md` — research outputs (not deployed)

### Files to modify
- `index.html` — landing page, 3-part navigation
- `styles.css` — new CSS classes for Part I/II sections
- `part3-reference.html` (after rename) — formula revision pass

---

## Sidebar Template (used in all files)

Every HTML file uses the same sidebar. The only difference is which link has `class="active"`. Template:

```html
<nav class="sidebar">
  <div class="sidebar-logo">PM Hard Skills</div>
  <a href="index.html">Главная</a>

  <div class="section-label">Часть I. Frameworks</div>
  <a href="part1-frameworks.html#product-sense" class="sub-link">Product Sense</a>
  <a href="part1-frameworks.html#analytical" class="sub-link">Analytical</a>
  <a href="part1-frameworks.html#estimation" class="sub-link">Estimation</a>
  <a href="part1-frameworks.html#system-design" class="sub-link">System Design</a>
  <a href="part1-frameworks.html#behavioral" class="sub-link">Behavioral</a>

  <div class="section-label">Часть II. Компании</div>
  <a href="part2-vk.html" class="sub-link">ВКонтакте</a>
  <a href="part2-yandex.html" class="sub-link">Яндекс</a>
  <a href="part2-ozon.html" class="sub-link">Ozon</a>
  <a href="part2-tinkoff.html" class="sub-link">Тинькофф</a>
  <a href="part2-avito.html" class="sub-link">Avito</a>
  <a href="part2-wildberries.html" class="sub-link">Wildberries</a>
  <a href="part2-middle.html" class="sub-link">Middle-tier</a>

  <div class="section-label">Часть III. Reference</div>
  <a href="part3-reference.html#metrics" class="sub-link">Метрики</a>
  <a href="part3-reference.html#experimentation" class="sub-link">Экспериментация</a>
  <a href="part3-reference.html#economics" class="sub-link">Экономика</a>
  <a href="part3-reference.html#data" class="sub-link">Данные</a>
  <a href="part3-reference.html#discovery" class="sub-link">Discovery</a>
  <a href="part3-reference.html#okr" class="sub-link">OKR</a>
  <a href="part3-reference.html#prioritization" class="sub-link">Приоритизация</a>
  <a href="part3-reference.html#backlog" class="sub-link">Бэклог</a>
  <a href="part3-reference.html#pmf" class="sub-link">PMF</a>
  <a href="part3-reference.html#pricing" class="sub-link">Pricing</a>
  <a href="part3-reference.html#growth" class="sub-link">Growth</a>
  <a href="part3-reference.html#strategy" class="sub-link">Strategy</a>
  <a href="part3-reference.html#stakeholders" class="sub-link">Stakeholders</a>
  <a href="part3-reference.html#competitive" class="sub-link">Competitive</a>
  <a href="part3-reference.html#gtm" class="sub-link">GTM</a>
  <a href="part3-reference.html#tech-fluency" class="sub-link">Tech Fluency</a>
  <a href="part3-reference.html#people" class="sub-link">People</a>
  <a href="part3-reference.html#marketplace" class="sub-link">Marketplace</a>
  <a href="part3-practice.html" class="sub-link">Практикум</a>
</nav>
```

**Note:** Sidebar logo changes from "VK PM Hard Skills" to "PM Hard Skills" (market-wide, not VK-only).

---

## Company Chapter Template (Part II)

All 7 company chapters follow this structure. Section IDs use the pattern `{company}-{section}` (e.g., `yandex-profile`, `yandex-format`).

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport"="width=device-width, initial-scale=1.0">
  <title>[Компания] — PM Interview Guide</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <!-- SIDEBAR (with this chapter's link active) -->

  <main class="main">
    <h1>[Компания]: подготовка к собеседованию</h1>

    <!-- §N.1 Профиль -->
    <h2 id="[company]-profile">Профиль компании</h2>
    <p>Бизнес-модель, продукты, культура, уровни...</p>

    <!-- §N.2 Формат собеседования -->
    <h2 id="[company]-format">Формат собеседования</h2>
    <p>Раунды, последовательность, где отсеивают...</p>

    <!-- §N.3 Реальные вопросы -->
    <h2 id="[company]-questions">Реальные вопросы с собеседований</h2>

    <h3>Product Sense</h3>
    <div class="question-card">
      <div class="question-meta">
        <span class="badge badge-frequency">Частый</span>
        <span class="source-year">2024</span>
      </div>
      <p class="question-text">[Формулировка вопроса]</p>
      <p class="question-tests"><strong>Проверяют:</strong> [что проверяют]</p>
      <p class="question-tips"><strong>На что обратить внимание:</strong> [совет]</p>
      <p class="question-source"><a href="[URL]">Источник</a></p>
    </div>
    <!-- ... more questions ... -->

    <h3>Analytical</h3>
    <!-- ... -->

    <h3>Estimation</h3>
    <!-- ... -->

    <h3>System Design</h3>
    <!-- ... -->

    <h3>Behavioral</h3>
    <!-- ... -->

    <!-- §N.4 Разборы кейсов -->
    <h2 id="[company]-cases">Разборы кейсов</h2>

    <div class="case-walkthrough" id="[company]-case-1">
      <h3>[Название кейса]</h3>
      <p><strong>Формулировка:</strong> [текст]</p>
      <p><strong>Структура ответа:</strong> [фреймворк]</p>
      <div class="answer-example">
        <p>[Пример ответа, 8-12 минут устного разбора]</p>
        <p>Применяю <a href="part3-reference.html#retention-rolling">rolling retention</a>
        с 30-дневным окном — для рекомендательной ленты короткое окно
        скрывает отложенное возвращение пользователей.</p>
      </div>
      <p><strong>Что часто упускают:</strong> [типичные ошибки]</p>
      <p><strong>Красные флаги:</strong> [что не говорит]</p>
    </div>

    <!-- §N.5 Что отличает -->
    <h2 id="[company]-distinct">Что отличает [компанию]</h2>
    <p>Специфика expectations, типичные ошибки, сигналы успеха...</p>

    <!-- §N.6 Чек-лист -->
    <h2 id="[company]-checklist">Чек-лист подготовки</h2>
    <ul class="checklist">
      <li>Повторить в Части I: <a href="part1-frameworks.html#analytical">Analytical methods</a>, <a href="part1-frameworks.html#estimation">Estimation</a></li>
      <li>Прорешать в Части III: <a href="part3-practice.html#metrics">Метрики 3.1-3.3</a>, <a href="part3-practice.html#experimentation">Эксперименты 3.2.1</a></li>
      <li>Почитать: [публичные материалы]</li>
      <li>Self-check: [5 вопросов]</li>
    </ul>
  </main>
</body>
</html>
```

---

## Phase 1 — Skeleton

### Task 1.1: Rename block1 and block3

**Files:**
- Rename: `block1-reference.html` → `part3-reference.html`
- Rename: `block3-practice.html` → `part3-practice.html`

- [ ] **Step 1: Rename block1**

```bash
cd /Users/nikolay.petrov/Documents/HardSkills
git mv block1-reference.html part3-reference.html
```

- [ ] **Step 2: Rename block3**

```bash
git mv block3-practice.html part3-practice.html
```

- [ ] **Step 3: Verify renames**

```bash
ls part3-reference.html part3-practice.html
```
Expected: both files listed, no `block1-*` or `block3-*` files.

### Task 1.2: Delete obsolete files

**Files:**
- Delete: `vk-pm-hard-skills-guide.html`
- Delete: `vk-pm-hard-skills-guide.pdf`

- [ ] **Step 1: Delete single-page HTML and PDF**

```bash
cd /Users/nikolay.petrov/Documents/HardSkills
git rm vk-pm-hard-skills-guide.html vk-pm-hard-skills-guide.pdf
```

- [ ] **Step 2: Verify deletions**

```bash
ls vk-pm-hard-skills-guide.* 2>/dev/null || echo "deleted"
```
Expected: `deleted`

### Task 1.3: Add new CSS classes to styles.css

**Files:**
- Modify: `styles.css` (append new classes)

- [ ] **Step 1: Read current styles.css to understand existing classes**

```bash
wc -l styles.css
```

- [ ] **Step 2: Append new classes for Part I/II sections**

Add to end of `styles.css`:

```css
/* === Part I & II additions === */

.question-card {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 16px;
  margin: 12px 0;
  background: #fafafa;
}

.question-meta {
  display: flex;
  gap: 8px;
  align-items: center;
  margin-bottom: 8px;
}

.badge-frequency {
  background: #ff6b35;
  color: white;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 0.8em;
}

.badge-frequency.rare {
  background: #888;
}

.source-year {
  color: #666;
  font-size: 0.85em;
}

.question-text {
  font-weight: 500;
  margin: 8px 0;
}

.question-tests, .question-tips {
  margin: 4px 0;
  font-size: 0.95em;
}

.question-source {
  font-size: 0.85em;
  color: #666;
  margin-top: 8px;
}

.case-walkthrough {
  border-left: 3px solid #4a90d9;
  padding-left: 16px;
  margin: 20px 0;
}

.answer-example {
  background: #f5f8fc;
  padding: 12px;
  border-radius: 4px;
  margin: 12px 0;
}

.formula-backref {
  background: #fff8e1;
  border: 1px solid #ffe082;
  border-radius: 4px;
  padding: 12px;
  margin: 16px 0;
}

.formula-backref-title {
  font-weight: 600;
  margin-bottom: 8px;
}

.formula-inline-ref {
  background: #e3f2fd;
  padding: 1px 4px;
  border-radius: 2px;
  text-decoration: none;
  border-bottom: 1px dotted #1565c0;
}
```

- [ ] **Step 3: Verify CSS file valid**

```bash
wc -l styles.css
```
Expected: line count increased by ~70 lines.

### Task 1.4: Update index.html for 3-part navigation

**Files:**
- Modify: `index.html` (full rewrite of body content)

- [ ] **Step 1: Read current index.html**

```bash
cat index.html
```

- [ ] **Step 2: Rewrite index.html**

Replace the entire `<body>` section with:

```html
<body>
  <nav class="sidebar">
    [SIDEBAR TEMPLATE — see "Sidebar Template" above, no link active]
  </nav>

  <main class="main">
    <div class="hero">
      <h1>PM Hard Skills — Interview Guide</h1>
      <p class="subtitle">Практическое руководство для подготовки к собеседованиям на продакт-менеджера в российский tech: ВК, Яндекс, Ozon, Тинькофф, Avito, Wildberries и middle-tier</p>
    </div>

    <div class="block-cards">
      <a href="part1-frameworks.html" class="block-card">
        <div class="card-icon">📐</div>
        <h3>Часть I. Frameworks</h3>
        <p>Универсальные методы: Product Sense, Analytical (метрики, A/B, SQL — с cross-company примерами), Estimation, System Design, Behavioral</p>
      </a>

      <a href="part2-vk.html" class="block-card">
        <div class="card-icon">🏢</div>
        <h3>Часть II. Компании</h3>
        <p>7 гайдов: ВК, Яндекс, Ozon, Тинькофф, Avito, Wildberries, middle-tier. Реальные вопросы с собеседований (2023-2026), разборы кейсов, чек-листы подготовки</p>
      </a>

      <a href="part3-reference.html" class="block-card full-width">
        <div class="card-icon">📚</div>
        <h3>Часть III. Reference</h3>
        <p>Справочник формул (18 разделов) и практикум (24 задачи) — с cross-company примерами и двусторонними ссылками на кейсы</p>
      </a>
    </div>

    <h2>Как пользоваться</h2>
    <ol class="checklist">
      <li><strong>Готовитесь к конкретной компании?</strong> — откройте Часть II, найдите нужный гайд</li>
      <li><strong>Готовитесь к формату (analytical, estimation)?</strong> — откройте Часть I</li>
      <li><strong>Нужна формула или практика?</strong> — Часть III</li>
      <li><strong>Кейсы ссылаются на формулы</strong> — переходите по ссылкам с аннотациями, чтобы понять, как метод применяется</li>
    </ol>
  </main>
</body>
```

- [ ] **Step 3: Verify index.html renders**

Open `index.html` in browser (or check HTML structure):
```bash
grep -c "part1-frameworks\|part2-vk\|part3-reference" index.html
```
Expected: 3+ (links to all three parts present)

### Task 1.5: Update sidebar in part3-reference.html

**Files:**
- Modify: `part3-reference.html` (sidebar section only)

- [ ] **Step 1: Find the sidebar in part3-reference.html**

```bash
grep -n "sidebar" part3-reference.html | head -5
```

- [ ] **Step 2: Replace the `<nav class="sidebar">...</nav>` block**

Replace the entire `<nav class="sidebar">` block (lines ~10-46) with the new sidebar template from "Sidebar Template" above. Mark the `part3-reference.html#metrics` link as active:

```html
<a href="part3-reference.html#metrics" class="sub-link active">Метрики</a>
```

- [ ] **Step 3: Verify sidebar updated**

```bash
grep -c "Часть I\|Часть II\|Часть III" part3-reference.html
```
Expected: 3+ (three part labels present)

### Task 1.6: Update sidebar in part3-practice.html

**Files:**
- Modify: `part3-practice.html` (sidebar section only)

- [ ] **Step 1: Replace the sidebar block**

Same as Task 1.5 but mark the practice link as active:

```html
<a href="part3-practice.html" class="sub-link active">Практикум</a>
```

- [ ] **Step 2: Verify**

```bash
grep -c "Часть I\|Часть II\|Часть III" part3-practice.html
```
Expected: 3+

### Task 1.7: Create stub files for Part I and Part II

**Files:**
- Create: `part1-frameworks.html`
- Create: `part2-vk.html`, `part2-yandex.html`, `part2-ozon.html`, `part2-tinkoff.html`, `part2-avito.html`, `part2-wildberries.html`, `part2-middle.html`

- [ ] **Step 1: Create part1-frameworks.html stub**

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Frameworks — PM Hard Skills</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <nav class="sidebar">
    [SIDEBAR TEMPLATE — mark part1-frameworks.html#product-sense as active]
  </nav>

  <main class="main">
    <h1>Часть I. Универсальные frameworks</h1>
    <p>Заглушка — будет заполнено в Фазе 5.</p>

    <h2 id="product-sense">1.1 Product Sense</h2>
    <p>TODO в Фазе 5.</p>

    <h2 id="analytical">1.2 Analytical methods</h2>
    <p>TODO в Фазе 5.</p>

    <h2 id="estimation">1.3 Estimation</h2>
    <p>TODO в Фазе 5.</p>

    <h2 id="system-design">1.4 System Design для PM</h2>
    <p>TODO в Фазе 5.</p>

    <h2 id="behavioral">1.5 Behavioral</h2>
    <p>TODO в Фазе 5.</p>
  </main>
</body>
</html>
```

- [ ] **Step 2: Create stub for each Part II chapter**

For each of `part2-vk.html`, `part2-yandex.html`, `part2-ozon.html`, `part2-tinkoff.html`, `part2-avito.html`, `part2-wildberries.html`, `part2-middle.html`, create a stub:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[Компания] — PM Interview Guide</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <nav class="sidebar">
    [SIDEBAR TEMPLATE — mark this chapter's link as active]
  </nav>

  <main class="main">
    <h1>[Компания]: подготовка к собеседованию</h1>
    <p>Заглушка — будет заполнено в Фазе 2 (ВК) или Фазе 4 (остальные).</p>
  </main>
</body>
</html>
```

- [ ] **Step 3: Verify all stubs created**

```bash
ls part1-frameworks.html part2-*.html
```
Expected: 8 files listed.

### Task 1.8: Commit and push Phase 1

- [ ] **Step 1: Stage all changes**

```bash
cd /Users/nikolay.petrov/Documents/HardSkills
git add -A
```

- [ ] **Step 2: Verify staged changes**

```bash
git status
```
Expected: renames, deletions, new stubs, modified index/styles.

- [ ] **Step 3: Commit**

```bash
git commit -m "$(cat <<'EOF'
Phase 1: restructure textbook to 3-part architecture

- Rename block1-reference.html → part3-reference.html
- Rename block3-practice.html → part3-practice.html
- Delete single-page version (vk-pm-hard-skills-guide.html, .pdf)
- Update index.html for 3-part navigation
- Update sidebar in part3 files
- Create stub files for Part I (frameworks) and Part II (7 companies)
- Add CSS classes for question cards, case walkthroughs, formula backrefs

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
EOF
)"
```

- [ ] **Step 4: Push to deploy**

```bash
git push origin main
```

- [ ] **Step 5: Verify deploy**

Wait 1-2 minutes, then check https://nikolaypetrovnsn-afk.github.io/vk-pm-hard-skills2/index.html — should show new 3-part landing page.

---

## Phase 2 — VK as Model Chapter

### Task 2.1: Migrate VK content from block2-cases.html to part2-vk.html

**Files:**
- Read: `block2-cases.html` (595 lines, 4 VK cases: Feed, Clips, Messenger, Video)
- Create: `part2-vk.html` (full chapter per template)

- [ ] **Step 1: Read block2-cases.html fully to inventory content**

```bash
cat block2-cases.html
```
Note: 4 cases (Feed/Clips/Messenger/Video), each with multiple sub-sections (NSM, A/B, economics, etc.)

- [ ] **Step 2: Create part2-vk.html with chapter template**

Write `part2-vk.html` following the "Company Chapter Template" above, with:
- §1 Профиль ВК: бизнес-модель (соцсеть + экосистема), продукты (Лента, Клипы, Мессенджер, Видео, Группы, Закладки), культура (data-driven, A/B-культура), уровни (middle/senior/lead)
- §2 Формат собеседования: типичные раунды (HR screen → product case → analytical → system design → final)
- §3 Реальные вопросы: использовать существующие материалы из `docs/` (vk-feed-*, vk-clips-*, messenger-superapp-*, etc.) + web-research для дополнения
- §4 Разборы кейсов: мигрировать 4 кейса из block2 (Feed, Clips, Messenger, Video) с inline-аннотациями формул
- §5 Что отличает ВК: фокус на рекомендательных системах, network effects, A/B с network effects
- §6 Чек-лист: ссылки на Часть I (analytical, system-design) и Часть III (metrics, experimentation, marketplace)

- [ ] **Step 3: Add inline formula annotations in case walkthroughs**

For each formula used in a case, add an annotation like:

```html
<p>Применяю <a href="part3-reference.html#retention-rolling" class="formula-inline-ref">rolling retention</a>
с 30-дневным окном — для рекомендательной Ленты короткое окно скрывает
отложенное возвращение пользователей.</p>
```

Key formulas to annotate in VK cases:
- Feed: NSM, rolling retention, A/B test design, network effects
- Clips: funnel metrics, LTV, unit economics, growth loops
- Messenger: network effects, indirect impact, discovery
- Video: cohort analysis, pricing, autoplay A/B

- [ ] **Step 4: Verify part2-vk.html structure**

```bash
grep -c "<h2" part2-vk.html
```
Expected: 6+ (6 main sections per template)

```bash
grep -c "formula-inline-ref" part2-vk.html
```
Expected: 10+ (multiple formula annotations across cases)

### Task 2.2: Delete block2-cases.html

**Files:**
- Delete: `block2-cases.html`

- [ ] **Step 1: Verify content migrated**

```bash
grep -c "feed-nsm\|clips-nsm\|messenger-nsm\|video-nsm" part2-vk.html
```
Expected: 4+ (all 4 cases present in new file)

- [ ] **Step 2: Delete block2-cases.html**

```bash
git rm block2-cases.html
```

### Task 2.3: Commit and push Phase 2

- [ ] **Step 1: Commit**

```bash
git add -A
git commit -m "$(cat <<'EOF'
Phase 2: migrate VK cases to company chapter template

- Migrate 4 VK cases (Feed, Clips, Messenger, Video) from block2-cases.html
  to part2-vk.html following the standard company chapter template
- Add §1 Profile, §2 Format, §5 Distinct, §6 Checklist sections
- Add inline formula annotations in case walkthroughs (10+ cross-refs)
- Delete block2-cases.html (content fully migrated)

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
EOF
)"
```

- [ ] **Step 2: Push**

```bash
git push origin main
```

- [ ] **Step 3: Verify VK chapter at live URL**

Check https://nikolaypetrovnsn-afk.github.io/vk-pm-hard-skills2/part2-vk.html

---

## Phase 3 — Parallel Research

**Use superpowers:dispatching-parallel-agents skill.** Dispatch 6 agents in parallel, one per company.

### Task 3.1: Dispatch 6 parallel research agents

- [ ] **Step 1: Dispatch agents in parallel (single message, 6 Agent tool calls)**

Each agent gets this prompt template (replace `{company}` and `{context}`):

```
Research real PM interview questions for {company} (Russian tech company).
Save findings to /Users/nikolay.petrov/Documents/HardSkills/docs/research/{slug}.md

Target: 15-25 interview questions for Product Manager positions (middle/senior/lead).
Distribute across 5 formats:
- Product Sense (5-8 questions)
- Analytical (5-8 questions: metrics, A/B testing, SQL)
- Estimation (3-5 questions)
- System Design (3-5 questions)
- Behavioral (3-5 questions)

For each question, provide:
1. Formulation (in Russian, as it was asked)
2. What it tests (1-2 sentences)
3. Source URL (Habr, Medium, YouTube, Telegram, company blog, Glassdoor analog)
4. Year of source (2023-2026 only — exclude older)

CRITICAL FILTER: Only include questions from sources dated 2023-2026.
If a source is older, exclude it. Do not include undated questions.

Triangulation: If 2+ independent sources mention similar questions, mark as "Частый" (frequent).
Unique/rare questions: mark as "Редкий".

Sources to search (priority order):
1. Habr — habr.com (search "{company} собеседование продакт", company blog)
2. Medium — Russian PM authors
3. YouTube — company tech channels, conference talks (HighLoad, Podlodka, PM Conf)
4. Telegram (public search) — PM channels, company chats
5. Company engineering/product blogs
6. Glassdoor analogs: joblab.ru, fantasy.jobs

{company_specific_context}

Output format (markdown):

# {Company} — Interview Questions Research

## Summary
- Total questions: N
- By format: Product Sense (X), Analytical (Y), Estimation (Z), System Design (W), Behavioral (V)
- Date range of sources: YYYY-YYYY
- Most frequent patterns: [list]

## Product Sense

### [Частый/Редкий] Question 1
- **Формулировка:** [text]
- **Проверяют:** [what it tests]
- **Источник:** [URL]
- **Год:** [year]

[... more questions ...]

## Analytical
[...]

## Estimation
[...]

## System Design
[...]

## Behavioral
[...]

## Notes
- Patterns observed: [summary]
- What distinguishes {company}: [observations from research]
- Gaps / couldn't find: [what's missing]
```

Agent dispatch parameters:
- `{company}` = "Яндекс", `{slug}` = "yandex", `{context}` = "Yandex is the largest Russian tech company. Key products: Search, Taxi, Market, Plus, Alice (AI), E-commerce. Known for analytical rigor, estimation questions, system design at scale."
- `{company}` = "Ozon", `{slug}` = "ozon", `{context}` = "Ozon is a major Russian e-commerce/marketplace. Products: marketplace, logistics, fintech (Ozon Bank), e-commerce. Known for marketplace mechanics, unit economics, P&L thinking."
- `{company}` = "Тинькофф", `{slug}` = "tinkoff", `{context}` = "Tinkoff (now T-Bank) is a Russian fintech. Products: banking, investments, business (Tinkoff Business), ecosystem. Known for fintech domain, mobile-first, metrics-driven culture."
- `{company}` = "Avito", `{slug}` = "avito", `{context}` = "Avito is a Russian classifieds/marketplace. Products: classifieds, real estate, auto, jobs, services. Known for marketplace dynamics, trust&safety, monetization."
- `{company}` = "Wildberries", `{slug}` = "wildberries", `{context}` = "Wildberries is the largest Russian e-commerce. Products: marketplace, logistics, fintech. Known for speed, P&L thinking, operational excellence. Note: less public info, may need indirect signals from hh.ru vacancies and PM talks."
- `{company}` = "Middle-tier (Контур, Skyeng, 2ГИС)", `{slug}` = "middle", `{context}` = "Three middle-tier Russian tech companies. Контур (B2B SaaS, accounting/HR tools), Skyeng (EdTech, online education), 2ГИС (maps/directories). Less public interview info — reconstruct from conference talks, company blogs, hh.ru. 2-3 questions per format per company."

- [ ] **Step 2: Wait for all 6 agents to complete**

Agents run in parallel. Each returns a summary of findings. The full research is saved to `docs/research/{slug}.md`.

### Task 3.2: Verify research outputs

- [ ] **Step 1: Check all 6 research files exist**

```bash
ls docs/research/
```
Expected: `yandex.md ozon.md tinkoff.md avito.md wildberries.md middle.md`

- [ ] **Step 2: Spot-check question counts**

```bash
for f in docs/research/*.md; do echo "=== $f ==="; grep -c "Формулировка:" "$f"; done
```
Expected: 15-25 per file.

- [ ] **Step 3: Spot-check year filter**

```bash
grep -E "Год: (201[0-9]|202[0-2])" docs/research/*.md
```
Expected: no matches (all pre-2023 excluded).

### Task 3.3: Commit research files

- [ ] **Step 1: Commit**

```bash
cd /Users/nikolay.petrov/Documents/HardSkills
git add docs/research/
git commit -m "$(cat <<'EOF'
Phase 3: collect interview questions from 6 companies

Research via parallel agents. Sources: Habr, Medium, YouTube, Telegram,
company blogs. Year filter: 2023-2026 only.

- docs/research/yandex.md
- docs/research/ozon.md
- docs/research/tinkoff.md
- docs/research/avito.md
- docs/research/wildberries.md
- docs/research/middle.md (Контур, Skyeng, 2ГИС)

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
EOF
)"
```

- [ ] **Step 2: Push**

```bash
git push origin main
```

---

## Phase 4 — Part II Company Chapters

For each company, write the chapter following the "Company Chapter Template" above, using the research file as input for §3 (questions) and §4 (case walkthroughs).

### Task 4.1: Write part2-yandex.html

**Files:**
- Modify: `part2-yandex.html` (replace stub)
- Read: `docs/research/yandex.md`

- [ ] **Step 1: Read research file**

```bash
cat docs/research/yandex.md
```

- [ ] **Step 2: Write part2-yandex.html following company chapter template**

Structure:
- §1 Профиль Яндекса: бизнес-модель (поисковик + экосистема: Taxi, Market, Plus, Alice), культура (аналитическая строгость, data-driven, формализация), уровни
- §2 Формат: ~5-6 раундов (HR → product sense → analytical → estimation → system design → final). Известно строгое интервью по estimation и analytical
- §3 Реальные вопросы: скопировать из `docs/research/yandex.md`, форматировать как question-card блоки
- §4 Разборы кейсов: 2-3 walked examples типичных Яндекс-кейсов (например: "оцени рынок Яндекс Такси в городе X", "улучши retention Яндекс Маркета")
- §5 Что отличает: формализация, оценки, аналитическая строгость, знание метрик поиска
- §6 Чек-лист: ссылки на Part I (estimation, analytical), Part III (metrics, experimentation), Part III practice tasks

- [ ] **Step 3: Add inline formula annotations**

In §4 case walkthroughs, annotate formulas used:
```html
<p>Применяю <a href="part3-reference.html#impact-top-down" class="formula-inline-ref">top-down estimation</a>
— начинаю с населения города, демографии, доли смартфонов...</p>
```

- [ ] **Step 4: Verify**

```bash
grep -c "question-card" part2-yandex.html
```
Expected: 15+ (one per question)

```bash
grep -c "formula-inline-ref" part2-yandex.html
```
Expected: 5+

### Task 4.2: Write part2-ozon.html

Same as Task 4.1 but for Ozon. Read `docs/research/ozon.md`.

Focus for Ozon:
- §1: e-commerce/marketplace, logistics, Ozon Bank, P&L thinking
- §4 cases: marketplace dynamics, unit economics, seller/buyer balance
- §5: P&L-мышление, marketplace mechanics, unit economics

- [ ] **Step 1-4: Same structure as Task 4.1, with Ozon-specific content**

### Task 4.3: Write part2-tinkoff.html

Same structure. Read `docs/research/tinkoff.md`.

Focus for Tinkoff:
- §1: fintech, banking, investments, Tinkoff Business, ecosystem
- §4 cases: fintech metrics (MAU, ARPU, LTV/CAC), mobile-first design, cross-sell
- §5: fintech domain knowledge, metrics-driven, mobile-first

### Task 4.4: Write part2-avito.html

Same structure. Read `docs/research/avito.md`.

Focus for Avito:
- §1: classifieds/marketplace, real estate, auto, jobs, services
- §4 cases: marketplace liquidity, trust&safety, monetization (paid placements)
- §5: marketplace dynamics, trust&safety, monetization models

### Task 4.5: Write part2-wildberries.html

Same structure. Read `docs/research/wildberries.md`.

Focus for Wildberries:
- §1: largest Russian e-commerce, marketplace, logistics, fintech
- §4 cases: operational excellence, speed, P&L, seller economics
- §5: скорость, P&L-мышление, operational rigor
- Note: research may be thinner — mark reconstructed content appropriately

### Task 4.6: Write part2-middle.html

Compact chapter covering 3 companies. Read `docs/research/middle.md`.

Structure:
- For each of Контур, Skyeng, 2ГИС:
  - Brief profile (1 paragraph)
  - Format summary (1 paragraph)
  - 2-3 questions per format (8-12 total per company)
  - 1 case walkthrough
  - What distinguishes
  - Mini checklist

- [ ] **Step 1: Write part2-middle.html with 3 sub-sections**

```html
<h2 id="kontur">Контур</h2>
[profile, format, questions, case, distinct, checklist]

<h2 id="skyeng">Skyeng</h2>
[profile, format, questions, case, distinct, checklist]

<h2 id="2gis">2ГИС</h2>
[profile, format, questions, case, distinct, checklist]
```

### Task 4.7: Commit and push Phase 4

- [ ] **Step 1: Commit all Part II chapters**

```bash
git add part2-*.html
git commit -m "$(cat <<'EOF'
Phase 4: write 6 company chapters (Yandex, Ozon, Tinkoff, Avito, Wildberries, middle-tier)

Each chapter follows the standard template:
- §1 Profile, §2 Format, §3 Real questions (from research), §4 Case walkthroughs,
  §5 Distinct features, §6 Preparation checklist
- Inline formula annotations with links to Part III
- Questions year-filtered to 2023-2026, with source URLs

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
EOF
)"
```

- [ ] **Step 2: Push**

```bash
git push origin main
```

---

## Phase 5 — Part I Frameworks

### Task 5.1: Write part1-frameworks.html — Product Sense section

**Files:**
- Modify: `part1-frameworks.html` (replace stub, section 1.1)

- [ ] **Step 1: Write section 1.1 Product Sense**

Content:
- CIRCLES framework (Comprehend, Identify, Report, Cut, List, Evaluate, Summarize)
- Walked example applying CIRCLES to a generic product case
- Cross-company note: how Yandex vs Ozon vs Tinkoff frame product sense questions (link to Part II chapters)
- Common pitfalls
- Links to Part III (PMF, growth, strategy sections)

### Task 5.2: Write Analytical methods section (1.2) — the cross-company chapter

**Files:**
- Modify: `part1-frameworks.html` (section 1.2)

- [ ] **Step 1: Write section 1.2 Analytical methods**

This is the central cross-company chapter. Content:

**1.2.1 Metric trees**
- North Star → input/output metrics
- How to build a metric tree
- Examples: Yandex Search NSM tree, Ozon GMV tree, Tinkoff MAU→revenue tree
- Link to Part III (metrics section)

**1.2.2 A/B testing methodology**
- Hypothesis → metric → MDE → sample size
- Common traps: network effects, peeking, multiple testing
- Cross-company: how Yandex asks about A/B (rigorous design), Ozon (marketplace A/B with spill-over), Tinkoff (fintech A/B with regulatory constraints)
- Link to Part III (experimentation section)

**1.2.3 SQL patterns**
- Cohort analysis (rolling retention SQL)
- Funnel analysis
- Window functions for time-series
- Cross-company: typical SQL tasks at Yandex vs Ozon vs Tinkoff
- Link to Part III (data section)

**1.2.4 Cross-company analytical question patterns**
- 3-5 worked examples showing how the same analytical concept is asked differently across companies
- Each example links to the relevant Part II chapter

### Task 5.3: Write Estimation section (1.3)

- [ ] **Step 1: Write section 1.3 Estimation**

Content:
- Top-down estimation (population → segment → conversion)
- Bottom-up estimation (supply × demand)
- Replacement theory
- Fermi problems
- Walked examples: "оцени рынок такси в Москве", "оцени GMV Ozon за год"
- Cross-company: Yandex estimation style vs others
- Link to Part III (where applicable)

### Task 5.4: Write System Design for PM section (1.4)

- [ ] **Step 1: Write section 1.4 System Design**

Content:
- Trade-off framework (build vs buy, latency vs consistency, etc.)
- Scaling concepts for PMs (not deep technical, but architectural)
- Architecture decision records
- Walked examples: "спроектируй рекомендательную систему для Ленты", "спроектируй систему выдачи для Ozon"
- Cross-company: Yandex system design expectations vs Avito vs Ozon
- Link to Part III (tech-fluency section)

### Task 5.5: Write Behavioral section (1.5)

- [ ] **Step 1: Write section 1.5 Behavioral**

Content:
- STAR framework (Situation, Task, Action, Result)
- Common leadership principles (customer obsession, ownership, etc.)
- Russian tech culture specifics (less formalized than FAANG, but growing)
- 5-7 common behavioral questions with STAR-structured sample answers
- Cross-company: behavioral focus at Yandex vs Tinkoff vs Avito
- Link to Part III (stakeholders, people sections)

### Task 5.6: Assemble and commit Part I

- [ ] **Step 1: Verify all 5 sections present**

```bash
grep -c "<h2" part1-frameworks.html
```
Expected: 5 (five main sections)

```bash
grep -c "part2-\|part3-" part1-frameworks.html
```
Expected: 15+ (cross-references to Part II and III)

- [ ] **Step 2: Commit and push**

```bash
git add part1-frameworks.html
git commit -m "$(cat <<'EOF'
Phase 5: write Part I — universal frameworks

5 chapters:
- 1.1 Product Sense (CIRCLES, walked example, cross-company notes)
- 1.2 Analytical methods (metric trees, A/B, SQL — cross-company chapter)
- 1.3 Estimation (top-down, bottom-up, Fermi)
- 1.4 System Design for PM (trade-offs, scaling, ADRs)
- 1.5 Behavioral (STAR, leadership principles)

Cross-references to Part II company chapters and Part III formulas.

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
EOF
)"
git push origin main
```

---

## Phase 6 — Part III Formula Revision

### Task 6.1: Audit formula sections for VK-only examples

**Files:**
- Read: `part3-reference.html`

- [ ] **Step 1: Read part3-reference.html and inventory formula blocks**

```bash
grep -n 'class="formula-block"' part3-reference.html
```
Note all formula block IDs and their current examples.

- [ ] **Step 2: Create audit checklist**

For each of the 18 formula sections, note:
- Current examples (VK-only or already cross-company?)
- Missing derivations?
- Missing "Используется в" back-reference section?

Save audit notes to `docs/research/formula-audit.md` (not deployed, working document).

### Task 6.2: Add cross-company examples to VK-only formulas

**Files:**
- Modify: `part3-reference.html`

- [ ] **Step 1: For each VK-only formula, add cross-company examples**

For example, in the `#retention-rolling` formula block, if only VK Feed is mentioned, add:

```html
<h4>Примеры применения</h4>
<ul>
  <li><strong>ВК Лента:</strong> retention рекомендательной vs подписочной, 30-дневное окно</li>
  <li><strong>Яндекс Поиск:</strong> retention пользователей после изменения выдачи</li>
  <li><strong>Ozon:</strong> retention покупателей после первого заказа</li>
  <li><strong>Tinkoff:</strong> retention после онбординга в новое приложение</li>
</ul>
```

Focus on formulas where cross-company examples add value:
- metrics (NSM, retention, MAU/DAU)
- experimentation (A/B design, MDE)
- economics (LTV, CAC, unit economics)
- growth (viral loops, growth rate)
- marketplace (liquidity, two-sided metrics)

- [ ] **Step 2: Add missing derivations**

For formulas stated without derivation, add a "Деривация" section:

```html
<h4>Деривация</h4>
<p>Из определения retention R = D_n / D_0, где D_n — активные пользователи
в день n, D_0 — пользователи дня 0. Rolling retention модифицирует:
D_n = пользователи, активные в день n ИЛИ вернувшиеся после паузы...</p>
```

### Task 6.3: Add "Используется в" back-reference sections

**Files:**
- Modify: `part3-reference.html`

- [ ] **Step 1: For each formula block, add a "Используется в" section at the bottom**

```html
<div class="formula-backref">
  <div class="formula-backref-title">Используется в:</div>
  <ul>
    <li><a href="part2-vk.html#feed-cohort">ВК Лента §II.1.4</a> — retention рекомендательной vs подписочной Ленты, 30-дневное окно</li>
    <li><a href="part2-yandex.html#search-retention">Яндекс Поиск §II.2.4</a> — DAU-impact оценки для A/B-теста</li>
    <li><a href="part2-ozon.html#marketplace-retention">Ozon Маркетплейс §II.3.4</a> — retention покупателей vs продавцов</li>
  </ul>
</div>
```

For each formula, list all Part II cases that use it. To find these:
```bash
grep -l "formula-inline-ref.*#retention-rolling" part2-*.html
```

- [ ] **Step 2: Verify bidirectionality**

For each formula F:
1. Find all cases that link to F: `grep -l "formula-inline-ref.*#F" part2-*.html`
2. Verify F's "Используется в" section lists all those cases

### Task 6.4: Commit and push Phase 6

- [ ] **Step 1: Commit**

```bash
git add part3-reference.html
git commit -m "$(cat <<'EOF'
Phase 6: revise Part III formulas

For each of 18 formula sections:
- Add cross-company examples (Yandex, Ozon, Tinkoff, Avito, Wildberries)
  where previously VK-only
- Add derivations where missing
- Add "Используется в" back-reference sections listing all Part II cases

Bidirectional cross-references verified between Part II cases and Part III formulas.

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
EOF
)"
git push origin main
```

---

## Phase 7 — Polish and Verification

### Task 7.1: Cross-reference verification — case → formula direction

- [ ] **Step 1: Extract all formula references from Part II**

```bash
grep -oh 'href="part3-reference.html#[^"]*"' part2-*.html | sort -u
```

- [ ] **Step 2: For each referenced formula, verify it exists in Part III**

```bash
for ref in $(grep -oh 'href="part3-reference.html#[^"]*"' part2-*.html | sort -u); do
  anchor=$(echo "$ref" | sed 's/href="part3-reference.html#//; s/"//')
  if ! grep -q "id=\"$anchor\"" part3-reference.html; then
    echo "BROKEN: $anchor"
  fi
done
```
Expected: no BROKEN lines.

### Task 7.2: Cross-reference verification — formula → case direction

- [ ] **Step 1: For each formula with "Используется в" section, verify back-references**

```bash
for formula_id in $(grep -oh 'id="[^"]*"' part3-reference.html | sed 's/id="//; s/"//'); do
  # Find cases claiming to use this formula
  backrefs=$(grep -A 20 "id=\"$formula_id\"" part3-reference.html | grep -o 'href="part2-[^"]*#[^"]*"')
  for ref in $backrefs; do
    file=$(echo "$ref" | sed 's/href="//; s/#.*//')
    anchor=$(echo "$ref" | sed 's/.*#//; s/"//')
    if [ ! -f "$file" ] || ! grep -q "id=\"$anchor\"" "$file"; then
      echo "BROKEN BACKREF: $formula_id → $ref"
    fi
  done
done
```
Expected: no BROKEN lines.

### Task 7.3: Verify annotation quality (no bare links)

- [ ] **Step 1: Check that formula-inline-ref links have surrounding context**

```bash
# Find formula-inline-ref links and check they have text around them
grep -B1 -A1 'formula-inline-ref' part2-*.html | grep -v "^--$" | head -40
```

Manually verify: each link should be in a sentence explaining why the formula applies. Bare links like `<a href="...">formula</a>` without context are failures.

### Task 7.4: Sidebar final pass

- [ ] **Step 1: Verify sidebar consistency across all files**

```bash
for f in index.html part1-frameworks.html part2-*.html part3-*.html; do
  echo "=== $f ==="
  grep -c "section-label" "$f"
done
```
Expected: 3 (three section labels: Часть I, II, III) in each file.

- [ ] **Step 2: Verify active link is correct in each file**

Each file should have exactly one `class="sub-link active"` corresponding to its location.

### Task 7.5: Mobile check

- [ ] **Step 1: Open site on mobile viewport**

Check https://nikolaypetrovnsn-afk.github.io/vk-pm-hard-skills2/ on mobile or use browser dev tools mobile view.

Verify:
- Sidebar collapses or is usable on mobile
- Question cards render correctly
- Code blocks (if any) don't overflow
- Formula backref boxes don't break layout

### Task 7.6: Link check

- [ ] **Step 1: Check all internal links resolve**

```bash
# Extract all internal links
grep -oh 'href="[a-z0-9-]*\.html[^"]*"' *.html | sort -u
```

For each, verify the target file exists and the anchor (if any) exists.

### Task 7.7: Final commit and push

- [ ] **Step 1: Commit any polish fixes**

```bash
git add -A
git commit -m "$(cat <<'EOF'
Phase 7: polish and verification

- Verify cross-reference bidirectionality (case ↔ formula)
- Verify annotation quality (no bare links)
- Sidebar consistency across all files
- Mobile layout check
- Link check (no broken anchors)

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
EOF
)"
git push origin main
```

- [ ] **Step 2: Final verification at live URL**

Open https://nikolaypetrovnsn-afk.github.io/vk-pm-hard-skills2/index.html and verify:
- 3-part landing page
- All 7 company chapters accessible
- Part I frameworks accessible
- Part III reference and practice accessible
- Cross-references work (click a formula link in a case → lands on formula → "Используется в" lists the case)

---

## Self-Review Notes

**Spec coverage:**
- ✅ 3-part architecture (Part I/II/III)
- ✅ 7 company chapters
- ✅ Cross-company analytical chapter (Task 5.2)
- ✅ Real interview questions from web research (Phase 3)
- ✅ Year filter 2023-2026 (Task 3.1 agent prompts)
- ✅ Bidirectional cross-references (Tasks 2.1, 6.3, 7.1, 7.2)
- ✅ Formula revision pass (Phase 6)
- ✅ Git push after each phase (every phase ends with commit+push)
- ✅ Delete obsolete files (Task 1.2)
- ✅ Sidebar update (Tasks 1.5, 1.6, 7.4)

**Placeholder scan:**
- Some tasks reference "[SIDEBAR TEMPLATE — see above]" — this is intentional DRY reference, not a placeholder
- Company-specific content in Phase 4 tasks is described by focus areas, not full HTML — content will be generated from research files
- Task 5.1-5.5 describe section content as bullet points — full HTML will be written during execution

**Type consistency:**
- Section IDs follow pattern `{company}-{section}` (yandex-profile, yandex-format, etc.)
- Formula anchors referenced consistently (e.g., `#retention-rolling`)
- File naming: `part{N}-{name}.html` consistent throughout
