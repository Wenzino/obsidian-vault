---
tags: [golang, roadmap, fase-3]
fase: 3
status: por-começar
semanas: 3
---

# Fase 03 — Concorrência

← [[Fase 02 - Intermédio]] | [[00 - HOME|Home]] | [[Fase 04 - HTTP e Stdlib]] →

> [!abstract] Objectivo
> Usar goroutines e channels com segurança — sem *leaks*, *races* nem *deadlocks*. É o que faz Go brilhar em backend.

## M3.1 — Goroutines e channels
- [ ] `go func()`, ciclo de vida, `sync.WaitGroup`
- [ ] Channels: unbuffered vs buffered, `close`, `range`, direcção (`<-chan`, `chan<-`)
- [ ] `select`, `default`, timeouts com `time.After`
- [ ] Quem **fecha** o channel? (sempre o produtor)

## M3.2 — Sincronização
- [ ] `sync.Mutex`, `RWMutex`, `Once`, `Pool`
- [ ] `sync/atomic`
- [ ] Channel vs mutex: quando usar cada um
- [ ] Race detector: `go test -race`, `go run -race`

## M3.3 — `context`
- [ ] `WithCancel`, `WithTimeout`, `WithDeadline`
- [ ] `ctx` como **primeiro parâmetro**, nunca guardado em struct
- [ ] Propagação de cancelamento até BD/HTTP
- [ ] `context.WithValue` — uso mínimo (request-id), nunca para dependências

## M3.4 — Padrões
- [ ] Worker pool
- [ ] Fan-out / fan-in
- [ ] Pipeline
- [ ] Semáforo com channel buffered
- [ ] `errgroup` (`golang.org/x/sync/errgroup`)
- [ ] Rate limiter (token bucket, `x/time/rate`)

## M3.5 — Diagnóstico
- [ ] Goroutine leaks (como detectar)
- [ ] Deadlocks e o erro "all goroutines are asleep"
- [ ] Noção do scheduler (G-M-P) e `GOMAXPROCS`
- [ ] Modelo de memória (*happens-before*) — nível conceptual

## Aulas
- [ ] M3.1 — `colar-link` · *pesquisa: "Go concurrency patterns Rob Pike"*
- [ ] M3.2 — `colar-link`
- [ ] M3.3 — `colar-link` · *pesquisa: "Go context package deep dive"*
- [ ] M3.4 — `colar-link` · *pesquisa: "Go worker pool fan-in fan-out"*
- [ ] M3.5 — `colar-link` · *pesquisa: "Go scheduler GMP"*

## Exercícios
- [ ] **EX3.1** — Fetcher concorrente de N URLs com timeout global e por request
- [ ] **EX3.2** — Worker pool que processa 1000 jobs com N workers
- [ ] **EX3.3** — Pipeline: gerar → quadrado → somar
- [ ] **EX3.4** — Rate limiter *token bucket* feito à mão
- [ ] **EX3.5** — Pub/Sub em memória (subscrever, publicar, cancelar)
- [ ] **EX3.6** — Hasher paralelo de ficheiros de uma pasta (`errgroup`)
- [ ] **EX3.7** — Código com *data race* → detectar com `-race` e corrigir
- [ ] **EX3.8** — Provocar e corrigir um goroutine leak
- [ ] ** Mini-projecto:** *Web crawler* concorrente com limite de profundidade, de concorrência e cancelamento por `context`

## Erros comuns
- Lançar goroutine **sem saber como/quando termina**
- Mutex copiado por valor
- Enviar para channel que ninguém lê (leak)
- Usar `context.Background()` no meio da cadeia de um request

## Dicas
> [!tip] Concorrência não é paralelismo
> Estrutura primeiro (concorrência), velocidade depois (paralelismo). Mede antes.

> [!tip] Começa síncrono
> Escreve a versão sem goroutines, testa, só depois paraleliza.

## Critério de conclusão
- [ ] Crawler passa `go test -race ./...`
- [ ] Explico como cancelar uma árvore de goroutines
- [ ] Sei quando um mutex é mais simples que um channel
