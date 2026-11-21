---
tags: [golang, recursos, repos]
---

# Repositórios de Referência

[[00 - HOME|Home]]

> [!warning] Verifica antes de usar
> Repos mudam e envelhecem. Confirma data do último commit, versão do Go e licença.

## Arquitectura em Go
- [ ] `bxcodec/go-clean-arch` — Clean Architecture clássica em Go
- [ ] `evrone/go-clean-template` — template de Clean Architecture
- [ ] `ThreeDotsLabs/wild-workouts-go-ddd-example` — DDD, CQRS, módulos, Go
- [ ] `ardanlabs/service` — serviço de produção (Ardan Labs)
- [ ] `golang-standards/project-layout` — layout comunitário (**não oficial**)

## Libs chave (ler o código/README)
- [ ] `go-chi/chi` · `gin-gonic/gin` · `labstack/echo` · `gofiber/fiber`
- [ ] `jackc/pgx` · `sqlc-dev/sqlc` · `go-gorm/gorm` · `ent/ent`
- [ ] `ThreeDotsLabs/watermill` — mensageria
- [ ] `uber-go/fx` · `google/wire` — DI
- [ ] `spf13/cobra` · `spf13/viper`
- [ ] `stretchr/testify` · `testcontainers/testcontainers-go`
- [ ] `hibiken/asynq` · `riverqueue/river`
- [ ] `golangci/golangci-lint`

## Como estudar um repo
1. Lê o README e o `go.mod` (dependências dizem muito)
2. Começa em `cmd/*/main.go` (composition root)
3. Segue **um** caso de uso do HTTP até à BD
4. Lê os testes
5. Regista aprendizagens numa nota (usa [[Template - Nota de Aula]])

| Repo | O que aprendi | Ideias a roubar | Data |
|---|---|---|---|
| | | | |
