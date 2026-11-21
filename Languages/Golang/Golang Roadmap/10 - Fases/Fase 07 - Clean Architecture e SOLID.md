---
tags: [golang, roadmap, fase-7, arquitectura]
fase: 7
status: por-começar
semanas: 4
---

# Fase 07 — Clean Architecture e SOLID

← [[Fase 06 - Testes]] | [[00 - HOME|Home]] | [[Fase 08 - Frameworks e Ecossistema]] →

> [!abstract] Objectivo
> Aplicar **SOLID** e **Clean/Hexagonal Architecture** em Go de forma **idiomática**: pouca cerimónia, fronteiras claras, dependências a apontar para dentro.
> Cheatsheets: [[SOLID em Go]] · [[Clean Architecture em Go]]

## M7.1 — Dependency Rule e camadas
- [ ] **Domain**: entidades, value objects, erros de domínio (sem imports externos)
- [ ] **Application / Use cases**: orquestração + **ports** (interfaces)
- [ ] **Adapters / Infra**: HTTP, Postgres, mensageria, clientes externos
- [ ] **Composition root**: `cmd/<app>/main.go` liga tudo
- [ ] Setas de dependência sempre → domínio

## M7.2 — SOLID traduzido para Go
- [ ] **S**RP — um pacote/struct, uma razão para mudar
- [ ] **O**CP — extensão por interfaces e *decorators* (middleware)
- [ ] **L**SP — implementações respeitam o **contrato** (erros, nil, semântica)
- [ ] **I**SP — interfaces mínimas, **definidas no consumidor**
- [ ] **D**IP — use case depende de port, não de Postgres
- [ ] Onde Go **desencoraja** abstracção prematura (e como não exagerar)

## M7.3 — Injecção de dependências
- [ ] DI manual por construtor (`NewService(repo, clock, log)`) — **defeito**
- [ ] Quando considerar `google/wire` (compile-time) ou `uber-go/fx`/`dig` (runtime)
- [ ] Evitar singletons e variáveis globais
- [ ] Config tipada e injectada (não `os.Getenv` espalhado)

## M7.4 — Domínio rico (DDD táctico, versão leve)
- [ ] Entities vs Value Objects
- [ ] Invariantes protegidos por construtores/métodos (campos não exportados)
- [ ] Aggregates e fronteira transaccional
- [ ] Domain events (noção)
- [ ] Linguagem ubíqua nos nomes dos pacotes/tipos

## M7.5 — Fronteiras e mapeamento
- [ ] DTOs de transporte ≠ entidades de domínio ≠ modelos de BD
- [ ] Mapear erros: domínio → HTTP (`ErrNotFound` → 404, etc.)
- [ ] Transacções na camada de aplicação (via `UnitOfWork` ou callback `WithTx`)
- [ ] Validação: sintáctica (adapter) vs de negócio (domínio)

## M7.6 — Estrutura de projecto
- [ ] `cmd/`, `internal/`, `pkg/` (e porque `pkg/` é opcional/debatido)
- [ ] **Por camada** vs **por feature** (preferir feature à medida que cresce)
- [ ] Nota: `golang-standards/project-layout` **não é oficial** — usa com critério
- [ ] Impor regras: linter `depguard` (golangci-lint) / `arch-go`

## Aulas
- [ ] M7.1 — `colar-link` · *pesquisa: "Clean Architecture Golang"*
- [ ] M7.2 — `colar-link` · *pesquisa: "SOLID principles in Go"*
- [ ] M7.3 — `colar-link` · *pesquisa: "Go dependency injection wire vs fx"*
- [ ] M7.4 — `colar-link` · *pesquisa: "DDD Golang tactical patterns"*
- [ ] M7.5 / M7.6 — `colar-link` · *pesquisa: "Go project structure"*

## Exercícios
- [ ] **EX7.1** — Refactor do URL Shortener para camadas (domain/app/adapter)
- [ ] **EX7.2** — Trocar repositório memória → Postgres **sem tocar** em domain/app
- [ ] **EX7.3** — Identificar e corrigir 1 violação por princípio SOLID em código teu antigo
- [ ] **EX7.4** — Mapeamento centralizado de erros domínio → HTTP
- [ ] **EX7.5** — `WithTx` / Unit of Work para transferência entre contas
- [ ] **EX7.6** — Configurar `depguard` para **proibir** `domain` de importar `adapter`
- [ ] **EX7.7** — Escrever um ADR ([[Template - ADR]]): "estrutura por feature vs por camada"
- [ ] ** Mini-projecto:** **Wallet Service** (criar conta, depositar, transferir, extracto) com Clean Architecture, testes unitários dos use cases com fakes e integração do adapter Postgres

## Erros comuns
- **Over-engineering**: uma interface para cada struct, 7 pastas para um CRUD
- Entidades anémicas (só getters/setters) com regras no service
- Interfaces definidas no produtor (estilo Java) → acoplamento
- Pacotes `models`, `utils`, `helpers` — nomes sem significado de domínio
- Vazar tipos de BD/HTTP para dentro do domínio

## Dicas
> [!tip] Começa simples, extrai quando doer
> Em Go, uma função + um struct chegam muitas vezes. Introduz o port quando houver segunda implementação **ou** necessidade real de teste/isolamento.

> [!tip] Um teste de arquitectura vale ouro
> Se o linter falha ao importar `adapter` no `domain`, a regra deixa de depender de disciplina.

## Critério de conclusão
- [ ] Wallet Service: troco repositório sem alterar use cases
- [ ] Use cases cobertos por testes sem BD, sem HTTP
- [ ] Consigo desenhar o diagrama de dependências de memória
