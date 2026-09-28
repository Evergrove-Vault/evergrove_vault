---
tags:
  - article
  - inbox
  - llm-research
source: arxiv
url: https://arxiv.org/abs/2506.05690
author: "Zhishang Xiang et al. (GraphRAG-Bench); Haoyu Han et al. (RAG vs GraphRAG); Dongzhe Fan et al. (RAGSearch)"
date: 2026-04-01
novelty: новое
hub: "inference: retrieval + preparation: graph DB (выбор архитектуры)"
checked: 2026-09-28
---
# 📝 Overview
Сводка независимых сравнений «обычный RAG против GraphRAG» и «граф против агентного поиска». Общий вывод: правильный вопрос звучит не «нужен ли граф», а «какие вопросы, какой корпус, какая цена индексации и запроса». В заметках волта сравнение есть только качественное (в [[sparse подходы к анализу текстов]] — «graph: дорого строить», в [[GraphRAG (Microsoft)]] — «стоимость»); количественных критериев выбора нет.

> [!important] Чего нет в моих заметках
> В [[graph RAG]] перечислены варианты графов, но нет ответа на вопрос «когда граф вообще стоит строить». Ниже — то, что известно из бенчмарков 2025–2026 годов.

---

# 💡 Что говорят исследования
1. **GraphRAG-Bench** ([arXiv:2506.05690](https://arxiv.org/abs/2506.05690), ICLR 2026): «recent studies report that GraphRAG frequently underperforms vanilla RAG on many real-world tasks». Бенчмарк содержит задачи возрастающей сложности — факты, сложные рассуждения, контекстная суммаризация, творческая генерация — и оценивает весь конвейер от построения графа до ответа.
2. **RAG vs. GraphRAG** ([arXiv:2502.11371](https://arxiv.org/abs/2502.11371), Han et al.): систематическое сравнение на вопросно-ответных задачах и суммаризации по запросу; предлагаются гибриды — *Selection* (маршрутизация запроса в RAG или GraphRAG по типу) и *Integration* (объединение свидетельств обоих).
3. **RAGSearch** ([arXiv:2604.09666](https://arxiv.org/abs/2604.09666), апрель 2026): агентный поиск с многошаговым обращением к базе «substantially improves dense RAG and narrows the performance gap to GraphRAG, particularly in RL-based settings». Но GraphRAG «remains advantageous for complex multi-hop reasoning, exhibiting more stable agentic search behavior when its offline cost is amortized». Отдельно измеряются цена предобработки, эффективность онлайна и стабильность.
4. **Цена на практике** ([[Unified KG RAG на AWS]]): на многошаговых вопросах графовые стратегии заметно точнее векторной (`local` 0,519/0,577 против 0,354/0,424), а `local` при этом стоит дешевле векторной в их замере ($5,23 против $8,56 за 1000 вопросов); `hybrid` точнее ещё на 0,12, но в 8 раз дороже. Бенчмарки специально сделаны «многошаговыми», поэтому граф в них выглядит лучше, чем на типичном корпусе.
5. **Минимум структуры** ([[EffiRAG и цена структуры графа]]): лучший компромисс — «minimum graph structure needed to connect relevant source evidence».

# 🛠 Чек-лист выбора (моя сводка из перечисленного)
| Признак задачи | Вероятный выбор |
|---|---|
| Фактоидные вопросы по одному документу | гибрид BM25 + плотный поиск + реранкинг, граф не нужен |
| Многошаговые вопросы через несколько документов | граф как указатель ([[LinearRAG - граф без извлечения отношений]], [[HippoRAG 2]]) или агентный поиск |
| Вопросы «о чём корпус в целом» | сводки сообществ (global) или [[LazyGraphRAG]] |
| Корпус меняется часто | сначала стоимость обновления: [[TG-RAG - временной граф знаний]], [[TagRAG - иерархия тегов вместо сообществ]], [[LightRAG]] |
| Бюджет жёсткий | «граф ищет, отвечает исходный текст», без сводок сообществ |
| Вопросы разной природы | маршрутизация *Selection* по типу вопроса ([[retrieval#Routing Retrieval]]) |

Измерять надо пять величин: качество ответа, цена индексации, цена запроса, задержка, стабильность.

# ⚠️ Ограничения
Каждое исследование использует свои датасеты и модели; выводы про порядок методов нельзя переносить на свой корпус без собственного замера ([[оценка качества RAG]]).

# 🧭 Где встроить в HUB RAG
Развилка перед блоком «для graph DB»: «строить ли граф вообще» и внутри «Retrieval»: маршрутизация *Selection* между векторным и графовым поиском.

---

# 📚 Связанные материалы
**Оригиналы**
- [When to use Graphs in RAG: A Comprehensive Analysis for Graph RAG](https://arxiv.org/abs/2506.05690) — GraphRAG-Bench, [репозиторий](https://github.com/GraphRAG-Bench/GraphRAG-Benchmark)
- [RAG vs. GraphRAG: A Systematic Evaluation and Key Insights](https://arxiv.org/abs/2502.11371)
- [Do We Still Need GraphRAG? Benchmarking RAG and GraphRAG for Agentic Search Systems](https://arxiv.org/abs/2604.09666)

**Заметки волта**
- [[graph RAG]], [[GraphRAG (Microsoft)]], [[retrieval]], [[agentic RAG]], [[hybrid RAG методы реализации часть 1]]
- [[Search-R1 - RL для поисковых агентов]] — RL-агентный поиск как альтернатива графу
