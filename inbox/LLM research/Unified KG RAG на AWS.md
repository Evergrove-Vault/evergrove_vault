---
tags:
  - article
  - inbox
  - llm-research
source: blog
url: https://aws.amazon.com/blogs/opensource/unified-knowledge-graph-rag-on-aws-graphrag-and-lightrag-on-one-stack/
author: "Jonas Kim, Ji Hyeon Kang (AWS)"
date: 2026-09-14
novelty: новое
hub: "preparation: graph DB + inference: retrieval"
checked: 2026-09-28
---
# 📝 Overview
Открытый фреймворк ([awslabs/unified-kg-rag-on-aws](https://github.com/awslabs/unified-kg-rag-on-aws), Apache-2.0), который реализует **и GraphRAG, и LightRAG на одном стеке AWS**: один конвейер индексации, три вида поиска (лексический, семантический, графовый) и восемь стратегий ответа, которые можно сравнивать на своих данных. Ценность — не в новом алгоритме, а в честном сравнении «качество ↔ цена ↔ задержка» и в инженерных решениях: хеши, lineage, чекпоинты.

> [!important] Чего нет в моих заметках
> В [[graph RAG]] и [[GraphRAG (Microsoft)]] нет сравнения режимов поиска по цене и нет практики «один индекс — несколько стратегий». Кроме того, нет тройного гибрида «BM25 + kNN + обход графа» с объединением через RRF.

---

# 💡 Архитектура
| Задача | Сервис |
|---|---|
| LLM, эмбеддинги, реранкинг | Amazon Bedrock |
| Граф сущностей и связей | Amazon Neptune (Gremlin) |
| BM25 + kNN | Amazon OpenSearch Service |
| Реестр документов (дельта) | Amazon DynamoDB (опционально) |
| Корпус и кэш конвейера | Amazon S3 |

**Конвейер из 12 стадий** с контрольной точкой после каждой: `document_parsing`, `document_loading`, `text_chunking`, `translation`, `graph_extraction`, `gleaning`, `graph_resolution`, `claim_extraction`, `claim_resolution`, `graph_analysis`, `community_detection`, `indexing`. Возобновление — флагами `--pipeline-id` и `--resume-from-stage`.

**Тройной гибрид:** каждый запрос объединяет три ранжированных списка — BM25 (OpenSearch), kNN по эмбеддингам Bedrock и расширение по графу Neptune — через Reciprocal Rank Fusion, затем опционально реранкинг Bedrock. Ср. [[методы Reranking#Reciprocal Rank Fusion (RRF)]] и [[retrieval#Hybrid Retrieval (Weighted scoring)]].

**Стратегии поиска:** GraphRAG — `simple`, `local`, `global`, `drift`, `auto`; LightRAG — `mix`, `hybrid`, `naive`.

**Обновления:** реестр хешей документов, идемпотентные upsert, lineage для удаления — [[Инкрементальная индексация - хеши, идемпотентность, мемоизация]], [[Provenance графа и безопасное удаление документов]].

# 📊 Замер стратегий
Условия: MuSiQue и 2WikiMultihopQA, по 100 вопросов, метрика token-F1, среднее по трём запускам; Claude Sonnet 4.5 и Titan Text Embeddings V2, цены август 2026, us-west-2; стоимость измерялась отдельными прогонами по 20 вопросов. Разницу менее ≈0,05 авторы предлагают считать шумом.

| Стратегия | Метод | MuSiQue | 2Wiki | Цена за 1000 вопросов | Медианный ответ |
|---|---|---|---|---|---|
| hybrid | LightRAG | 0,634 | 0,628 | $42,29 | 19,8 с |
| mix | LightRAG | 0,602 | 0,654 | $38,67 | 23,6 с |
| local | GraphRAG | 0,519 | 0,577 | $5,23 | 6,5 с |
| drift | GraphRAG | 0,379 | 0,541 | $7,99 | 12,0 с |
| naive | вектор | 0,354 | 0,424 | $8,56 | 7,3 с |
| global | GraphRAG | 0,231 | 0,396 | $66,22 | 17,3 с |
| simple | GraphRAG | 0,209 | 0,374 | $7,41 | 5,1 с |

Выводы автора: «hybrid gains about 0.12 F1 over local at eight times the cost and three times the latency»; `local` даёт примерно 82% точности `hybrid` за восьмую часть цены; `global` уместен, когда «a corpus-wide narrative is itself the deliverable», а лучший ответ — `mix`. Низкий результат `global` на этих бенчмарках объясняется тем, что вопросы требуют одиночных именованных значений, а сводка сообществ пишется «отбрасыванием» деталей.

**Проверка верности реализаций:** оба upstream-проекта установлены на тот же корпус и вопросы. Различия по LightRAG лежат внутри статистического шума; GraphRAG `local` в этом фреймворке лучше upstream (0,519/0,577 против 0,404/0,471), потому что он передаёт модели исходные пассажи (17% контекста против 4%), а upstream — однострочные описания графа. Это тот же вывод, что и в [[EffiRAG и цена структуры графа]].

**Неожиданная деталь:** суммаризация сообществ составила лишь 7,6% стоимости индексации на одном корпусе, потому что «the dominant cost is extracting entities and relationships from every document, which both methodologies need».

# ⚠️ Ограничения
- Оба бенчмарка *специально* требуют переходов между документами и «flatter graph retrieval by construction»; на вашем корпусе порядок изменится.
- 100 вопросов на бенчмарк, экстрактивные ответы; для обзорных вопросов метрика невыгодна `global`.
- Цены привязаны к Bedrock и региону.

# 🧭 Где встроить в HUB RAG
Блок «Retrieval» получает правило маршрутизации *по цене*: по умолчанию `local`-подобный поиск, дорогие режимы — по запросу. Расширение [[retrieval#Routing Retrieval]]: классификатор выбирает не только тип поиска, но и бюджет.

---

# 📚 Связанные материалы
**Оригиналы**
- [Unified Knowledge Graph RAG on AWS](https://aws.amazon.com/blogs/opensource/unified-knowledge-graph-rag-on-aws-graphrag-and-lightrag-on-one-stack/) — AWS Open Source Blog, 14.09.2026
- [Репозиторий](https://github.com/awslabs/unified-kg-rag-on-aws)

**Заметки волта**
- [[GraphRAG (Microsoft)]], [[LightRAG]], [[LazyGraphRAG]], [[graph RAG]]
- [[Когда граф окупается - GraphRAG-Bench и RAGSearch]]
