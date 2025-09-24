---
aliases:
  - GraphRAG
---
# Краткое описание

**GraphRAG** — открытый проект Microsoft Research для граф-ориентированного Retrieval-Augmented Generation. Основная идея: при индексировании корпус преобразуется в граф знаний (узлы — сущности/утверждения/фрагменты текста, рёбра — отношения), затем поиск и генерация ответов комбинируют графовую навигацию, community-summaries и классические retrieval-подходы (sparse/dense).

---

# Основные возможности

- **Построение графа знаний из неструктурированного текста**
    - извлечение сущностей, отношений и ключевых утверждений с помощью LLM;
    - представление как локальных mini-графов (TextUnit → node) и объединение в глобальный граф.
- **Кластеризация/сообщества (community detection)**
    - алгоритм Leiden для обнаружения тематических сообществ;
    - для каждого сообщества генерация краткого summary (summary per community).
- **Гибридные режимы поиска**
    - **Global (Whole-dataset reasoning)** — опрос summary сообществ для вопросов высокого уровня;
    - **Local (Entity-based reasoning)** — извлечение сущностей из запроса → traversal соседних узлов → сбор релевантных текстов;
    - **DRIFT (iterative / primer → follow-up)** — комбинированный, интерактивный режим: сначала общий ответ + уточняющие вопросы → далее локальные уточнения.
- **Provenance / traceability**
    - каждый ответ снабжается ссылками на исходные куски текста (chunk IDs, URL, quotes).
- **Поддержка multi-hop и аргументированной агрегации**
    - распределённые утверждения (claims) и рёбра support/attack позволяют строить ответы, опираясь на цепочки фактов.
- **Инструменты / примеры из репозитория**
    - примеры pipeline, YAML-конфиги для индексации, интеграция с NetworkX/Neo4j/Weaviate в примерах.

---

# Ограничения и риски

- **Высокая вычислительная стоимость** — множество LLM-вызовов при построении графа и генерации summaries.
- **Необходимость реиндексирования** при потоковой / «живой» подсекции данных: изменения корпуса требуют перестройки community summaries/графа.
- **Чувствительность к качеству извлечения** — ошибки NER/RE приводят к некорректным связям/галлюцинациям.
- **Масштабируемость** — при очень плотных графах (много рёбер, «hub»-сущности) нужно фильтровать и нормализовать (stop-entities), иначе traversal/quality падают.
- **Сложность настройки промптов** для разных доменов и тонкая балансировка параметров (clustering, summary length, traversal depth).

---

# Архитектура и pipeline (технически)

**Основные этапы:**
1. **Ingest / Preprocessing**
    - входы: plain text / CSV / JSON / PDF → unified DataFrame `{id, title, text, metadata}`.
    - очистка, language detection, metadata extraction.
2. **Semantic splitting**
    - разбиение на TextUnits (с учётом заголовков/абзацев/смысловых границ); параметры chunk_size/overlap настраиваемы.
3. **LLM-based extraction**
    - для каждого TextUnit: извлечь сущности, отношения, claims, краткое summary → создать узлы и ребра.
4. **Graph construction**
    - объединение узлов, нормализация сущностей (aliasing), создание глобального графа (в памяти: NetworkX / Pandas) или выгрузка в GraphDB (Neo4j/FalkorDB) для scale.
5. **Community detection & summarization**
    - кластеризация (Leiden), генерация community summaries (LLM) → индекс summaries.
6. **Indexing**
    - dense embeddings (векторная БД) и/или sparse представления для быстрых lookup; связи graph ↔ text chunks сохраняются в provenance.
7. **Query processing (runtime)**
    - Query routing: выбрать Global / Local / DRIFT режим.
    - Global: опрос summaries (map-reduce); Local: entity extraction → retrieve nodes → expand neighbours → collect chunks.
    - Rerank (cross-encoder / MMR) → assemble context → LLM генерация ответа + validation (self-RAG / CRAG style).
8. **Provenance & trace**
    - для каждого output сохраняются: used nodes, used chunks, prompts, LLM responses, timestamps, source URLs, confidence scores.

---

# Режимы retrieval — подробности
- **Global (summary-first)**
    - для high-level / overview вопросов; быстрый because operates on compact summaries; хорошо для «What are the top 5 themes in corpus».
- **Local (entity-first)**
    - для factoid и relation questions; извлечение сущностей из запроса → subgraph expansion (1–N hops) → retrieve chunks for those nodes.
- **DRIFT (hybrid iterative)**
    - primer: краткий ответ из summaries; follow-up: генерация уточняющих вопросов/запросов → локальный поиск и расширение; полезен для chain-of-thought style interactions.

---

# Интеграция в стэк (реализации и примеры)
- **LLM / prompt orchestration**
    - GraphRAG использует LLM для extraction и summarization; легко интегрируется с OpenAI / Azure OpenAI / local LLMs через стандартные SDK.
- **LangChain / LangGraph**
    - Можно встроить GraphRAG как компонент в цепочку: есть примеры использования LLM→Cypher (GraphCypherQAChain), а LangGraph/GraphState удобно хранит промежуточные результаты и маршрутизирует retrieval между graph и vector.
- **GraphDB**
    - локальный: NetworkX (прототип), переход на Neo4j / ArangoDB / FalkorDB для production traversal и cypher/gremlin запросов.
- **Vector DB**
    - Weaviate / Milvus / Qdrant / FAISS — для hybrid retrieval (entity embedding lookup + chunk lookup).
- **Monitoring / tracing**
    - LangSmith / custom logging для traceability (prompts, LLM responses, provenance), сбор «bad cases».

---

# Требования к данным и инфраструктуре

- **Данные**
    - корпус текстов (plain/CSV/JSON/PDF). Рекомендуется: `id`, `title`, `text`, `metadata` (date, source, license).
    - нет жёсткой необходимости внешнего NER/NEL — GraphRAG использует LLM для извлечения; можно усилить с помощью внешних KB (Wikipedia) для нормализации.
- **Инфраструктура**
    - LLM-инфраструктура (API или GPU кластеры) — много LLM вызовов при индексировании и при генерации summaries.
    - Хранилище графа: для прототипа — in-memory (NetworkX), для production — Neo4j/ArangoDB/FalkorDB.
    - Vector DB для dense retrieval (Weaviate/Milvus/Qdrant) — опционально.
    - Хранилище provenance и версий (S3 + metadata DB).
- **Ресурсы**
    - CPU/GPU для LLM и embedding; достаточно RAM/SSD для графа и индексов; планировать бюджет на LLM вызовы.

---

# Применимость к аргументированному мышлению и multi-hop reasoning
- **Аргументный слой**: GraphRAG извлекает claims/premises и создаёт support/attack рёбра → естественная структура для[[ argument mining]] и for/against summaries.
- **Multi-hop**: traversal по рёбрам + aggregation через community summaries позволяет соединять факты из разных документов → даёт устойчивую базу для multi-hop ответов.
- **Graph-of-Thought**: DRIFT и iterative primer→follow-up задают структуру, близкую к Tree/Graph-of-Thought: сначала high-level hypothesis, затем локальная проверка через подграф и уточнения.
- **Практические преимущества**
    - объяснимые ответы с ссылками на доказательства;
    - возможность показать pro/contra и confidence per claim;
    - удобство для юридических/научных обзоров, где нужна трассируемость доводов.

---

# Практические рекомендации (best practices)
- **Start small**: сначала prototype с NetworkX + small LLM model (для extraction), затем переносить в Neo4j/VectorDB по мере роста.
- **Hybrid retrieval**: всегда держать dense/sparse fallback — graph не заменяет быстрый lookup по терминам.
- **Stop-entities & pruning**: фильтровать «универсальные» сущности (dates, common nouns) чтобы уменьшить шум и degree-explosion.
- **Provenance at every step**: хранить input chunks → extracted claims → summaries → prompts → LLM outputs.
- **Incremental updates**: если corpus живой, проектировать ingestion с возможностью partial reindex (update node/edge rather than full rebuild).
- **Evaluation**: проверять faithfulness, recall@k, path-relevance (процент полезных reasoning paths), human eval на аргументы.
