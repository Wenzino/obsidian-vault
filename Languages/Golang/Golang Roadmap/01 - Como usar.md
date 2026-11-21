---
tags: [golang, roadmap, meta]
---

# Como usar este vault

← [[00 - HOME|Home]]

## 1. Instalação
1. Descompacta a pasta `Golang Roadmap` **dentro** do teu vault (ou abre-a como vault novo).
2. Plugins recomendados (Settings → Community plugins):
   - **Dataview** — alimenta o [[02 - Dashboard]]
   - **Tasks** — consultas de checkboxes
   - **Templates** (core) ou **Templater** — usa a pasta `90 - Templates`
   - **Spaced Repetition** — revê o [[Flashcards Go]]
   - **Obsidian Git** — backup do progresso num repo privado
   - *(opcional)* **Kanban**, **Excalidraw** para diagramas de arquitectura
3. Settings → Templates → pasta de templates: `90 - Templates`.

## 2. Ritmo semanal sugerido (≈ 10 h)
| Bloco | Tempo | O que fazer |
|---|---|---|
| Aprender | 3 h | Vídeo/leitura de 1–2 milestones |
| Praticar | 5 h | Exercícios + mini-projecto da fase |
| Rever | 2 h | Notas, flashcards, [[Template - Weekly Review]] |

## 3. Fluxo por milestone
1. Marca o milestone como `em-curso` no frontmatter da fase.
2. Vê a aula → cria nota com [[Template - Nota de Aula]] em `50 - Notas/Aulas/`.
3. Cola o link do vídeo em [[YouTube - Inbox de Aulas]] **e** na secção  da fase.
4. Faz os exercícios → regista com [[Template - Exercício]] em `50 - Notas/Exercícios/`.
5. Só marcas `- [x]` quando consegues **fazer sem consultar**.
6. Decisões de arquitectura → [[Template - ADR]] em `60 - Projectos/`.

## 4. Convenções
- Estado da fase (frontmatter `status`): `por-começar` · `em-curso` · `feito`
- IDs: milestones `M<fase>.<n>`, exercícios `EX<fase>.<n>`
- Secções fixas em cada fase: Objectivo, Milestones, Aulas, Exercícios, Erros comuns, Dicas, Critério de conclusão

> [!tip] Regra dos 3 níveis para marcar `[x]`
> 1) Percebi → 2) Consigo repetir olhando → 3) Consigo explicar/fazer **sem olhar**. Só o nível 3 conta.
