---
tags: [golang, recursos, ferramentas]
---

# Ferramentas e Libs — escolhas por defeito

[[00 - HOME|Home]]

> [!info] Como ler
> **Defeito** = onde começar. **Alternativas** = quando o contexto justifica. Regista desvios com [[Template - ADR]].

## Aplicação
| Área | Defeito | Alternativas |
|---|---|---|
| Router HTTP | `net/http` (1.22+) / `chi` | Gin, Echo, Fiber |
| Validação | `go-playground/validator` | validação manual no domínio |
| Config | env vars + `caarlos0/env` ou `koanf` | `viper` |
| Logging | `log/slog` | zap, zerolog |
| CLI | `cobra` | `urfave/cli`, `flag` |
| DI | manual (construtores) | `wire`, `fx`, `dig` |
| Auth | `golang-jwt/jwt` + argon2id | OIDC (`go-oidc`), casbin |
| OpenAPI | `oapi-codegen` | `swaggo/swag`, Huma |
| Jobs | `asynq` / `river` | `gocron`, Temporal |

## Dados
| Área | Defeito | Alternativas |
|---|---|---|
| BD | PostgreSQL | MySQL, SQLite (testes/CLI) |
| Driver | `pgx` | `database/sql` + `lib/pq` |
| Acesso | `sqlc` | sqlx, GORM, Ent, Bun |
| Migrações | `goose` / `golang-migrate` | `atlas` |
| Cache | Redis (`go-redis`) | in-memory (`ristretto`) |

## Distribuído
| Área | Defeito | Alternativas |
|---|---|---|
| RPC | gRPC + `buf` | Connect, REST |
| Broker | NATS / RabbitMQ | Kafka (volume/replay) |
| Abstracção de eventos | Watermill | cliente nativo |
| Resiliência | `gobreaker`, `x/sync/errgroup`, `x/time/rate` | |
| Workflows | — | Temporal |

## Qualidade e operação
| Área | Defeito | Alternativas |
|---|---|---|
| Lint | `golangci-lint` (+ `depguard`) | `staticcheck` |
| Testes | `testing` + `testify` | `go-cmp`, `ginkgo` |
| Mocks | fakes à mão | `mockery`, `go.uber.org/mock` |
| Integração | `testcontainers-go` | docker compose |
| Hot reload | `air` | `reflex` |
| Observabilidade | OpenTelemetry + Prometheus + Grafana | Datadog, etc. |
| Segurança | `govulncheck` | Trivy, Snyk |
| Release | `goreleaser` | scripts |
| CI | GitHub Actions | GitLab CI |

## Comandos úteis
```bash
go mod tidy                 # limpa dependências
go vet ./...                # análise estática básica
go test ./... -race -cover  # testes + race + cobertura
go test -bench=. -benchmem  # benchmarks
go build -gcflags=-m        # escape analysis
govulncheck ./...           # vulnerabilidades
golangci-lint run           # lint agregado
```
