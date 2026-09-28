---
tags:
  - article
  - inbox
  - llm-research
source: arxiv
url: https://arxiv.org/abs/2403.14403
author: "Soyeong Jeong et al. (Adaptive-RAG); Zhengbao Jiang et al. (FLARE); Zhuowan Li et al. (Self-Route)"
date: 2024-03-21
novelty: новое
hub: "inference: routing + retrieval"
checked: 2026-09-28
---
# 📝 Overview
Три работы решают одну задачу с разных сторон: **не тратить retrieval там, где он не нужен, и не жалеть его там, где нужен**. *Adaptive-RAG* выбирает стратегию по сложности вопроса, *FLARE* запускает поиск в момент, когда модель во время генерации сомневается, *Self-Route* решает, отвечать по найденным чанкам или по полному длинному контексту.

> [!important] Что уже есть и чего не хватает
> [[retrieval#Routing Retrieval]] выбирает *какой ретривер* (по правилам: короткий запрос → sparse, про отношения → graph). Self-RAG в [[building the Entire RAG Ecosystem and Optimizing Every Component#**Self-RAG (Self-Reflective RAG)**]] содержит «адаптивный retrieval», но как одну из рефлексий. Отдельных приёмов «сколько поиска потратить» и «когда идти в поиск» нет.

---

# 💡 Три приёма
**Adaptive-RAG** ([arXiv:2403.14403](https://arxiv.org/abs/2403.14403), NAACL 2024). Маленькая языковая модель-классификатор предсказывает *сложность* вопроса и выбирает: без поиска, одношаговый поиск или итеративный многошаговый. Метки для обучения собираются автоматически — по результатам самих стратегий и по устройству датасетов. Идея: простые вопросы не нужно гонять через дорогой многошаговый конвейер, а сложные — отвечать по одному поиску.

**FLARE** ([arXiv:2305.06983](https://arxiv.org/abs/2305.06983), EMNLP 2023). «Forward-Looking Active REtrieval»: модель генерирует *следующее предложение* заранее; если в нём есть токены с низкой уверенностью, предложение (или его маскированный вариант) служит запросом, находятся документы и предложение перегенерируется. То есть поиск запускается посреди длинного ответа, когда модель «нервничает», а не один раз на входе.

**Self-Route** ([arXiv:2407.16833](https://arxiv.org/abs/2407.16833), EMNLP 2024, industry). Сравнение RAG и моделей с длинным контекстом (LC): «when resourced sufficiently, LC consistently outperforms RAG» по среднему качеству, но RAG заметно дешевле. Метод: модели предлагают ответить по найденным чанкам и при необходимости сказать, что вопрос неотвечаем; только такие вопросы отправляются в LC. Результат авторов: стоимость заметно ниже при качестве, сопоставимом с LC.

# ⚠️ Ограничения
- Классификатор сложности нужно обучать/подбирать; ошибка классификации ведёт либо к пропуску нужного поиска, либо к лишним расходам.
- FLARE зависит от калибровки уверенности модели (тот же вопрос — в [[Sufficient Context - достаточность контекста и отказ от ответа]]).
- Self-Route требует доступной LC-модели и делает два прохода для трудных запросов.

# 🧭 Где встроить в HUB RAG
Блок «routing» (до «предобработки запроса»): к правилам `sparse/dense/graph` добавляется ось *бюджета* — «без поиска / одношаговый / многошаговый / полный контекст». В «Self-Correction» — триггер «дозапросить» по уверенности токенов (FLARE). Экономику «положить всё в контекст» описывает и [[Contextual Retrieval]] (порог 200 000 токенов), и [[Cache-Augmented Generation (CAG) Is Here To Replace RAG]].

---

# 📚 Связанные материалы
**Оригиналы**
- [Adaptive-RAG: Learning to Adapt Retrieval-Augmented LLMs through Question Complexity](https://arxiv.org/abs/2403.14403), код [starsuzi/Adaptive-RAG](https://github.com/starsuzi/Adaptive-RAG)
- [Active Retrieval Augmented Generation (FLARE)](https://arxiv.org/abs/2305.06983), код [jzbjyb/FLARE](https://github.com/jzbjyb/FLARE)
- [Retrieval Augmented Generation or Long-Context LLMs? A Comprehensive Study and Hybrid Approach](https://arxiv.org/abs/2407.16833)

**Заметки волта**
- [[retrieval]], [[agentic RAG]], [[маршрутизация llm агентов]], [[Semantic Router]]
- [[Microsoft AI Introduces CoRAG]] — итеративный retrieval, обученный на цепочках
