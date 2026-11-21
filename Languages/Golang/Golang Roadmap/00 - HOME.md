---
tags: [golang, roadmap, home]
cssclasses: [wide-page]
---

# Golang Roadmap — Backend, Microservices & Modular Monolith

> [!abstract] Objectivo
> Ir de **zero em Go** até construir **backends em produção**: APIs REST/gRPC, **modular monolith** e **microservices**, com **SOLID + Clean Architecture** aplicados de forma *idiomática* (sem copiar Java para Go).

## Como navegar
- Progresso geral → [[02 - Dashboard]]
- Como usar este vault → [[01 - Como usar]]
- Colar links de aulas → [[YouTube - Inbox de Aulas]]
- Notas diárias/semanais → `50 - Notas/`

## Mapa de Fases

### Trilho A — Core (Go sólido)
- [ ] [[Fase 00 - Setup e Mentalidade]] — ~1 semana
- [ ] [[Fase 01 - Fundamentos]] — ~3 semanas
- [ ] [[Fase 02 - Intermédio]] — ~3 semanas
- [ ] [[Fase 03 - Concorrência]] — ~3 semanas
- [ ] [[Fase 04 - HTTP e Stdlib]] — ~3 semanas
- [ ] [[Fase 05 - Bases de Dados]] — ~3 semanas
- [ ] [[Fase 06 - Testes]] — ~2 semanas

### Trilho B — Arquitectura
- [ ] [[Fase 07 - Clean Architecture e SOLID]] — ~4 semanas
- [ ] [[Fase 08 - Frameworks e Ecossistema]] — ~3 semanas
- [ ] [[Fase 09 - Modular Monolith]] — ~3 semanas

### Trilho C — Sistemas distribuídos
- [ ] [[Fase 10 - Microservices]] — ~4 semanas
- [ ] [[Fase 11 - Mensageria e Event-Driven]] — ~2 semanas
- [ ] [[Fase 12 - Observabilidade, Segurança e DevOps]] — ~3 semanas

### Trilho D — Maestria
- [ ] [[Fase 13 - Performance e Avançado]] — ~3 semanas
- [ ] [[Fase 14 - Capstone]] — ~6 semanas

> [!info] Duração total
> ~40 semanas a ~8–10 h/semana (≈ 9–10 meses). Ajusta ao teu ritmo; **a ordem importa mais que a velocidade**.

## Biblioteca
- [[Livros e Documentação]]
- [[Repositórios de Referência]]
- [[Ferramentas e Libs]]

## Cheatsheets
- [[Go - Sintaxe Essencial]]
- [[Go - Idiomas e Convenções]]
- [[SOLID em Go]]
- [[Clean Architecture em Go]]
- [[Flashcards Go]]

## Princípios deste roadmap
1. **Go idiomático primeiro, arquitectura depois.** Só aplica Clean Architecture quando já escreves Go sem pensar na sintaxe.
2. **Cada fase termina com código a correr + testes.** Sem "só vi o vídeo".
3. **Escreve a biblioteca padrão antes de escolher framework.** Frameworks são conveniência, não fundação.
4. **Pequenas interfaces, definidas no consumidor.** É o SOLID à moda de Go.
5. **Mede antes de optimizar.** `pprof` e benchmarks, não palpites.
