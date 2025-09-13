---
tags:
  - article
  - evergrow/bud
aliases:
  - chunking
  - методы chunking
source: freedium
url: https://freedium.cfd/https://ai.gopubby.com/21-chunking-strategies-for-rag-f28e4382d399
author: "[@dataprincess](https://medium.com/@dataprincess)"
---
# 📝 Overview  
Обзорная статья на методы разделения информации для построения чанков для [[MOC RAG|RAG]]
Рассказывает о том, с какими сложностями можно столкнуться при создании механизма Retrieve в системе RAG.
На практике чтобы заставить механизм доставать действительно полезную информацию часто надо перепробовать следующие способы:
- tweaking chunks
- switching embedding models
- swapping out retrievers
- fine-tuning rankers
- rewriting prompts

---

# 💡 Полезные заметки  
## Основные идеи и мысли:
- нужно правильно классифицировать тип поступающей информации для того, чтобы его корректно обработать своим особым образом 
- нужно проследить, чтобы обработка мультимодальных запросов была одними и теми же инструментами (как для запроса, так и для информации в RAG)
- часть техник требует тщательного подбора параметров под конкретный домен информации

---

# 🛠 Инструменты / подходы  

>[!info] Данный список содержит всю информацию из статьи с моими уточнениями и связкой с дополнительной информацией. **Исходник читать смысла не имеет**

## 1. Naïve chunking (split by newline) 
Подходит, когда текст будет равномерно разделен переносом строки (заметки, списки, FAQ, логи)

>[!attention] Будь внимателен к длине получившихся чанков, она может быть как очень большой, так и слишком маленькой 

## 2. Fixed-size (fixed window chunking)
Подходит для больших, слабо структурированных текстов. Так же, когда не уверен в качестве распознавания данного текста или же в тексте отсутствует пунктуация 

## 3. Sliding window chunking
Делим на чанки с пересечением. Это позволяет лучше связывать контекст чанков между собой.

Подходит для текстов с длинными предложениями, эссе и других свободных форматов описания.

## 4. Sentence-based chunking
Разбиение по целиковым предложениям 

Подходит для документации, статей из блогов и других хорошо структурированных текстов

## 5. Paragraph-based chunking
Если нужен более широкий контекст, нежели чем в [[21 Chunking Strategies for RAG#4. Sentence-based chunking|Sentence-based chunking]]. Может хорошо сочетаться с ним при помощи техники [[building the Entire RAG Ecosystem and Optimizing Every Component#Hierarchical Indexing (RAPTOR) Knowledge Tree|RAPTOR]]

Подходит для документации, статей из блогов и других хорошо структурированных текстов

## 6. Page-based chunking
Так же хорошо подходит для [[building the Entire RAG Ecosystem and Optimizing Every Component#Hierarchical Indexing (RAPTOR) Knowledge Tree|RAPTOR]]. Для того, чтобы не терять детали контекста стоит использовать технику [[building the Entire RAG Ecosystem and Optimizing Every Component#Multi-Representation Indexing|Multi-Representation Indexing]] для генерации суммаризации контекста. 

 Подходит для документов с явным оглавлением или же слайдов и скорее всего сложных многостраничных схем, когда речь заходит о [[MOC RAG#Hybrid RAG|Hybrid RAG]], [[hybrid RAG методы реализации часть 1#Multimodal RAG|Multimodal RAG]].

>[!tip] Полезным дополнением будет сохранять номер страницы и источник информации для дальнейшей обработки 
> 

## 7. Structured chunking
Выделяем отдельные компоненты полу структурированных данных, таких ка логи, csv, json, html

## 8. Document-Based Chunking:
Разделение по частям (разделам) структурированных текстов (по типу данной заметки или мануалов) 
Так же хорошо подходит для [[building the Entire RAG Ecosystem and Optimizing Every Component#Hierarchical Indexing (RAPTOR) Knowledge Tree|RAPTOR]]. Для того, чтобы не терять детали контекста стоит использовать технику [[building the Entire RAG Ecosystem and Optimizing Every Component#Multi-Representation Indexing|Multi-Representation Indexing]] для генерации суммаризации контекста. 

## 9. Keyword-based chunking
Разделение по ключевым словам, которые часто обозначают начало новой секции в неявном виде 

## 10. Entity-based chunking
Использует [[Name Entity Recognition|NER]] для поиска сущностей в тексте. Далее берется контекст вокруг данной сущности, связанный по смыслу. Это можно сделать с помощью [[semantic text segmentation]]

Подходит для новостные статьи, юридические контракты, тематические исследования (специфических документов) или сценарии фильмов.

>[!attention] Будь внимателен к тому, как изначальные куски из которых собирается чанк  мапятся на границы предложений. 
>

## 11. Token-based chunking
Обычно эту технику целесообразно сочетать с разбиением на фрагменты предложений, чтобы избежать разделения предложений таким образом, что теряется контекст.

Подходит для неструктурированных документов без заголовков и абзацев.

## 12. Topic-based chunking
Развитие идеи с [[semantic text segmentation]] только теперь мы ищем не последовательно, а во всем тексте и используем алгоритмы кластеризации получившихся векторов 

Подходит для документов, содержащих неявно несколько тем 

## 13. Table-aware chunking

Вы можете выделять и разбивать таблицы отдельно в формате JSON или Markdown. Это может быть построчно, по столбцам или вся таблица целиком.
Дополнительные методы преобразования таблиц смотри [[hybrid RAG методы реализации часть 1#Табличные данные|тут]]

## 14. Content-aware chunking

Подход с настройкой чанкинга в зависимости от типа контекста внутри сложного документа.

## 15. Contextual chunking
Использование LLM для генерации дополнительного контекста. Походит на [[building the Entire RAG Ecosystem and Optimizing Every Component#Multi-Representation Indexing|Multi-Representation Indexing]]

## 16. [[semantic text segmentation|Semantic similarity-based chunking]]

## 17. Recursive chunking
Рекурсивное применение методик с уменьшением размера чанков. 

Подходит для текстов с неравномерной или непредсказуемой длиной предложений, таких как интервью, речи или тексты в свободной форме.

## 18. Embedding chunking 
Тут не понял чем от пункта 16 отличается 

## 19. Agentic / LLM-based chunking
Может использоваться в [[agentic RAG]], однако очень большой вопрос о стоимости данного подхода, а так же его качестве и устойчивости 

## 20. [[building the Entire RAG Ecosystem and Optimizing Every Component#Hierarchical Indexing (RAPTOR) Knowledge Tree|Hierarchical chunking]]


---

# 📚 Связанные материалы  
## Библиотеки и витрины 
- [chonkie](https://github.com/SecludedCoder/chonkie)
- [bRAG-langchain](https://github.com/bragai/bRAG-langchain) базовые имплементации RAG
- [chunking_strategies_langchain](https://github.com/deepshamenghani/chunking_strategies_langchain)
- [advanced-chunker](https://github.com/rango-ramesh/advanced-chunker)
- [chunkit](https://github.com/hypergrok/chunkit)  Turn URLs into LLM-friendly markdown chunks
---

# 🙂 Мои мысли  
Статья интересная, однако в ней есть техники, которые очень схожи и выделять их в отдельные пункты выглядит как высасывание из пальца. Кроме того, тут почти нет подходов к мультимодальным медиа. 
Кроме того тут нет кода, что было бы полезно, но как обзор и отправная точка вполне пойдет 

