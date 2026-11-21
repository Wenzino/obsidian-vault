---
tags: [golang, roadmap, fase-8, frameworks]
fase: 8
status: por-começar
semanas: 3
---

# Fase 08 — Frameworks e Ecossistema

← [[Fase 07 - Clean Architecture e SOLID]] | [[00 - HOME|Home]] | [[Fase 09 - Modular Monolith]] →

> [!abstract] Objectivo
> Conhecer as ferramentas de maior impacto na comunidade e **escolher com critério**. Regra: o framework fica no **adapter**, nunca no domínio.
> Tabela de escolhas por defeito → [[Ferramentas e Libs]]

## M8.1 — Routers / frameworks HTTP
- [ ] **`net/http` (1.22+)** — baseline; muitas vezes basta
- [ ] **chi** — router leve, 100% compatível com `net/http`
- [ ] **Gin** — o mais popular; binding, middleware, ecossistema enorme
- [ ] **Echo** — API limpa, bom suporte a OpenAPI/middleware
- [ ] **Fiber** — inspirado no Express; baseado em `fasthttp` (**não** compatível com `net/http`)
- [ ] Noção de **Huma** (OpenAPI-first) e **Connect** (gRPC-compatível sobre HTTP)
- [ ] Matriz de decisão preenchida (ver exercício)

## M8.2 — Validação, auth e segurança
- [ ] `go-playground/validator`
- [ ] `golang-jwt/jwt` (JWT) e fluxo access/refresh
- [ ] OIDC/OAuth2 (`golang.org/x/oauth2`, `coreos/go-oidc`)
- [ ] Autorização: RBAC/ABAC (`casbin` — noção)

## M8.3 — Dados
- [ ] Aprofundar **sqlc + pgx** (defeito) e **GORM** / **Ent**
- [ ] Migrações: `goose` / `golang-migrate` / `atlas`
- [ ] Redis: `go-redis`

## M8.4 — Configuração, CLI e logging
- [ ] **cobra** (CLI) e **viper**/**koanf** (config)
- [ ] `log/slog` (defeito), **zap** e **zerolog** (alto desempenho)
- [ ] Hot reload com **air**

## M8.5 — Jobs e agendamento
- [ ] **asynq** (Redis) ou **river** (Postgres)
- [ ] `gocron` / cron simples
- [ ] Idempotência de jobs

## M8.6 — Documentação e geração de código
- [ ] OpenAPI: `oapi-codegen` (spec-first) ou `swaggo/swag` (code-first)
- [ ] `go generate`, `stringer`, `mockery`
- [ ] Protobuf + **buf** (preparação para a Fase 10)

## Aulas
- [ ] M8.1 chi — `colar-link`
- [ ] M8.1 Gin — `colar-link`
- [ ] M8.1 Echo — `colar-link`
- [ ] M8.1 Fiber — `colar-link`
- [ ] M8.2 — `colar-link` · *pesquisa: "Go JWT authentication refresh token"*
- [ ] M8.4 / M8.5 / M8.6 — `colar-link`

## Exercícios
- [ ] **EX8.1** — Implementar a **mesma API de tasks** em chi, Gin e Echo (reutilizando o mesmo use case!)
- [ ] **EX8.2** — Preencher matriz de decisão (abaixo) + ADR ([[Template - ADR]])
- [ ] **EX8.3** — Autenticação JWT com refresh token e middleware
- [ ] **EX8.4** — Spec OpenAPI + código gerado com `oapi-codegen`
- [ ] **EX8.5** — CLI `cobra` com subcomandos `serve`, `migrate`, `seed`
- [ ] **EX8.6** — Job assíncrono (email falso) com retries via `asynq`/`river`
- [ ] ** Mini-projecto:** Wallet Service com chi + JWT + OpenAPI + CLI de migrações

### Matriz de decisão (preenche)
| Critério | net/http | chi | Gin | Echo | Fiber |
|---|---|---|---|---|---|
| Compatível com `net/http` | | | | | |
| Curva de aprendizagem | | | | | |
| Middleware/ecossistema | | | | | |
| Desempenho (medido por ti) | | | | | |
| Testabilidade | | | | | |
| Risco de lock-in | | | | | |

## Erros comuns
- Importar `gin.Context` nos use cases (vazar framework)
- Escolher por *benchmarks de marketing* sem medir o teu caso
- Ignorar compatibilidade com `net/http` (ecossistema de middleware)
- Adoptar ORM e deixar entidades do ORM virarem entidades de domínio

## Dicas
> [!tip] Teste de lock-in
> Se trocar o framework HTTP obriga a mexer fora de `adapter/http`, a arquitectura está a vazar.

> [!tip] Para a maioria dos backends
> `chi` (ou stdlib) + `pgx` + `sqlc` + `slog` + `validator` é um ponto de partida sólido e fácil de manter.

## Critério de conclusão
- [ ] Mesma API em 3 routers com **zero** alterações em domain/app
- [ ] ADR de escolha escrito e justificado
- [ ] JWT + refresh a funcionar com testes
