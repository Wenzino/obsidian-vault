---
tags: [golang, flashcards]
---

# Flashcards Go

[[00 - HOME|Home]]

> [!info] Usa o plugin **Spaced Repetition** (formato `pergunta::resposta`). Adiciona os teus cartões à medida que estudas — #flashcards

## Fundamentos #flashcards/go
Qual é o *zero value* de `string`, `int`, `bool`, ponteiro e slice?::`""`, `0`, `false`, `nil`, `nil`
Qual a diferença entre `len` e `cap` num slice?::`len` = elementos visíveis; `cap` = capacidade do backing array a partir do início do slice
O que acontece ao escrever num `map` nil?::Panic. Ler devolve zero value; escrever exige `make`
Quando usar receiver por ponteiro?::Quando o método muta estado, a struct é grande, ou para consistência no tipo
`rune` vs `byte`?::`byte` = `uint8`; `rune` = `int32` (code point Unicode)
Em que ordem executam os `defer`?::LIFO (último a entrar, primeiro a sair); argumentos avaliados no momento do `defer`

## Interfaces e erros #flashcards/go
Porque é que interface com valor nil ≠ interface nil?::Uma interface guarda (tipo, valor); com tipo concreto e valor nil, a interface não é nil
`errors.Is` vs `errors.As`?::`Is` compara com um erro/sentinela na cadeia; `As` extrai um tipo concreto da cadeia
O que significa "accept interfaces, return structs"?::Parâmetros abstractos para flexibilidade; retornos concretos para clareza e extensibilidade
Onde se deve definir uma interface em Go?::No pacote **consumidor**

## Concorrência #flashcards/go
Quem deve fechar um channel?::O produtor (nunca o consumidor); fechar duas vezes dá panic
O que faz `select` com `default`?::Torna a operação não bloqueante
Para que serve `errgroup`?::Lançar goroutines, esperar por todas e propagar o 1.º erro (com cancelamento via context)
Onde se passa `context.Context`?::Primeiro parâmetro; nunca guardado em struct
Como detectar data races?::`go test -race` / `go run -race`

## Arquitectura #flashcards/go
O que diz a Dependency Rule?::Dependências de código apontam sempre para dentro (adapters → app → domain)
O que é o composition root?::O sítio (`main`) onde se instanciam e ligam as dependências concretas
Porque proibir JOINs entre módulos num modular monolith?::Evita acoplamento de dados e permite extrair o módulo para serviço depois
O que resolve o Outbox pattern?::Dual write: grava dado + evento na mesma transacção e um relay publica depois
Porque consumidores devem ser idempotentes?::Entrega *at-least-once* implica duplicados
Saga: coreografia vs orquestração?::Coreografia = serviços reagem a eventos; orquestração = coordenador central dita os passos
