Тут описываются часть подходов по совмещению извлечения информации из разных типов хранения ([[sparse подходы к анализу текстов|sparse]], [[building the Entire RAG Ecosystem and Optimizing Every Component#Стратегии индексирования|dense]], [[graph RAG|graph]]). Все эти методы можно сочетать с [[reasonin методы построения размышлений модели|reasoning]] и [[building the Entire RAG Ecosystem and Optimizing Every Component#Трансформации запросов|техниками трансформации запросов]]

# retrieval подходы 
## Hybrid Retrieval (Weighted scoring)

- Один retriever считает sparse score, другой dense score.
- Итоговый ранг = `α * dense_score + β * sparse_score`.
- Подходит для простых задач без сложного графа.
- SPLADE — хороший пример: сочетает sparse + dense прямо в embedding.

---

## Graph-aware Retrieval

1. Запрос сначала идёт в **граф**, чтобы определить релевантные узлы (темы, сущности).
2. Потом для найденных узлов мы берём их тексты и прогоняем через **sparse+dense retrievers**.
3. Генерация идёт по объединённому контексту.

Пример:

- Вопрос: _«Как связаны квантовая теория поля и Хиггсовский механизм?»_
- Graph: находит связь между узлом _«QFT»_ и _«Higgs mechanism»_.
- Sparse: подтянет статьи с терминами «QFT», «Higgs».
- Dense: найдёт объяснения «field theory» ↔ «Higgs boson».
- LLM получит смесь фактов и текстов.

---

## Routing Retrieval

- Классификатор (или LLM) решает, какой retriever использовать.
- Пример правил:
    - Запрос короткий, с цифрами → sparse.
    - Запрос длинный, семантический → dense.
    - Запрос про отношения → graph.
- Можно комбинировать: если уверенности мало → запускаем [[методы Reranking#Reciprocal Rank Fusion (RRF)|fusion]].

---

## Iterative Retrieval (CoT/ToT + retrievers)

- Система начинает с dense-поиска → видит, что данных мало → запускает sparse. или [[building the Entire RAG Ecosystem and Optimizing Every Component#HyDE (Hypothetical Document Embeddings)|HyDE]]
- Потом делает reasoning в графе, чтобы уточнить связи.
- Итог: многошаговый поиск (retrieval + reasoning).