# Механики модели рекомендаций для роста таймспента в VK Feed

**Дата:** 2026-07-14
**Подход:** Pipeline-layer-first (от слоёв pipeline к механикам)
**Scope:** Модель рекомендаций + pipeline (candidate generation → re-ranking)
**Статус:** Draft

---

## 1. Контекст

Предыдущий спек (2026-07-08) покрывал гипотезы роста таймспента в ленте в целом: UI, algo, cross-product. Этот документ фокусируется исключительно на механиках модели рекомендаций и pipeline — что можно менять внутри реком-системы для влияния на 4 целевые метрики.

**Целевые метрики:**
- **Таймспент:** timespent/DAU, avg_session_length
- **Частота:** sessions_per_DAU (замена ретеншена — не отслеживается)
- **Глубина сессий:** scroll_depth, content_completion_rate, posts_per_session
- **Вовлечённость:** DAU/MAU stickiness, interaction_rate, dwell_time_per_post

**Guard metrics:** unlike_rate, hide_rate, feed_exit_rate, ad_fatigue_rate

**Декомпозиция:** timespent/DAU = sessions_per_DAU × avg_session_length

---

## 2. Структура pipeline

5 слоёв рекомендательного pipeline, каждый — точка рычага:

| # | Слой | Что делает | Таймспент | Sessions/DAU | Глубина | Вовлечённость |
|---|------|-----------|-----------|--------------|---------|---------------|
| 1 | **Content Relevance & Quality** | Что попадает в систему: свежесть, релевантность, качество | Свежий контент → дольше сидим | Актуальный → причина зайти | Релевантный → скроллим не отсеивая | Качественный → взаимодействуем |
| 2 | **Candidate Generation** | Отбирает кандидатов из миллионов | Широта пула → есть что показать | Разнообразие источников → есть что возвращаться | Разнообразие форматов → скроллим дальше | Новые топики → чаще заходим |
| 3 | **Feature Engineering** | Сигналы для скоринга | Контекст → точное попадание в интент | Long-term фичи → персонализация роста | Сессионные фичи → адаптация в моменте | Social-фичи → вовлечение через друзей |
| 4 | **Target / Loss Design** | Что модель оптимизирует | Engagement-таргет → больше dwells | Frequency-таргет → возвращаемость | Multi-objective → баланс глубины и ширины | Composite target → клики + interaction |
| 5 | **Re-ranking / Post-processing** | Бизнес-правила, diversity, freshness | Diversity → нет выгорания | Freshness-boost → причина вернуться | Contrast → нет баннерной слепоты | Slot-стратегии → правильный контент в нужном месте |

**Принцип:** Изменение на раннем слое (content quality) даёт больший потенциал, но сложнее валидировать. Изменение на позднем слое (re-ranking) — быстрее экспериментировать, но потенциал ограничен качеством входного пула.

---

## 3. Слой 1: Content Relevance & Quality

### R1: Freshness window — ограничение возраста контента в пуле (ICE: 504)

**Идея:** Не подавать в candidate generation контент старше N часов/дней. Свежий контент — причина зайти и причина остаться.

**Механика:** Exponential decay по age поста. Кандидаты старше 48ч получают penalty -50% к retrieval score. Для subscription-контента — tighter window (12ч), для viral-контента — wider (72ч). Альтернативно: hard cutoff — контент старше N дней не входит в pool.

**Метрики:** `fresh_impression_share`, `content_age_at_impression_p50`, `sessions_per_DAU`, `dwell_time_per_post`

**Риски:** Пустой feed для пользователей с малым числом подписок; penalised но качественный evergreen-контент; разные оптимальные окна для разных content types

---

### R2: Content quality scoring — предфильтрация на входе (ICE: 420)

**Идея:** Перед candidate generation — quality gate: удалить низкокачественный контент (clickbait, спам, дубликаты, low-effort посты).

**Механика:** Quality-модель (отдельно от основной реком-модели) скорит каждый пост по: текстовая размеченность, наличие медиа, авторский скор, жалобы/скрытия. Порог отсечения → контент ниже не попадает в candidate pool.

**Метрики:** `quality_gate_pass_rate`, `feed_exit_rate` (ожидаем снижение), `dwell_time_per_post`, `unlike_rate`

**Риски:** Отсечь «народный» контент (мемы, простой текст); bias против новых авторов без истории; need separate quality model maintenance

---

### R3: Topical relevance — соответствие контента текущим интересам (ICE: 336)

**Идея:** Контент должен быть не просто «хорошим», а релевантным текущим интересам пользователя. Топик-модель фильтрует кандидатов по совпадению с user interest profile.

**Механика:** User interest profile обновляется на основе недавних действий (лайки, dwells, shares за 7 дней). Candidate post получает topical score = cosine similarity между post embedding и user profile. Порог → неремелевантный контент не подаётся.

**Метрики:** `topical_match_rate`, `CTR` (ожидаем рост), `scroll_depth`, `sessions_per_DAU`

**Риски:** Filter bubble — сужение интересов; cold start для новых пользователей без профиля; topic model quality dependency

---

### R4: Trend detection — буст трендового контента (ICE: 378)

**Идея:** Контент, который набирает engagement быстрее обычного, получает приоритет в pool — пока не стал массовым, показать раньше.

**Механика:** `trend_score = engagement_velocity / author_median_velocity`. Посты с trend_score > 2.0 получают priority injection в candidate pool. Не бустить в ранжировании, а именно подавать в pool раньше.

**Метрики:** `trend_content_impression_rate`, `early_adopter_satisfaction`, `sessions_per_DAU`

**Риски:** Хайп-контент низкого качества; echo chamber — все видят одно и то же; velocity нестабилен для мелких авторов

---

### R5: Content deduplication — удаление дубликатов и парафразов (ICE: 315)

**Идея:** Один и тот же контент (новость, мем, событие) от разных авторов загромождает ленту. На этапе pool — кластеризовать и оставить по одному представителю.

**Механика:** Semantic similarity между постами за последние 24ч. Кластеры с similarity > 0.85 (начальное значение, tuning по A/B) → оставить пост с максимальным quality score, остальные отложить. Если пользователь подписан на автора дубликата — показать отдельно.

**Метрики:** `dedup_rate`, `scroll_depth` (ожидаем рост — больше уникального), `feed_exit_rate` (снижение от устранения «опять одно и то же»)

**Риски:** Удалить контент с разной перспективой на одно событие; техническая сложность real-time clustering; threshold tuning

---

### R6: Format-specific quality gates — разные пороги для разных форматов (ICE: 252)

**Идея:** Качество текстового поста и видео — разные вещи. Единый quality gate penalises целые форматы. Разные пороги → каждый формат представлен качественно.

**Механика:** Отдельные quality-модели для: видео (completion rate, production quality), статьи (text length, structure, images), фото (resolution, album size), короткий текст (engagement velocity, author authority). Порог подбирается per-format.

**Метрики:** `format_diversity_in_feed`, `dwell_time_per_format`, `interaction_rate_per_format`

**Риски:** Over-engineering; сложность поддержки N моделей качества; bias против «простых» форматов

---

## 4. Слой 2: Candidate Generation

### CG1: Multi-source retrieval — диверсификация источников кандидатов (ICE: 504)

**Идея:** Кандидаты сейчас приходят из ограниченного числа источников (subscriptions, collaborative filtering, content-based). Добавление новых retrieval path → более широкий и разнообразный pool.

**Механика:** Параллельные retrieval path: (1) subscription-based, (2) CF-based, (3) content-embedding similarity, (4) graph-based (2-hop social graph), (5) trending pool, (6) geo-based. Каждый path отдаёт top-K. Union + merge перед основным ранжированием.

**Метрики:** `source_diversity_in_feed`, `coverage_rate` (% пользователей с 3+ источниками), `sessions_per_DAU`, `scroll_depth`

**Риски:** Увеличение latency; dilution качества если слабый path доминирует; сложность балансировки источников

---

### CG2: Exploration quota — зарезервировать слоты для нового контента (ICE: 378)

**Идея:** Модель склонна к exploitation — показывает то, что уже работает. Exploration quota гарантирует, что N% слотов в pool — контент, который модель ещё не знает как пользователь оценит.

**Механика:** На каждый retrieval request: 90% — exploitation (высокий predicted score), 10% — exploration (случайный из tail distribution). Exploration-кандидаты скорятся отдельно, если показывают высокий CTR — переводятся в exploitation.

**Метрики:** `exploration_slot_ctr`, `novel_topic_discovery_rate`, `D7_return_rate_for_exploration_users`, `long_tail_author_exposure`

**Риски:** Низкий CTR на exploration-слотах в краткосроке; пользователь видит нерелевантное; guard: если exploration CTR < threshold — уменьшить квоту

---

### CG3: Session-aware retrieval — адаптация пула под текущую сессию (ICE: 336)

**Идея:** На старте сессии — один pool (discovery, свежее), через 10 свайпов — другой (deep dive, похожее), на 30+ свайпах — третий (variety, переключение темы). Retrieval адаптируется к позиции в сессии.

**Механика:** 3 retrieval profile: (1) Session start: freshness + diversity + social, (2) Mid-session: interest deep-dive + format continuity, (3) Late session: topic switching + re-engagement triggers. Переключение по scroll_position.

**Метрики:** `late_session_scroll_depth`, `session_length_by_retrieval_profile`, `session_end_trigger_rate`

**Риски:** Overfitting к среднему паттерну; задержка на переключение профиля; сложность A/B тестирования (3 профиля × вариации)

---

### CG4: Negative sampling from rejection signals — учёт скрытий при candidate sourcing (ICE: 294)

**Идея:** Пользователь скрыл пост X → не показывать похожие посты из тех же источников/топиков. Сейчас учитывается только в ranking, а не в retrieval. Внедрить в candidate generation.

**Механика:** User rejection profile = кластер скрытых постов по (source, topic, format). При retrieval — hard exclude кандидатов с similarity > 0.7 к rejection profile. Период полураспада rejection signal — 30 дней.

**Метрики:** `hide_rate` (ожидаем снижение), `relevant_candidate_share`, `dwell_time_per_post`, `sessions_per_DAU`

**Риски:** Over-filtering — пользователь вырос, а rejection profile ещё старый; cold start для нового топика; similarity threshold tuning

---

### CG5: Author-candidate boosting — поддержка создателей контента (ICE: 231)

**Идея:** Новые и mid-tier авторы недопредставлены в candidate pool потому что у них низкий predicted engagement. Quota на авторов из long-tail → больше разнообразия и мотивации для создателей.

**Механика:** Author tier: top 10%, mid 40%, long-tail 50%. Minimum quota: каждый tier получает минимум K кандидатов в pool. Long-tail авторы с высоким quality score но низким predicted CTR — инъекция в pool.

**Метрики:** `author_diversity_in_feed`, `long_tail_author_ctr`, `creator_retention_D30` (экосистемный эффект)

**Риски:** Низкий CTR на long-tail слотах; создатели не = потребители; metric mismatch (short-term CTR vs long-term ecosystem)

---

### CG6: Cross-format retrieval — поиск кандидатов в других форматах (ICE: 210)

**Идея:** Пользователь смотрит видео в Clips → в Feed показать связанные статьи. Пользователь читает статью → в Feed показать related видео. Cross-format retrieval расширяет пул.

**Механика:** Cross-format embedding space: контент разных форматов маппится в единое векторное пространство. Для пользователя с video-heavy историей — подтянуть text/photo кандидатов из того же topic cluster и наоборот.

**Метрики:** `cross_format_impression_rate`, `cross_format_ctr`, `format_diversity_per_session`, `total_timespent_cross_format_users`

**Риски:** Качество cross-format embeddings; пользователь не хочет читать когда привык смотреть; latency

---

## 5. Слой 3: Feature Engineering

### F1: Dwell time normalized by format — нормализованный dwell как фича (ICE: 420)

**Идея:** Dwell time на видео и на тексте — разные масштабы. Сырой dwell bias'ит модель в сторону видео. Нормализация → модель учитывает глубину интереса, а не формат.

**Механика:** `dwell_norm = dwell_time / median_dwell_for_format`. Нормализация по (format_type, content_length_bucket). Используется как input feature в scoring model и как training target component.

**Метрики:** `text_content_ctr` (ожидаем рост), `video_completion_rate` (не должен упасть), `dwell_time_per_post_norm`, `scroll_depth`

**Риски:** Форматная сегментация слишком грубая; median нестабилен для редких форматов; bucket boundaries arbitrary

---

### F2: Session context features — позиция в сессии как сигнал (ICE: 378)

**Идея:** Поведение на 1-м посте и на 30-м — разное. Добавить фичи: `position_in_session`, `time_since_session_start`, `formats_seen_so_far`, `topics_seen_so_far`. Модель подстраивает ранжирование под момент сессии.

**Механика:** Session state vector обновляется с каждым действием. Фичи: (1) `scroll_position` (1, 2-5, 6-15, 16+), (2) `session_duration_bucket` (0-1min, 1-5, 5-15, 15+), (3) `topic_entropy_so_far` (как много разных тем уже видел), (4) `format_repetition_count` (сколько подряд одного формата).

**Метрики:** `session_length`, `late_session_dwell_time`, `session_end_position`, `format_diversity_per_session`

**Риски:** Feature drift в реальном времени; холодный старт сессии; сложно дебажить

---

### F3: Long-term interest features — профиль долгосрочных интересов (ICE: 336)

**Идея:** Модель видит в основном последние действия (recency bias). Добавить long-term profile: стабильные интересы за 30-90 дней. Это особенно важно для sessions_per_DAU — причина вернуться = стабильная связь с интересными топиками.

**Механика:** Two-tower: short-term profile (7 дней) + long-term profile (90 дней). Long-term = weighted average of topic embeddings, decay = 0.95/day. При scoring: `interest_score = 0.6 x short_term + 0.4 x long_term`. Если short-term и long-term расходятся — показывать из обоих.

**Метрики:** `sessions_per_DAU`, `topical_match_rate`, `D7_return_rate`, `interest_stability_score`

**Риски:** Long-term профиль может быть устаревшим; вычислительная нагрузка на 90-дневную агрегацию; transition моменты (смена интересов)

---

### F4: Social graph features — сигналы из социального графа (ICE: 294)

**Идея:** В VK богатый social graph, но модель может не использовать его полностью. Добавить: friend-of-friend engagement, group co-membership, social proximity к автору.

**Механика:** Фичи: (1) `social_distance_to_author` (1 = direct friend, 2 = friend-of-friend, 3+), (2) `friend_engagement_on_post` (сколько друзей провзаимодействовали), (3) `group_overlap_with_author` (общие группы), (4) `friend_topic_affinity` (друзья с похожими интересами лайкают).

**Метрики:** `social_content_ctr`, `interaction_rate` (лайки, комментарии на social content), `sessions_per_DAU` (social content → чаще заходят)

**Риски:** Privacy concerns; social bias → echo chamber; не всем пользователям важен social content

---

### F5: Content embedding features — семантические признаки контента (ICE: 252)

**Идея:** Использовать content embeddings (текст, изображение, видео) не только в retrieval, но и как features в scoring. Модель видит не только «автор + формат + CTR», но и «о чём этот пост».

**Механика:** Post embedding (from multimodal model) → cosine similarity с user profile embedding → feature для scoring. Дополнительно: `topic_novelty` = 1 - max_similarity_to_seen_topics (новый для пользователя топик).

**Метрики:** `topical_diversity_per_session`, `novel_topic_ctr`, `content_completion_rate`

**Риски:** Embedding quality varies; multimodal model latency; difficult to interpret

---

### F6: Temporal engagement features — паттерны вовлечённости по времени (ICE: 224)

**Идея:** Из Clips мы знаем, что intent зависит от времени дня. То же для Feed — утренний scroll отличается от вечернего. Добавить time-aware features.

**Механика:** Фичи: (1) `time_of_day_bucket` (morning/day/evening/night), (2) `day_of_week`, (3) `historical_engagement_at_this_time` (как пользователь обычно ведёт себя в это время), (4) `time_since_last_session`. Interaction features: `time_bucket x topic_affinity`.

**Метрики:** `time_intent_match_rate`, `sessions_per_DAU` (особенно в слабые слоты), `first_post_dwell_time`

**Риски:** Сезонность vs паттерн; shift при изменении рутины; маленькие бакеты → noise

---

## 6. Слой 4: Target / Loss Design

### T1: Multi-objective optimization — баланс engagement × frequency × depth (ICE: 448)

**Идея:** Сейчас модель оптимизирует CTR (или похожий short-term engagement). Это даёт клики, но не таймспент и не возврат. Multi-objective: `target = alpha x CTR + beta x dwell_norm + gamma x sessions_per_DAU_signal + delta x scroll_depth_signal`.

**Механика:** Pareto-optimal model через multi-task learning. Each objective — отдельный head, shared backbone. Weights подбираются через offline NDCG + online A/B с guard metrics. Начальные веса: CTR=0.4, dwell=0.3, scroll=0.2, frequency=0.1 (frequency — proxy sessions_per_DAU через 24h return signal).

**Метрики:** Все 4 целевые метрики одновременно. Guard: `unlike_rate`, `feed_exit_rate`

**Риски:** Conflicting objectives (CTR vs dwell); долгий цикл tuning; нет единого offline metric для сравнения моделей

---

### T2: Dwell-time-weighted loss — функция потерь с весом по dwell (ICE: 378)

**Идея:** Стандартный loss (logistic, pairwise) одинаково штрафует ошибку на посте, который пролистали за 1 сек, и на посте, который смотрели 30 сек. Весить примеры по dwell time → модель фокусируется на контенте, который реально удерживает.

**Механика:** `sample_weight = min(dwell_time / median_dwell, 3.0)`. Cap на 3x чтобы не over-weight outliers. Loss = weighted cross-entropy или weighted pairwise hinge.

**Метрики:** `dwell_time_per_post`, `session_length`, `high_dwell_impression_share` (% impressions с dwell > median)

**Риски:** Selection bias — модель видит только показанный контент; dwell bias к видео; нужен IPS (inverse propensity scoring)

---

### T3: Retention-aware training — добавление сигнала возврата в target (ICE: 336)

**Идея:** Модель не знает, вернётся ли пользователь завтра. Добавить в training target proxy: `return_signal = 1 if user returns within 24h, else 0`. Модель учится показывать контент, который вызывает возврат.

**Механика:** Two-stage: (1) Основная модель предсказывает engagement, (2) Auxiliary head предсказает return. Aux loss с весом 0.2. На inference — только основной head, но он обучен с учётом return signal.

**Метрики:** `sessions_per_DAU`, `inter_session_gap` (ожидаем уменьшение), `D1_return_rate`

**Риски:** Credit assignment — какой именно пост вызвал возврат?; долгий feedback loop; return зависит не только от ленты

---

### T4: Counterfactual learning — обучение на «что было бы если» (ICE: 280)

**Идея:** Модель обучается на показанных постах (selection bias). Если бы показали другие — engagement мог бы быть выше. Counterfactual approach (IPS, doubly robust) корректирует bias.

**Механика:** Inverse Propensity Scoring: `weight = 1 / P(show | user, context)`. P — вероятность показа текущей моделью. Редко показываемый но хороший контент получает высокий вес → модель перестает его penalise.

**Метрики:** `long_tail_ctr`, `exploration_slot_ctr`, `offline_NDCG_vs_online_correlation`

**Риски:** Высокая variance при малых propensity; need logging policy; сложность внедрения

---

### T5: Position-debiased training — удаление positional bias из target (ICE: 252)

**Идея:** Пост на позиции 1 получает высокий CTR не потому что релевантен, а потому что первый. Модель учит позиционный bias. Удалить → модель видит «истинный» engagement.

**Механика:** Position-as-feature: добавить `position` в features, при inference подставить константу (например, 1). Или position-as-intervention: separate model для estimating positional bias, вычесть из target.

**Метрики:** `NDCG_unbiased`, `late_position_ctr` (ожидаем рост — ранее скрытый хороший контент), `scroll_depth`

**Риски:** Unobserved confounders; не все bias = positional; сложность валидации

---

### T6: Delayed feedback modeling — учёт отложенного engagement (ICE: 210)

**Идея:** Пользователь увидел пост → не отреагировал сразу → через 2 часа вернулся и лайкнул. Модель в момент показа не знает об этом. Delayed feedback → модель недооценивает «медленный» контент.

**Механика:** Wait-n-seconds подход: перед обновлением training data ждать 1ч/4ч. Или probabilistic: model предсказывает P(eventual_engagement | seen_so_far). Если через 1ч нет клика — не считать negative, а weighted negative.

**Метрики:** `delayed_engagement_recovery_rate`, `slow_burn_content_ctr`, `diversity_of_liked_topics`

**Риски:** Latency в обучении; не все delayed engagement = полезный; over-credit случайным возвратам

---

## 7. Слой 5: Re-ranking / Post-processing

### RR1: Content contrast penalty — штраф за однообразие (ICE: 420)

**Идея:** Три видео подряд → баннерная слепота, выход из сессии. Diversity penalty в re-ranking: снижать score третьего поста того же формата/топика подряд.

**Механика:** `adjusted_score = base_score x (1 - penalty)`. `penalty = 0.3` для 3-го одинакового формата подряд, `0.5` для 4-го. Отдельно: topical penalty если 3+ поста из одного topic cluster подряд.

**Метрики:** `format_diversity_per_session`, `scroll_depth`, `session_length`, `feed_exit_rate`

**Риски:** Снижение relevance ради diversity; пустые слоты если контент одного типа доминирует; сложность tuning penalty

---

### RR2: Freshness boost в re-ranking — приоритет свежего контента (ICE: 378)

**Идея:** Свежий контент от подписок и других источников получает временный boost в re-ranking. Обобщение гипотезы B3 из предыдущего спека.

**Механика:** `freshness_factor = exp(-age_hours / half_life)`. Half-life: 2ч для breaking news, 12ч для подписок, 48ч для viral. Boost: `score = base_score x (1 + freshness_factor)`. Decay до 1.0 (нет boost) по мере старения.

**Метрики:** `fresh_impression_share`, `sessions_per_DAU` (причина вернуться), `time_to_first_fresh_post`

**Риски:** Старый но качественный контент penalised; noise для пользователей с редкими заходами (всё свежее для них)

---

### RR3: Session momentum — re-ranking под текущий momentum сессии (ICE: 336)

**Идея:** Если пользователь активно скроллит и кликает — momentum high → дать больше глубокого контента (статьи, длинные видео). Если скроллит без взаимодействия — momentum low → переключить на variety, quick hits, social content.

**Механика:** `momentum = rolling_avg(interactions_per_last_5_posts)`. High momentum (>0.4): boost deep content, reduce quick hits. Low momentum (<0.1): boost variety, social, quick satisfaction. Реактивное переключение каждые 5 позиций.

**Метрики:** `session_length`, `interaction_rate`, `late_session_dwell_time`, `session_end_trigger_rate`

**Риски:** Over-reaction к noise; momentum lag; не все пользователи следуют паттерну

---

### RR4: Engagement pacing — управление «пиками» вовлечённости (ICE: 294)

**Идея:** Показать весь лучший контент в начале сессии → потом обвисание → выход. Pacing: распределять высоко-engaging контент равномерно, чтобы поддерживать интерес на протяжении всей сессии.

**Механика:** Score adjustment: `paced_score = base_score x pacing_factor`. `pacing_factor` зависит от позиции: в начале сессии (1-5) — capped на 0.8 (не всё лучшее сразу), в середине (6-20) — 1.0, в конце (20+) — boost 1.2 для удержания. Или: top-N постов shuffle с stratification по engagement quintile.

**Метрики:** `session_length`, `dwell_time_by_position_decile`, `late_session_exit_rate`

**Риски:** Пользователь не видит лучшее сразу → разочарование; сложность балансировки; unrealised potential (good content delayed = never seen)

---

### RR5: Slot-specific re-ranking — разные стратегии для разных позиций ленты (ICE: 252)

**Идея:** Позиция 1-3 — «hook» (привлечь внимание, высокий CTR), 4-10 — «engage» (глубокий контент, высокий dwell), 11-20 — «retain» (разнообразие, discovery), 20+ — «re-engage» (социальное, триггеры возврата).

**Механика:** Каждая зона имеет свою re-ranking стратегию с разными весами: Hook: CTR weight 0.7; Engage: dwell weight 0.5; Retain: diversity weight 0.4; Re-engage: social weight 0.4, freshness 0.3.

**Метрики:** `zone_ctr`, `zone_dwell`, `zone_exit_rate`, `sessions_per_DAU`

**Риски:** Zone boundaries arbitrary; не у всех пользователей одинаковый паттерн; over-segmentation

---

### RR6: Fatigue detection — снижение разнообразия при признаках усталости (ICE: 210)

**Идея:** Пользователь быстро скроллит, не останавливаясь — признак fatigue. Не показывать больше того же. Переключить на полностью другой контентный профиль или предложить выход из ленты (в другой продукт).

**Механика:** `fatigue_signal = avg_dwell_last_5 < 1_sec AND scroll_speed > 5/sec`. Trigger: inject 2-3 контрастных поста (другой формат, другой топик, social content). Если fatigue продолжается после injection → понизить refresh rate, показать «вы всё посмотрели» CTA.

**Метрики:** `fatigue_recovery_rate`, `session_end_after_fatigue_rate`, `cross_product_navigation_rate`

**Риски:** False positive fatigue (быстрый browsing ≠ усталость); «вы всё посмотрели» может ранить sessions_per_DAU; сложно отличить fatigue от niche-interest search

---

## 8. Сводная приоритизация (ICE)

### По слоям

| Слой | Лучшая гипотеза | ICE | Средний ICE по слою |
|------|----------------|------|---------------------|
| Content Relevance & Quality | R1: Freshness window | 504 | 368 |
| Candidate Generation | CG1: Multi-source retrieval | 504 | 326 |
| Feature Engineering | F1: Dwell normalized | 420 | 317 |
| Target / Loss Design | T1: Multi-objective | 448 | 311 |
| Re-ranking / Post-processing | RR1: Content contrast | 420 | 318 |

### Топ-10 по ICE

| # | Гипотеза | Слой | ICE | Главная метрика |
|---|----------|------|-----|-----------------|
| 1 | R1: Freshness window | Content Quality | 504 | sessions_per_DAU |
| 2 | CG1: Multi-source retrieval | Candidate Gen | 504 | scroll_depth, sessions_per_DAU |
| 3 | T1: Multi-objective optimization | Target/Loss | 448 | Все 4 метрики |
| 4 | F1: Dwell normalized by format | Features | 420 | dwell_time, session_length |
| 5 | R2: Content quality scoring | Content Quality | 420 | feed_exit_rate, dwell_time |
| 6 | RR1: Content contrast penalty | Re-ranking | 420 | scroll_depth, session_length |
| 7 | R4: Trend detection | Content Quality | 378 | sessions_per_DAU |
| 8 | CG2: Exploration quota | Candidate Gen | 378 | novel_topic_discovery |
| 9 | T2: Dwell-time-weighted loss | Target/Loss | 378 | dwell_time, session_length |
| 10 | RR2: Freshness boost in re-ranking | Re-ranking | 378 | sessions_per_DAU |

---

## 9. Рекомендуемый порядок экспериментов

### Фаза 1: Quick wins (re-ranking layer)

Эксперименты на re-ranking — самые быстрые, не требуют переобучения модели.

- **RR1: Content contrast penalty** — 2 недели, A/B 50/50
- **RR2: Freshness boost** — 2 недели, A/B 50/50
- **RR3: Session momentum** — 3 недели, более сложный эксперимент

**Ожидаемый эффект:** +3-5% scroll_depth, +2-3% session_length

### Фаза 2: Feature и Target (ядро модели)

Требует переобучения, но даёт системный эффект.

- **F1: Dwell normalized** + **T2: Dwell-weighted loss** — вместе, 3 недели
- **T1: Multi-objective optimization** — 4-6 недель, cluster randomization
- **F2: Session context features** — 3 недели

**Ожидаемый эффект:** +5-8% dwell_time_per_post, +3-5% session_length

### Фаза 3: Pipeline-изменения (инфраструктурные)

Требуют инженерных инвестиций, но открывают новые возможности.

- **CG1: Multi-source retrieval** — 6-8 недель
- **R1: Freshness window** — 4 недели
- **R2: Content quality scoring** — 6-8 недель (отдельная модель)

**Ожидаемый эффект:** +5-10% sessions_per_DAU, +3-5% timespent/DAU

### Фаза 4: Продвинутые механики

- **T4: Counterfactual learning** — исследовательский, 8+ недель
- **CG3: Session-aware retrieval** — 6 недель
- **F3: Long-term interest features** — 4-6 недель

---

## 10. Риски

| Риск | Вероятность | Влияние | Митигация |
|------|-------------|---------|-----------|
| Conflicting objectives в multi-objective (CTR vs dwell) | Высокая | Высокое | Pareto front, gradual weight tuning, guard metrics |
| Selection bias в training data | Высокая | Среднее | IPS (T4), diversity injection (CG2) |
| Filter bubble от topical relevance | Средняя | Высокое | Exploration quota (CG2), novelty features (F5) |
| Over-engineering pipeline (сложность поддержки) | Средняя | Среднее | Incremental rollout, kill criteria для каждой гипотезы |
| Latency от новых retrieval paths и features | Средняя | Высокое | Performance budgets, feature importance audit |
| Metric gaming (dwell inflation без реальной ценности) | Низкая | Высокое | Quality Engagement Score = Time_Spent x (1 - Unlike_Rate) |

---

## 11. Связь с предыдущим спеком

Гипотезы из спека 2026-07-08, которые вошли в этот документ с развитием:

| Предыдущая | Новая | Развитие |
|------------|-------|----------|
| B3 Freshness-boost | RR2 Freshness boost в re-ranking | Обобщена: не только подписки, все источники |
| B5 Content Contrast | RR1 Content contrast penalty | Расширена: format + topical penalty |
| B1 Dwell Time in Model | T2 Dwell-weighted loss + F1 Dwell normalized | Разделена на feature (F1) и loss (T2) |
| B2 Niche clusters | CG2 Exploration quota + F5 Content embeddings | Декомпозирована на retrieval + feature |

Новые гипотезы (не были в предыдущем спеке): R2-R6, CG1-CG6, F2-F6, T1-T6, RR3-RR6
