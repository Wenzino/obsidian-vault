---
tags: [golang, cheatsheet, solid, arquitectura]
---

# SOLID em Go

[[00 - HOME|Home]] · [[Fase 07 - Clean Architecture e SOLID]] · [[Clean Architecture em Go]]

> [!info] Princípio-guia
> Go não tem classes nem herança. SOLID aplica-se com **pacotes, structs pequenos, interfaces implícitas e composição**.

## S — Single Responsibility
Uma razão para mudar por pacote/tipo.
```go
//  faz tudo
type UserService struct{ db *sql.DB }
func (s *UserService) Register(w http.ResponseWriter, r *http.Request) { /* parse + valida + SQL + email */ }

//  separado
type Registrar struct{ users UserSaver; mail Mailer }  // regra de negócio
func (r *Registrar) Register(ctx context.Context, in Input) error { /* ... */ return nil }
// handler HTTP → só traduz HTTP <-> Registrar
// repo Postgres → só persistência
```

## O — Open/Closed
Estende sem modificar: **decorators** e middleware.
```go
type Notifier interface{ Notify(ctx context.Context, msg string) error }

type LoggingNotifier struct {
    next Notifier
    log  *slog.Logger
}
func (l LoggingNotifier) Notify(ctx context.Context, msg string) error {
    l.log.InfoContext(ctx, "notify", "msg", msg)
    return l.next.Notify(ctx, msg)
}
// Novo comportamento (retry, métricas) = novo wrapper, sem tocar no original.
```

## L — Liskov Substitution
Qualquer implementação respeita o **contrato** (semântica, erros, nil).
```go
// Contrato: FindByID devolve ErrNotFound quando não existe — nunca (nil, nil).
type UserRepo interface {
    FindByID(ctx context.Context, id string) (*User, error)
}
//  implementação que devolve (nil, nil) quebra quem usa o resultado
//  memory e postgres devolvem ErrNotFound — validar com UMA suite de testes partilhada
```
> [!tip] Contract test
> Escreve `func RunUserRepoContract(t *testing.T, repo UserRepo)` e corre-o contra memory e Postgres.

## I — Interface Segregation
Interfaces mínimas, **no consumidor**.
```go
//  interface gigante no produtor
type UserStore interface { Save; Find; List; Delete; Count; Search /* ... */ }

//  cada use case declara só o que usa
package register
type userSaver interface{ Save(ctx context.Context, u user.User) error }
```

## D — Dependency Inversion
Alto nível depende de abstracções; o **composition root** liga as implementações.
```go
// app (alto nível): conhece o port, não o Postgres
type Transfer struct{ wallets WalletRepo; clock Clock }
func NewTransfer(w WalletRepo, c Clock) *Transfer { return &Transfer{w, c} }

// cmd/api/main.go (composition root)
repo := postgres.NewWalletRepo(pool)
uc   := app.NewTransfer(repo, clock.System{})
```

## Quando NÃO aplicar
- [ ] Função simples sem 2.ª implementação nem necessidade de teste → **sem interface**
- [ ] Script/CLI pequeno → sem camadas
- [ ] Introduz o port **quando doer** (testes difíceis, 2.ª implementação, fronteira de módulo)
