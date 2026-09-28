---
tags:
  - article
  - inbox
  - llm-research
source: blog
url: https://blog.getzep.com/how-zep-tracks-provenance-in-agent-memory/
author: "Daniel Chalef (Zep); Jonas Kim, Ji Hyeon Kang (AWS)"
date: 2026-07-14
novelty: новое
hub: "preparation: graph DB"
checked: 2026-09-28
---
# 📝 Overview
Как связать каждый узел и ребро графа знаний с документами-источниками так, чтобы **удаление документа убирало только его собственные артефакты, а общие сущности оставались**. Это не то же самое, что «в ответе есть ссылка на чанк»: тут провенанс нужен самому графу для обслуживания.

> [!important] Чего нет в моих заметках
> [[GraphRAG (Microsoft)]] говорит про provenance для ответов (chunk IDs, quotes), а [[graph RAG#Dynamic Graph Update]] — про обновление узлов при новых фактах. Про то, что происходит со связями при **удалении** и **слиянии** сущностей, ничего нет.

---

# 💡 Правила, которые сформулировали источники

**Zep / Graphiti** (блог Даниэля Чалефа, 14.07.2026):
1. У ребра есть список `episodes` — *все* эпизоды (сообщения, чанки документов, JSON-записи), которые привели к этому факту. Одиночная ссылка на источник, по словам автора, «holds in deterministic ETL and breaks under generative extraction».
2. **Слияние сущностей** («J. Smith» + «John Smith»): «the merged node inherits the episode associations from both sides». Потеряете любую из сторон — «facts remain in the graph with no path back to the episode that produced them».
3. **Повторное подтверждение** факта добавляет эпизод в список, а не перезаписывает его.
4. **Противоречие**: Zep фиксирует, какой эпизод инвалидировал факт и когда; инвалидированный факт остаётся в графе с историей (подробнее — [[Bi-temporal факты - Zep и Graphiti]]).
5. Каскадное удаление эпизода — `remove_episode`.

**AWS-фреймворк** (14.09.2026): реестр документов ведёт lineage каждого артефакта, поэтому «deleting a document removes only the graph/index artifacts exclusive to it — shared entities survive».

**CocoIndex**: lineage «end-to-end», каждая запись, вектор или узел графа прослеживается до исходного байта источника.

# 🛠 Минимальная модель данных (иллюстрация)
```
узел/ребро:  sources = {doc_A, doc_B}
удалить doc_A:
   для каждого артефакта, где doc_A ∈ sources → убрать doc_A
   если sources пусто → удалить артефакт
   иначе → артефакт остаётся (общая сущность)
слияние двух узлов:  sources(новый) = sources(a) ∪ sources(b)
```

# ⚠️ Открытые вопросы
- **Производные тексты.** Описания сущностей и community-сводки собраны LLM из нескольких документов. После удаления одного из них устаревшая формулировка может остаться внутри описания, даже если `sources` уже почищен. В источниках про пересборку описаний при удалении не сказано; это надо проектировать отдельно (пересуммаризация затронутых узлов, ср. [[TagRAG - иерархия тегов вместо сообществ]], где описания при слиянии переписываются).
- **Устаревание не равно удалению.** FalkorDB прямо предупреждает: автоматическое снятие устаревших или вытесненных фактов «is currently a roadmap item, not a shipped feature», нужен ручной шаг инвалидации, см. [[GraphRAG SDK (FalkorDB) - онтология и изоляция тенантов]].
- **Аудит и безопасность.** Тот же провенанс нужен, чтобы отвечать «откуда взялся факт» и ограничивать доверие к источникам: [[Безопасность RAG - poisoning, ACL и происхождение данных]].

# 🧭 Где встроить в HUB RAG
Блок «для graph DB» (после IE, NER, RE): на этапе записи узла/ребра сохранять множество источников; при слиянии — объединять множества. Для vector DB достаточно метаданных чанка (документ, версия, хеш).

---

# 📚 Связанные материалы
**Оригиналы**
- [How Zep tracks provenance in agent memory](https://blog.getzep.com/how-zep-tracks-provenance-in-agent-memory/) — 14.07.2026, изменено 24.07.2026
- [Graphiti (README)](https://github.com/getzep/graphiti) — «every entity and relationship traces back to the episodes (raw data) that produced it»
- [Unified Knowledge Graph RAG on AWS](https://aws.amazon.com/blogs/opensource/unified-knowledge-graph-rag-on-aws-graphrag-and-lightrag-on-one-stack/)

**Заметки волта**
- [[graph RAG]], [[GraphRAG (Microsoft)]], [[information extraction]], [[Relation Extraction]]
- [[Инкрементальная индексация - хеши, идемпотентность, мемоизация]]
- [[Entity resolution - эмбеддинги и LLM-судья]] — слияние сущностей как источник потери provenance
