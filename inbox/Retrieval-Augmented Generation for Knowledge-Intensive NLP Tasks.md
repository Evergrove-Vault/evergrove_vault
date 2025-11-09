---
tags:
  - article
  - todo
  - inbox
source:
url:
author:
---
# 📝 Overview  
[[MOC RAG]]
Large pre-trained language models have been shown to store factual knowledge in their parameters, and achieve state-of-the-art results when fine-tuned on downstream NLP tasks. However, their ability to access and precisely manipulate knowledge is still limited, and hence on knowledge-intensive tasks, their performance lags behind task-specific architectures. Additionally, providing provenance for their decisions and updating their world knowledge remain open research problems. Pre-trained models with a differentiable access mechanism to explicit non-parametric memory can overcome this issue, but have so far been only investigated for extractive downstream tasks. We explore a general-purpose fine-tuning recipe for retrieval-augmented generation (RAG) -- models which combine pre-trained parametric and non-parametric memory for language generation. We introduce RAG models where the parametric memory is a pre-trained seq2seq model and the non-parametric memory is a dense vector index of Wikipedia, accessed with a pre-trained neural retriever. We compare two RAG formulations, one which conditions on the same retrieved passages across the whole generated sequence, the other can use different passages per token. We fine-tune and evaluate our models on a wide range of knowledge-intensive NLP tasks and set the state-of-the-art on three open domain QA tasks, outperforming parametric seq2seq models and task-specific retrieve-and-extract architectures. For language generation tasks, we find that RAG models generate more specific, diverse and factual language than a state-of-the-art parametric-only seq2seq baseline


https://arxiv.org/abs/2005.11401

Краткое содержание статьи (о чём, зачем, основные мысли).  
- Основная идея:  
- Ключевые тезисы:  

---

# 💡 Полезные заметки  
Конкретные выделенные идеи, цитаты, наблюдения.  
-  
-  
> Можно выносить отдельные заметки в формат [[]] Название заметки для атомарных идей.

---

# 🛠 Инструменты / подходы  
Перечень упомянутых инструментов, библиотек, методов.  
- [ ] Название (ссылка) — чем полезен  
-  

---

# 📚 Связанные материалы  

[[]] если это цикл статей  

- [[]] Название статьи/книги — контекст или развитие идеи  
- Ссылка на внешние ресурсы  
 
---

# 🙂 Мои мысли  
Личное отношение, выводы, что можно применить на практике.  
-  
-  

---