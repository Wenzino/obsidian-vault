---
tags: [golang, roadmap, fase-12, devops, seguranca, observabilidade]
fase: 12
status: por-começar
semanas: 3
---

# Fase 12 — Observabilidade, Segurança e DevOps

← [[Fase 11 - Mensageria e Event-Driven]] | [[00 - HOME|Home]] | [[Fase 13 - Performance e Avançado]] →

> [!abstract] Objectivo
> Pôr serviços Go em produção com visibilidade, segurança e entrega contínua.

## M12.1 — Observabilidade
- [ ] **Logs estruturados** com `slog` (JSON, níveis, `request_id`/`trace_id`)
- [ ] **Métricas**: `prometheus/client_golang` (RED: rate, errors, duration)
- [ ] **Tracing**: **OpenTelemetry** (`otelhttp`, `otelgrpc`), propagação de contexto
- [ ] Stack: Prometheus + Grafana + Loki + Tempo/Jaeger (via compose)
- [ ] Health/readiness, SLIs/SLOs (noção)
- [ ] Alertas úteis (poucos e accionáveis)

## M12.2 — Segurança
- [ ] OWASP API Security Top 10 (lista de verificação)
- [ ] Hashing de passwords: **argon2id** (`x/crypto/argon2`) ou bcrypt
- [ ] JWT bem feito: expiração curta, rotação, `aud/iss`, revogação (noção)
- [ ] OAuth2/OIDC em produção; CORS e CSRF
- [ ] Rate limiting, limites de body, timeouts
- [ ] Gestão de segredos (nunca no repo; env/secret manager)
- [ ] TLS e `crypto/tls` (noção)
- [ ] **`govulncheck`** e actualização de dependências (Dependabot/Renovate)

## M12.3 — CI/CD
- [ ] GitHub Actions: `fmt`, `vet`, `golangci-lint`, `test -race`, `govulncheck`, build
- [ ] Build e push de imagem Docker (multi-stage, tags por versão/commit)
- [ ] `goreleaser` para binários e releases
- [ ] Migrações no pipeline; estratégia de deploy (rolling/blue-green)
- [ ] Feature flags (noção)

## M12.4 — Plataforma
- [ ] 12-factor app aplicado
- [ ] Deploy: VPS (systemd + Caddy/Nginx), Fly.io/Railway/Render **ou** Kubernetes
- [ ] IaC com Terraform (noção)
- [ ] Backups e restauro testado da BD
- [ ] Runbooks e post-mortems (ver [[Template - ADR]] como base)

## Aulas
- [ ] M12.1 — `colar-link` · *pesquisa: "OpenTelemetry Go tutorial", "Prometheus Grafana Go"*
- [ ] M12.2 — `colar-link` · *pesquisa: "Go API security best practices"*
- [ ] M12.3 — `colar-link` · *pesquisa: "GitHub Actions Go CI"*
- [ ] M12.4 — `colar-link`

## Exercícios
- [ ] **EX12.1** — Adicionar logs JSON com `trace_id` a um serviço
- [ ] **EX12.2** — Endpoint `/metrics` + dashboard Grafana (latência p95, erros)
- [ ] **EX12.3** — Trace distribuído atravessando 2 serviços (Jaeger/Tempo)
- [ ] **EX12.4** — Auditar a tua API contra OWASP API Top 10 e corrigir 3 achados
- [ ] **EX12.5** — Pipeline completo: lint → test -race → vuln → build → push imagem
- [ ] **EX12.6** — Deploy num servidor real com HTTPS e restauro de backup testado
- [ ] ** Mini-projecto:** Stack de observabilidade completa para o mini e-commerce

## Erros comuns
- Logar dados sensíveis (tokens, passwords, PII)
- Métricas com cardinalidade explosiva (labels com IDs)
- Segredos em variáveis commitadas
- Backups nunca testados

## Dicas
> [!tip] Um `trace_id` nos logs poupa horas
> Propaga-o via `context` e inclui-o em **todos** os logs e respostas de erro.

## Critério de conclusão
- [ ] Consigo seguir um request ponta-a-ponta por trace + logs
- [ ] Pipeline bloqueia merge com lint/teste/vuln a falhar
- [ ] Deploy reproduzível documentado
