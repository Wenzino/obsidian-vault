---
tags: [golang, roadmap, fase-6]
fase: 6
status: por-começar
semanas: 2
---

# Fase 06 — Testes

← [[Fase 05 - Bases de Dados]] | [[00 - HOME|Home]] | [[Fase 07 - Clean Architecture e SOLID]] →

> [!abstract] Objectivo
> Testes como ferramenta de design. Sem esta fase, Clean Architecture não se prova.
> *(Prática transversal: continua a testar em todas as fases seguintes.)*

## M6.1 — Fundamentos
- [ ] `testing`, `*_test.go`, `go test ./...`
- [ ] **Table-driven tests** + `t.Run` (subtests)
- [ ] `t.Helper()`, `t.Cleanup()`, `t.TempDir()`, `t.Parallel()`
- [ ] Packages `x` vs `x_test` (black-box)
- [ ] Cobertura: `-cover`, `-coverprofile` (cobertura ≠ qualidade)

## M6.2 — Asserções e doubles
- [ ] `testify` (`assert`, `require`, `suite`) — ou `go-cmp` + stdlib
- [ ] **Fakes escritos à mão** (preferidos em Go) vs mocks gerados
- [ ] `mockery` / `go.uber.org/mock` quando justificado
- [ ] Testar tempo (injectar `Clock`), aleatoriedade, IDs

## M6.3 — Testes HTTP e integração
- [ ] `httptest.NewRecorder` / `httptest.NewServer`
- [ ] Testes de integração com **`testcontainers-go`** (Postgres real)
- [ ] Separar com build tags / `-short`
- [ ] Golden files

## M6.4 — Avançado
- [ ] **Fuzzing** nativo (`go test -fuzz`)
- [ ] **Benchmarks** (`b.N` / `b.Loop`, `-benchmem`)
- [ ] `go test -race`, `-count`, `-shuffle`
- [ ] Contract tests (noção; usado na Fase 10)
- [ ] Pirâmide: muitos unit, alguns integration, poucos E2E

## M6.5 — TDD
- [ ] Ciclo red → green → refactor
- [ ] Ler/seguir *Learn Go with Tests* ([[Livros e Documentação]])

## Aulas
- [ ] M6.1 — `colar-link` · *pesquisa: "Go table driven tests"*
- [ ] M6.2 — `colar-link`
- [ ] M6.3 — `colar-link` · *pesquisa: "testcontainers-go postgres"*
- [ ] M6.4 — `colar-link` · *pesquisa: "Go fuzzing tutorial"*
- [ ] M6.5 — `colar-link`

## Exercícios
- [ ] **EX6.1** — Kata TDD: numerais romanos
- [ ] **EX6.2** — Testes table-driven para `Conta` (Fase 1) com casos de borda
- [ ] **EX6.3** — Testar handlers do URL Shortener com `httptest`
- [ ] **EX6.4** — Fake `Clock` para testar expiração sem `time.Sleep`
- [ ] **EX6.5** — Teste de integração do repositório Postgres com testcontainers
- [ ] **EX6.6** — Fuzz de um parser (ex.: parser de URL curto / CSV)
- [ ] **EX6.7** — Benchmark: `strings.Builder` vs concatenação

## Erros comuns
- Testar **implementação** em vez de **comportamento**
- Mocks excessivos (testes frágeis)
- `time.Sleep` em testes
- Testes que dependem de ordem ou estado global

## Dicas
> [!tip] Interfaces pequenas = fakes baratos
> Se um fake tem 8 métodos, a interface está grande demais (ISP).

> [!tip] Nomeia casos claramente
> `{name: "saldo insuficiente", ...}` — o output do teste torna-se documentação.

## Critério de conclusão
- [ ] Escrevo testes primeiro num kata sem consultar
- [ ] Suite do projecto corre com `-race` no CI local
- [ ] Teste de integração com Postgres real passa
