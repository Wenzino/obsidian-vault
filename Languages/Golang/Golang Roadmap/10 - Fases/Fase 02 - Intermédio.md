---
tags: [golang, roadmap, fase-2]
fase: 2
status: por-começar
semanas: 3
---

# Fase 02 — Intermédio

← [[Fase 01 - Fundamentos]] | [[00 - HOME|Home]] | [[Fase 03 - Concorrência]] →

> [!abstract] Objectivo
> Interfaces, erros, generics, módulos e I/O. Aqui nasce o Go **idiomático** — e a base de todo o SOLID em Go.

## M2.1 — Interfaces
- [ ] Interfaces implícitas (*duck typing* estático)
- [ ] **Interfaces pequenas** (`io.Reader`, `io.Writer`, `fmt.Stringer`)
- [ ] *Accept interfaces, return structs*
- [ ] `interface{}`/`any`, type assertion, type switch
- [ ] Armadilha: interface com valor `nil` ≠ interface `nil`
- Ligação: [[SOLID em Go]] (ISP e DIP)

## M2.2 — Composição
- [ ] Embedding de structs e de interfaces
- [ ] Promoção de métodos e limites do embedding
- [ ] Composição em vez de herança
- [ ] Functional options pattern (`WithTimeout(...)`)

## M2.3 — Tratamento de erros
- [ ] `error` como valor; erros sentinela (`ErrNotFound`)
- [ ] Wrapping com `%w`, `errors.Is`, `errors.As`, `errors.Join`
- [ ] Tipos de erro personalizados
- [ ] `panic/recover` — quando **não** usar
- [ ] Estratégia: erros do domínio vs erros de infra (vais precisar na Fase 7)

## M2.4 — Generics
- [ ] Type parameters, constraints, `comparable`, `cmp.Ordered`
- [ ] Quando usar (estruturas de dados, utilitários) e quando **não** usar
- [ ] `slices`, `maps`, `cmp` da stdlib
- [ ] Iteradores (`iter.Seq`, range-over-func) — noção básica

## M2.5 — Módulos e pacotes
- [ ] `go.mod`, `go.sum`, versionamento semântico, `replace`
- [ ] `internal/` (encapsulamento imposto pelo compilador)
- [ ] Módulos privados (`GOPRIVATE`), `vendor`
- [ ] Nomes de pacotes curtos, sem `util`/`common`/`base`

## M2.6 — I/O e stdlib essencial
- [ ] `io`, `bufio`, `os`, `path/filepath`
- [ ] `encoding/json` (tags, `Marshal/Unmarshal`, `Decoder/Encoder`)
- [ ] `time` (formatos, durations, time zones)
- [ ] `log/slog` (logging estruturado)
- [ ] `flag` e variáveis de ambiente

## Aulas
- [ ] M2.1 — `colar-link` · *pesquisa: "Go interfaces explained"*
- [ ] M2.2 — `colar-link`
- [ ] M2.3 — `colar-link` · *pesquisa: "Go error handling errors.Is errors.As"*
- [ ] M2.4 — `colar-link` · *pesquisa: "Go generics tutorial"*
- [ ] M2.5 / M2.6 — `colar-link`

## Exercícios
- [ ] **EX2.1** — Interface `Shape` (Área/Perímetro) com 3 implementações
- [ ] **EX2.2** — Erros personalizados `ValidationError` + `errors.As`
- [ ] **EX2.3** — Genéricos: `Stack[T]`, `Set[T]`, `Map/Filter/Reduce`
- [ ] **EX2.4** — Loader de config (JSON + env) com *functional options*
- [ ] **EX2.5** — Pipeline com `io.Reader`: ler ficheiro grande linha a linha e contar
- [ ] **EX2.6** — Reproduzir o bug "interface nil não é nil"
- [ ] ** Refactor do Todo:** extrair `TodoRepository` (interface) com implementações `memory` e `file(JSON)`; o "serviço" só conhece a interface

## Erros comuns
- Criar interfaces **antes** de ter 2 implementações / consumidor real
- Devolver interfaces em vez de structs
- Comparar erros com `==` em vez de `errors.Is`
- `panic` para fluxo normal

## Dicas
> [!tip] Define a interface onde ela é **usada**
> Em Go, o consumidor define o contrato — o produtor não precisa de saber.

> [!tip] Erros em minúsculas, sem pontuação
> `fmt.Errorf("carregar config: %w", err)`

## Critério de conclusão
- [ ] Refactor do Todo concluído sem tocar na lógica ao trocar de repositório
- [ ] Explico `errors.Is` vs `errors.As`
- [ ] Escrevo um generic útil sem copiar
