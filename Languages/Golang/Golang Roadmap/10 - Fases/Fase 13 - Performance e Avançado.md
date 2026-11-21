---
tags: [golang, roadmap, fase-13, performance]
fase: 13
status: por-começar
semanas: 3
---

# Fase 13 — Performance e Avançado

← [[Fase 12 - Observabilidade, Segurança e DevOps]] | [[00 - HOME|Home]] | [[Fase 14 - Capstone]] →

> [!abstract] Objectivo
> Medir, perceber e optimizar com rigor; conhecer os cantos escuros da linguagem.

## M13.1 — Profiling
- [ ] `pprof` (CPU, heap, goroutines, mutex, block) e `net/http/pprof`
- [ ] `go tool pprof` + flamegraphs
- [ ] `runtime/trace` e `go tool trace`
- [ ] Benchmarks com `-benchmem` e `benchstat`

## M13.2 — Memória e GC
- [ ] Stack vs heap, **escape analysis** (`-gcflags=-m`)
- [ ] Reduzir alocações (pré-alocar slices/maps, `strings.Builder`, `sync.Pool`)
- [ ] GC: `GOGC`, `GOMEMLIMIT`
- [ ] Layout/alinhamento de structs

## M13.3 — Linguagem avançada
- [ ] Reflection (`reflect`) — e porque evitar em hot paths
- [ ] `unsafe` (só para perceber; evitar)
- [ ] Geração de código (`go generate`, templates, AST básico)
- [ ] `cgo` (custos) e WebAssembly (noção)
- [ ] Iteradores e padrões modernos de generics

## M13.4 — Sistemas
- [ ] Servidor TCP/UDP com `net`
- [ ] Protocolos binários e `encoding/binary`
- [ ] Ler código da stdlib: `net/http`, `sync`, `context`
- [ ] Ler *100 Go Mistakes* ([[Livros e Documentação]]) e anotar os que já cometeste

## Aulas
- [ ] M13.1 — `colar-link` · *pesquisa: "Go pprof profiling tutorial"*
- [ ] M13.2 — `colar-link` · *pesquisa: "Go escape analysis", "Go garbage collector"*
- [ ] M13.3 / M13.4 — `colar-link`

## Exercícios
- [ ] **EX13.1** — Optimizar um hot path com pprof (antes/depois com `benchstat`)
- [ ] **EX13.2** — Cache **LRU genérico** thread-safe + benchmarks
- [ ] **EX13.3** — Gerador de carga HTTP concorrente (mini `wrk`/`vegeta`)
- [ ] **EX13.4** — Servidor de chat TCP com broadcast
- [ ] **EX13.5** — Key-Value store em memória com persistência (WAL simples)
- [ ] **EX13.6** — Gerar código com `go generate` (ex.: enums → `String()`)
- [ ] **EX13.7** — Resumir 10 erros de *100 Go Mistakes* com exemplos teus
- [ ] ** Mini-projecto:** Reduzir p99 do Wallet/e-commerce em ≥ 30% com evidências de profiling

## Erros comuns
- Optimizar sem medir
- Micro-benchmarks que o compilador elimina
- `sync.Pool` como cache de longa duração
- Reflection/`unsafe` por "performance" sem prova

## Dicas
> [!tip] Ordem de ganhos típica
> Algoritmo/estrutura de dados → I/O e queries → alocações → micro-otimizações.

## Critério de conclusão
- [ ] Relatório de optimização com números reais antes/depois
- [ ] Sei ler um flamegraph e explicar o gargalo
