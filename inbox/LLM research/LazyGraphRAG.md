---
tags:
  - article
  - inbox
  - llm-research
source: blog
url: https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/
author: "Darren Edge, Ha Trinh, Jonathan Larson (Microsoft Research)"
date: 2024-11-25
novelty: новое
hub: "preparation: graph DB + inference: retrieval"
checked: 2026-09-28
---
# 📝 Overview
**LazyGraphRAG** — вариант GraphRAG, который **откладывает все LLM-вызовы на момент запроса**. При индексации LLM не используется вообще: концепты и их со-встречаемость извлекаются обычным NLP (noun phrases), из них строится граф и иерархия сообществ. Стоимость индексации, по данным Microsoft, равна стоимости векторного RAG и составляет **0,1%** от полного GraphRAG.

> [!important] Чего нет в моих заметках
> В [[GraphRAG (Microsoft)#Ограничения и риски]] главный риск — «высокая вычислительная стоимость». LazyGraphRAG — прямой ответ Microsoft на этот риск, и в заметках он не описан.

---

# 💡 Как это работает

## Индексация
- Извлечение концептов и их со-встречаемости обычным NLP (noun phrases), без LLM.
- Оптимизация графа по его статистике, выделение иерархии сообществ.
- **Нет** LLM-суммаризации сообществ.

## Запрос
Комбинация двух поисковых стратегий, которая идёт итеративно:
- **best-first**: ранжирование чанков по эмбеддингам (как в векторном RAG);
- **breadth-first**: LLM-оценщик на уровне предложений проверяет релевантность top-k ещё не проверенных чанков; цепочка углубляется в подсообщества, если несколько подряд сообществ не дали ни одного релевантного чанка («iterative deepening»).
- Единственный параметр управления — **relevance test budget**: им плавно регулируется баланс «цена ↔ качество».

# 📊 Результаты (по блогу Microsoft)
- Индексация: «identical to vector RAG and 0.1% of the costs of full GraphRAG».
- Для глобальных запросов — более чем в **700 раз** ниже стоимость запроса при сопоставимом качестве, чем у GraphRAG Global Search.
- При **4%** стоимости запроса GraphRAG global search LazyGraphRAG, по словам авторов, «significantly outperforms all competing methods» на локальных и глобальных запросах.
- Заметка редакторов от 06.06.2025: технология вошла в публичное превью Microsoft Discovery и Azure Local.
- Сами авторы отмечают, что гибрид «предсуммаризованный индекс GraphRAG + поиск в стиле LazyGraphRAG» может оказаться лучше чистого LazyGraphRAG.

# ⚠️ Ограничения
Стоимость не исчезает, а переносится в запрос: для частых одинаковых глобальных вопросов предвычисленные сводки могут окупаться (AWS-блог показывает обратную сторону: у обычного GraphRAG суммаризация сообществ составила лишь 7,6% стоимости индексации на одном корпусе, см. [[Unified KG RAG на AWS]]). Все цифры — от вендора.

# 🧭 Где встроить в HUB RAG
Блок «для graph DB»: вариант «граф без LLM при индексации» вместе с [[LinearRAG - граф без извлечения отношений]]. В блоке «reasoning» — родственник режима DRIFT ([[GraphRAG (Microsoft)#Режимы retrieval — подробности]]): оба чередуют локальную проверку и расширение.

---

# 📚 Связанные материалы
**Оригиналы**
- [LazyGraphRAG: Setting a new standard for quality and cost](https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/) — Microsoft Research, 25.11.2024

**Заметки волта**
- [[GraphRAG (Microsoft)]], [[graph RAG]], [[LightRAG]], [[HippoRAG 2]]
- [[EffiRAG и цена структуры графа]], [[Когда граф окупается - GraphRAG-Bench и RAGSearch]]
