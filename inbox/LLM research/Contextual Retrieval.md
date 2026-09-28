---
tags:
  - article
  - inbox
  - llm-research
source: blog
url: https://www.anthropic.com/engineering/contextual-retrieval
author: "Anthropic"
date: 2024-09-19
novelty: углубление
hub: "preparation: chunking / vector DB + sparse"
checked: 2026-09-28
---
# 📝 Overview
**Contextual Retrieval** от Anthropic: перед индексацией к каждому чанку **приписывается короткий (50–100 токенов) LLM-сгенерированный контекст**, объясняющий, где этот чанк находится в документе. Контекст добавляется и к тексту для эмбеддинга (*Contextual Embeddings*), и к тексту для BM25 (*Contextual BM25*). Вместе с реранкингом это снижает долю неудачных извлечений на 67%.

> [!important] Что уже есть и чего не хватает
> В [[21 Chunking Strategies for RAG#15. Contextual chunking]] записано одной строкой: «использование LLM для генерации дополнительного контекста, похоже на Multi-Representation Indexing». Это *углубление*: у Anthropic есть конкретный промпт, замер по стадиям и экономика через кэш промптов. Отличие от [[building the Entire RAG Ecosystem and Optimizing Every Component#Multi-Representation Indexing]]: там суммаризация *заменяет* чанк при поиске, здесь контекст *дописывается* к чанку.

---

# 💡 Как это работает
**Проблема.** Чанк «Выручка компании выросла на 3% по сравнению с прошлым кварталом» не говорит, о какой компании и о каком периоде, поэтому и находится, и используется плохо.

**Промпт** (из статьи):
```
<document> {{WHOLE_DOCUMENT}} </document>
Here is the chunk we want to situate within the whole document
<chunk> {{CHUNK_CONTENT}} </chunk>
Please give a short succinct context to situate this chunk within the overall
document for the purposes of improving search retrieval of the chunk.
Answer only with the succinct context and nothing else.
```
**Цена.** Благодаря prompt caching документ кэшируется и переиспользуется для всех его чанков; разовая стоимость — $1,02 за миллион токенов документов (оценка Anthropic).

**Индекс.** Контекстуализированный чанк идёт и в эмбеддинги, и в BM25 (см. [[sparse подходы к анализу текстов]]). Для финальной выдачи — реранкер (Cohere): берутся top-150, реранкер оставляет top-20 ([[методы Reranking#Cross-Encoder Re-Ranking]]).

# 📊 Результаты (доля неудачных извлечений среди top-20)
| Вариант | Доля неудач | Снижение |
|---|---|---|
| Обычные эмбеддинги | 5,7% | — |
| Contextual Embeddings | 3,7% | 35% |
| Contextual Embeddings + Contextual BM25 | 2,9% | 49% |
| То же + реранкинг | 1,9% | 67% |

Лучшие эмбеддинги в их тесте — Gemini и Voyage.

# 🛠 Рекомендации автора
- Если база знаний **меньше 200 000 токенов** (около 500 страниц), проще положить её целиком в промпт с кэшированием, чем строить RAG.
- Подбирать границы чанков, эмбеддер, кастомный промпт контекстуализации под домен, число чанков (top-20 оказался лучшим) и «always run evals».

# ⚠️ Ограничения
- Стоимость и задержка индексации растут на каждый чанк (один LLM-вызов); при обновлении документа контексты придётся пересчитывать ([[Инкрементальная индексация - хеши, идемпотентность, мемоизация]]).
- Цифры получены на подборке корпусов Anthropic, не на вашем домене.
- Сравнение с late chunking: [[Late Chunking]] — контекстуализация лучше сохраняет смысл, но дороже.

# 🧭 Где встроить в HUB RAG
Группа «для vector DB» + «sparse методы»: это один из немногих приёмов, который улучшает **обе** ветки — и dense, и BM25 (в canvas они разнесены). После него блок «Пост процессинг чанков» (reranking) работает на лучших кандидатах.

---

# 📚 Связанные материалы
**Оригиналы**
- [Contextual Retrieval in AI Systems](https://www.anthropic.com/engineering/contextual-retrieval) — Anthropic, 19.09.2024

**Заметки волта**
- [[21 Chunking Strategies for RAG]], [[hybrid RAG методы реализации часть 1]], [[методы Reranking]]
- [[Late Chunking]], [[Proposition indexing - Dense X Retrieval]]
- [[Cache-Augmented Generation (CAG) Is Here To Replace RAG]] — идея «положить всё в контекст с кэшем»
