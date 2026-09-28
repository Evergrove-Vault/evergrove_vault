---
tags:
  - article
  - inbox
  - llm-research
source: arxiv
url: https://arxiv.org/abs/2409.04701
author: "Michael Günther, Isabelle Mohr, Daniel James Williams, Bo Wang, Han Xiao (Jina AI)"
date: 2024-09-07
novelty: новое
hub: "preparation: chunking / vector DB"
checked: 2026-09-28
---
# 📝 Overview
**Late chunking** меняет порядок операций: вместо «сначала режем текст, потом эмбеддим каждый кусок» — сначала пропускаем через трансформер **весь длинный документ**, получаем эмбеддинги токенов, и только *после* трансформера, перед mean pooling, разбиваем их на чанки. Каждый чанковый вектор «видит» весь документ. Работает без дополнительного обучения, но требует эмбеддера с длинным контекстом (у Jina — 8192 токенов).

> [!important] Чего нет в моих заметках
> В [[21 Chunking Strategies for RAG]] 21 способ, и все они меняют *границы чанков*. Late chunking не меняет границы: он меняет то, *как считается вектор чанка*. В списке из 21 такого нет — рядом стоит только [[21 Chunking Strategies for RAG#15. Contextual chunking]], который решает ту же проблему другим способом (см. [[Contextual Retrieval]]).

---

# 💡 Как это работает
**Проблема** (пример Jina со статьёй про Берлин): в чанке вида «население города составляет N человек» слова «город» и «его» относятся к Берлину, названному только в первом предложении; вектор такого чанка не знает про Берлин, и вопрос «какое население у Берлина?» находит его плохо: название города и цифра не встречаются в одном чанке.

**Три шага**
1. Весь текст (до 8192 токенов ≈ десять страниц) целиком идёт через слои трансформера.
2. Получаются контекстные эмбеддинги каждого токена.
3. Для каждого чанка усредняются (mean pooling) векторы токенов, попавших в его границы.

Итоговые векторы — «conditional», а не независимые; при этом «boundary cues» (где резать) по-прежнему нужны.

# 📊 Результаты (блог Jina, BEIR, nDCG@10)
| Датасет | Средняя длина документа | Naive chunking | Late chunking | Без чанкинга |
|---|---|---|---|---|
| SciFact | 1498 симв. | 64,20 | **66,10** | 63,89 |
| TRECCOVID | 1117 | 63,36 | 64,70 | 65,18 |
| FiQA2018 | 767 | 33,25 | 33,84 | 33,43 |
| NFCorpus | 1590 | 23,46 | 29,98 | 30,40 |
| Quora | 62 | 87,19 | 87,19 | 87,19 |

Во всех случаях late chunking лучше naive; чем длиннее документ, тем больше выигрыш. На Quora (короткие тексты) разницы нет — как и следовало ожидать.

**Независимое сравнение** ([Reconstructing Context](https://arxiv.org/abs/2504.19754), ECIR 2025 workshop): contextual retrieval лучше сохраняет смысловую связность, но требует больше вычислений; late chunking эффективнее, но «tends to sacrifice relevance and completeness».

# ⚠️ Ограничения
- Нужен эмбеддер с длинным контекстом; документ длиннее окна придётся делить, и на границах окон эффект теряется.
- Vector DB и retriever должны хранить именно эти «контекстные» чанковые векторы, а не пересчитывать их из текста.
- Для BM25 и других sparse-методов не подходит: у них нет векторов токенов (вывод из устройства метода). Для sparse-ветки контекст добавляет [[Contextual Retrieval]].

# 🧭 Где встроить в HUB RAG
Группа «для vector DB», блок chunking: вместо/вместе с [[21 Chunking Strategies for RAG#3. Sliding window chunking]] (перекрытие даёт контекст лишь соседним чанкам, late chunking — всему документу). Хорошо сочетается со структурной нарезкой (по заголовкам, [[21 Chunking Strategies for RAG#6. Page-based chunking]]).

---

# 📚 Связанные материалы
**Оригиналы**
- [Late Chunking: Contextual Chunk Embeddings Using Long-Context Embedding Models](https://arxiv.org/abs/2409.04701) — arXiv, v1 07.09.2024, v3 07.07.2025
- [Late Chunking in Long-Context Embedding Models](https://jina.ai/news/late-chunking-in-long-context-embedding-models/) — Jina AI, 22.08.2024
- [Reconstructing Context: Evaluating Advanced Chunking Strategies for RAG](https://arxiv.org/abs/2504.19754)

**Заметки волта**
- [[21 Chunking Strategies for RAG]], [[semantic text segmentation]]
- [[Contextual Retrieval]], [[Matryoshka и квантизация эмбеддингов]]
- [[building the Entire RAG Ecosystem and Optimizing Every Component#Multi-Representation Indexing]] — другой способ сохранить контекст (сводки)
