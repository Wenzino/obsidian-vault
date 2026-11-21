---
tags: [golang, roadmap, dashboard]
---

# Dashboard de Progresso

← [[00 - HOME|Home]]

## Estado das fases (requer Dataview)
```dataview
TABLE fase AS "Fase", status AS "Estado", semanas AS "Semanas"
FROM "10 - Fases"
SORT fase ASC
```

## Tarefas por fazer nas fases (requer Tasks)
```tasks
not done
path includes 10 - Fases
limit 25
```

## Notas recentes
```dataview
TABLE file.mtime AS "Modificado"
FROM "50 - Notas"
SORT file.mtime DESC
LIMIT 10
```

## Registo manual (funciona sem plugins)
| Fase | Início | Fim | Confiança (1-5) | Notas |
|---|---|---|---|---|
| [[Fase 00 - Setup e Mentalidade]] | | | | |
| [[Fase 01 - Fundamentos]] | | | | |
| [[Fase 02 - Intermédio]] | | | | |
| [[Fase 03 - Concorrência]] | | | | |
| [[Fase 04 - HTTP e Stdlib]] | | | | |
| [[Fase 05 - Bases de Dados]] | | | | |
| [[Fase 06 - Testes]] | | | | |
| [[Fase 07 - Clean Architecture e SOLID]] | | | | |
| [[Fase 08 - Frameworks e Ecossistema]] | | | | |
| [[Fase 09 - Modular Monolith]] | | | | |
| [[Fase 10 - Microservices]] | | | | |
| [[Fase 11 - Mensageria e Event-Driven]] | | | | |
| [[Fase 12 - Observabilidade, Segurança e DevOps]] | | | | |
| [[Fase 13 - Performance e Avançado]] | | | | |
| [[Fase 14 - Capstone]] | | | | |
