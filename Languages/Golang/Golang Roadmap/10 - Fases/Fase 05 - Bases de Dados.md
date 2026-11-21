---
tags: [golang, roadmap, fase-5]
fase: 5
status: por-começar
semanas: 3
---

# Fase 05 — Bases de Dados

← [[Fase 04 - HTTP e Stdlib]] | [[00 - HOME|Home]] | [[Fase 06 - Testes]] →

> [!abstract] Objectivo
> Persistência com PostgreSQL de forma segura e testável: SQL puro, geração de código (sqlc), ORMs e cache.

## M5.1 — `database/sql` e pgx
- [ ] `sql.DB` é um **pool** (`SetMaxOpenConns`, `SetMaxIdleConns`, `SetConnMaxLifetime`)
- [ ] `QueryContext`, `QueryRowContext`, `ExecContext`
- [ ] `Scan`, `rows.Close()`, `rows.Err()`
- [ ] Driver/toolkit **pgx** (`pgxpool`)
- [ ] NULLs (`sql.Null*` / tipos pgx)
- [ ] Queries parametrizadas (nunca concatenar strings)

## M5.2 — Transacções
- [ ] `BeginTx`, `Commit`, `Rollback` (com `defer`)
- [ ] Níveis de isolamento
- [ ] Locks: `SELECT ... FOR UPDATE`, optimistic locking (coluna `version`)
- [ ] Padrão para transacção atravessar vários repositórios (Unit of Work)

## M5.3 — Migrações
- [ ] `golang-migrate`, `goose` ou `atlas`
- [ ] Migrações *up/down* versionadas e no CI
- [ ] Estratégia sem downtime (expand/contract)

## M5.4 — Acesso a dados: escolhas
- [ ] **sqlc** — SQL → código Go tipado (recomendado como defeito)
- [ ] **sqlx** — extensões leves sobre `database/sql`
- [ ] **GORM** — ORM mais popular (conveniência vs previsibilidade)
- [ ] **Ent** — ORM por *schema-as-code*; **Bun** — alternativa SQL-first
- [ ] Padrão **Repository** (interface no consumidor, implementação no adapter)
- [ ] Problema N+1, índices, `EXPLAIN ANALYZE`

## M5.5 — Cache e outros stores
- [ ] Redis com `go-redis`: cache-aside, TTL, invalidação
- [ ] Distributed lock (noção) e rate limit com Redis
- [ ] MongoDB (`mongo-go-driver`) — noção, quando faz sentido
- [ ] Supabase/Postgres gerido: ligação por pooler, RLS (noção)

## Aulas
- [ ] M5.1 — `colar-link` · *pesquisa: "Go database/sql connection pool"*
- [ ] M5.2 — `colar-link`
- [ ] M5.3 — `colar-link`
- [ ] M5.4 — `colar-link` · *pesquisa: "sqlc Go tutorial", "GORM vs sqlc"*
- [ ] M5.5 — `colar-link` · *pesquisa: "go-redis cache aside"*

## Exercícios
- [ ] **EX5.1** — `docker-compose` com Postgres + Redis
- [ ] **EX5.2** — CRUD de `tasks` com `database/sql` + pgx
- [ ] **EX5.3** — O mesmo CRUD com **sqlc**
- [ ] **EX5.4** — O mesmo com **GORM** e escrever 3 linhas: o que ganhaste/perdeste
- [ ] **EX5.5** — Transferência entre contas numa transacção com `FOR UPDATE` + teste de concorrência
- [ ] **EX5.6** — Migrações versionadas aplicadas no arranque (ou por comando)
- [ ] **EX5.7** — Cache-aside com Redis e invalidação
- [ ] ** Mini-projecto:** Evoluir o URL Shortener para Postgres + Redis (cache de redirecções)

## Erros comuns
- Esquecer `rows.Close()` → fuga de conexões
- Pool sem limites → esgotar o Postgres
- Transacção sem `Rollback` em caso de erro
- Misturar tipos do ORM com o **domínio** (Fase 7!)

## Dicas
> [!tip] Define o repositório pelo que o caso de uso precisa
> `Save`, `FindByID`, `ListByUser` — não "CRUD genérico".

> [!tip] Testa contra Postgres real
> Usa `testcontainers-go` (Fase 6) em vez de mockar SQL.

## Critério de conclusão
- [ ] Transferência segura sob concorrência (teste prova)
- [ ] Migrações reproduzíveis do zero
- [ ] Consigo justificar sqlc vs GORM para um projecto
