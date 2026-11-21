---
tags: [golang, roadmap, fase-4]
fase: 4
status: por-começar
semanas: 3
---

# Fase 04 — HTTP e Stdlib

← [[Fase 03 - Concorrência]] | [[00 - HOME|Home]] | [[Fase 05 - Bases de Dados]] →

> [!abstract] Objectivo
> Construir APIs REST **só com a stdlib**. Se perceberes `net/http`, qualquer framework torna-se trivial.

## M4.1 — Servidor HTTP
- [ ] `http.Handler`, `http.HandlerFunc`, `http.ServeMux`
- [ ] Routing com padrões do Go 1.22+ (`"GET /users/{id}"`, `r.PathValue("id")`)
- [ ] Ler body (`json.NewDecoder`), query params, headers
- [ ] Escrever respostas: status codes, `Content-Type`, JSON
- [ ] Limitar tamanho do body (`http.MaxBytesReader`)

## M4.2 — Middleware
- [ ] Padrão `func(http.Handler) http.Handler`
- [ ] Logging, recover de panics, request-id, CORS
- [ ] Autenticação (Bearer token) como middleware
- [ ] Encadear middlewares (e a ordem importa)
- Ligação: Open/Closed → [[SOLID em Go]]

## M4.3 — Robustez do servidor
- [ ] `http.Server` com timeouts (`ReadHeaderTimeout`, `WriteTimeout`, `IdleTimeout`)
- [ ] **Graceful shutdown** (`signal.NotifyContext` + `srv.Shutdown`)
- [ ] Health endpoints (`/healthz`, `/readyz`)
- [ ] Configuração por env vars (12-factor)

## M4.4 — Design de API REST
- [ ] Recursos, verbos, códigos de estado correctos
- [ ] Paginação (offset vs cursor), filtros, ordenação
- [ ] Formato de erro consistente (ex.: *Problem Details* RFC 9457)
- [ ] Idempotência (`Idempotency-Key`), versionamento
- [ ] Validação de input (`go-playground/validator`)
- [ ] OpenAPI (`oapi-codegen` ou `swaggo/swag`)

## M4.5 — HTTP Client e extras
- [ ] `http.Client` com timeout, reutilização de `Transport`
- [ ] Retry com *backoff + jitter*
- [ ] `httptest` (servidor e recorder)
- [ ] `embed` (ficheiros estáticos), `html/template` (noção)
- [ ] WebSockets / SSE (noção)

## Aulas
- [ ] M4.1 — `colar-link` · *pesquisa: "Go 1.22 net/http routing"*
- [ ] M4.2 — `colar-link` · *pesquisa: "Go http middleware pattern"*
- [ ] M4.3 — `colar-link` · *pesquisa: "Go graceful shutdown http server"*
- [ ] M4.4 — `colar-link`
- [ ] M4.5 — `colar-link`

## Exercícios
- [ ] **EX4.1** — API REST de `tasks` (CRUD) só com stdlib, armazenamento em memória
- [ ] **EX4.2** — Middlewares: logger, recover, request-id, CORS
- [ ] **EX4.3** — Graceful shutdown a terminar requests em curso
- [ ] **EX4.4** — Rate limit por IP como middleware
- [ ] **EX4.5** — HTTP client com retry exponencial + jitter e testes com `httptest`
- [ ] **EX4.6** — Paginação por cursor
- [ ] ** Mini-projecto:** **URL Shortener** (POST cria, GET redirecciona, contador de cliques, expiração)

## Erros comuns
- `http.ListenAndServe` sem timeouts (vulnerável a conexões lentas)
- Esquecer de `defer resp.Body.Close()`
- `http.DefaultClient` sem timeout em produção
- Escrever header **depois** do body

## Dicas
> [!tip] Handlers finos
> O handler faz: decode → chama caso de uso → encode. Lógica de negócio **não** vive aqui (Fase 7).

> [!tip] `Let's Go` (Alex Edwards)
> Livro excelente para web em Go com stdlib — ver [[Livros e Documentação]].

## Critério de conclusão
- [ ] URL Shortener com testes de handlers via `httptest`
- [ ] Shutdown limpo comprovado
- [ ] Respostas de erro com formato único
