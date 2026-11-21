---
tags: [golang, cheatsheet, estilo]
---

# Go — Idiomas e Convenções

[[00 - HOME|Home]] · [[Go - Sintaxe Essencial]]

## Nomes
- [ ] Pacotes: curtos, minúsculos, sem `_`, sem plural; **sem** `util`, `common`, `base`, `models`
- [ ] Evitar repetição: `user.Service`, não `user.UserService`
- [ ] Sem prefixo `Get` em getters: `u.Name()`, não `u.GetName()`
- [ ] Interfaces de um método: sufixo `-er` (`Reader`, `Notifier`)
- [ ] Receivers: 1–2 letras consistentes (`func (a *Account)`)
- [ ] Siglas em maiúsculas: `ID`, `URL`, `HTTP`

## Design
- [ ] **Accept interfaces, return structs**
- [ ] Interfaces **pequenas** e **definidas no consumidor**
- [ ] Zero value útil (`sync.Mutex`, `bytes.Buffer`)
- [ ] `context.Context` como **1.º parâmetro**; nunca em struct
- [ ] Construtores `NewX`; opções com *functional options* quando há muitas
- [ ] Preferir composição (embedding) a hierarquias
- [ ] "Um pouco de cópia é melhor que uma pequena dependência"

## Erros
- [ ] Tratar **sempre** o erro (ou ignorar explicitamente com comentário)
- [ ] Mensagens em minúsculas, sem ponto final
- [ ] Adicionar contexto com `%w`
- [ ] Erro tratado **uma vez**: ou loga ou devolve — não ambos
- [ ] `panic` só para erros de programação/arranque

## Concorrência
- [ ] Quem lança a goroutine sabe como ela termina
- [ ] O produtor fecha o channel
- [ ] Passa `ctx` para cancelamento
- [ ] Corre `-race` no CI

## Estrutura
- [ ] `cmd/` para binários; `internal/` para código privado
- [ ] Ficheiros pequenos e coesos; `_test.go` ao lado do código
- [ ] Sem ciclos de import (sinal de fronteiras mal desenhadas)

## Checklist antes de commit
```bash
gofmt -l . && go vet ./... && golangci-lint run && go test ./... -race
```
