---
tags: [golang, roadmap, fase-1]
fase: 1
status: por-começar
semanas: 3
---

# Fase 01 — Fundamentos

← [[Fase 00 - Setup e Mentalidade]] | [[00 - HOME|Home]] | [[Fase 02 - Intermédio]] →

> [!abstract] Objectivo
> Dominar a sintaxe e o modelo de memória básico. Referência principal: **A Tour of Go** (go.dev/tour) + **Go by Example** (gobyexample.com).

## M1.1 — Tipos, variáveis e controlo de fluxo
- [x] `var`, `:=`, constantes, `iota`
- [x] Tipos: `int*`, `uint*`, `float64`, `bool`, `string`, `byte`, `rune`
- [x] **Zero values** (porque Go não tem `undefined`)
- [x] `for` (único loop), `range`, `break/continue`, labels
- [x] `if` com statement de inicialização, `switch` (sem `break`), type switch
- [x] `fmt`: `Printf`, `Sprintf`, `Errorf`, verbos `%v %+v %#v %T %q`

## M1.2 — Funções
- [ ] Múltiplos retornos e *named returns*
- [ ] Variádicas, funções como valores, closures
- [ ] `defer` (ordem LIFO, avaliação dos argumentos)
- [ ] Padrão `(valor, error)` desde o dia 1

## M1.3 — Colecções
- [ ] Arrays vs **slices** (`len`, `cap`, `append`, `copy`, slicing `a[i:j:k]`)
- [ ] Como o slice partilha o *backing array*
- [ ] `map` (criação, `v, ok := m[k]`, `delete`, iteração sem ordem)
- [ ] Pacotes `slices` e `maps` da stdlib

## M1.4 — Structs, métodos e ponteiros
- [ ] Structs, literais, structs anónimas, *struct tags*
- [ ] Métodos: receiver por **valor vs ponteiro** (quando usar cada)
- [ ] Ponteiros (`&`, `*`), `new`, *escape* para heap (conceito)
- [ ] Construtores: `NewX(...) *X`
- [ ] Comparação e cópia de structs

## M1.5 — Strings e texto
- [ ] Strings são bytes UTF-8 imutáveis; `rune` vs `byte`
- [ ] `strings`, `strconv`, `unicode`, `strings.Builder`
- [ ] `bufio.Scanner` para ler input

## M1.6 — Packages
- [ ] Visibilidade por maiúscula/minúscula
- [ ] Imports, aliases, `init()`, ciclo de imports (proibido)
- [ ] Organizar código em vários ficheiros/pacotes

## Aulas
- [ ] M1.1 — `colar-link`
- [ ] M1.2 — `colar-link`
- [ ] M1.3 — `colar-link` · *pesquisa: "Go slices internals"*
- [ ] M1.4 — `colar-link` · *pesquisa: "Go pointer vs value receiver"*
- [ ] M1.5 / M1.6 — `colar-link`

## Exercícios
- [ ] **EX1.1** — FizzBuzz + tabuada com `for`
- [ ] **EX1.2** — Contador de frequência de palavras (`map`)
- [ ] **EX1.3** — Inverter string com suporte a Unicode (`rune`)
- [ ] **EX1.4** — Stack e Queue com slices
- [ ] **EX1.5** — `Conta` (struct) com `Depositar/Levantar` e erro de saldo insuficiente
- [ ] **EX1.6** — Reproduzir o bug do *slice aliasing* e corrigi-lo
- [ ] **EX1.7** — 20 exercícios do Exercism (trilha Go)
- [ ] ** Mini-projecto:** CLI **Todo** com persistência em ficheiro (sem JSON ainda: texto simples)

## Erros comuns
- Escrever em `map` nil → **panic**
- `x := ...` dentro de `if/for` a **sombrear** variável externa
- Esquecer que `append` pode devolver outro slice
- Variável de loop em closures *(corrigido no Go 1.22: cada iteração tem a sua variável — mas vais ver código antigo)*

## Dicas
> [!tip] Não ignores `go vet`
> Corre `go vet ./...` e `go test ./...` sempre antes de commit.

> [!tip] Ponteiro vs valor
> Receiver por ponteiro quando muta estado ou a struct é grande; **não misturar** os dois no mesmo tipo.

## Critério de conclusão
- [ ] Escrevo o Todo CLI sem copiar
- [ ] Explico a diferença entre `len` e `cap`
- [ ] Explico quando usar receiver ponteiro
