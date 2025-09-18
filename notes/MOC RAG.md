---
tags:
  - MOC
---
# Notes
[[HUB RAG.canvas|HUB RAG]]
## Base RAG
- [bRAG-langchain](https://github.com/bragai/bRAG-langchain) Implementations
- [[building the Entire RAG Ecosystem and Optimizing Every Component]]
- [[интеграция RAG и MCP]]
- [[оценка качества RAG]]
- [[21 Chunking Strategies for RAG]]
- [[методы Reranking]]
## Hybrid RAG
- [[hybrid RAG методы реализации часть 1]]
## Agentic RAG
- [[agentic RAG]]

## Graph RAG
- [[graph RAG]]
- [[GraphRAG (Microsoft)]]
- [[Graph Neural Network|GNN]]
# Connected MOC
- [[]]

# Все связанные статьи
```dataviewjs
function collectLinks(page, visited = new Set()) {
    if (visited.has(page)) return [];
    visited.add(page);

    let p = dv.page(page);
    if (!p) return [];

    let links = p.file.outlinks.map(l => l.path);
    let result = [page];
    for (let link of links) {
        result.push(...collectLinks(link, visited));
    }
    return result;
}

let root = dv.current().file.path;
let all = collectLinks(root);
dv.list(all.map(p => dv.fileLink(p)));
```
