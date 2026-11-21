---
tags: [golang, roadmap, fase-0]
fase: 0
status: por-começar
semanas: 1
---

# Fase 00 — Setup e Mentalidade

[[00 - HOME|Home]] | [[Fase 01 - Fundamentos]] →

> [!abstract] Objectivo
> Ambiente profissional pronto e a filosofia de Go compreendida **antes** de escrever código a sério.

## M0.1 — Ambiente
- [x] Instalar Go a partir de go.dev/dl e validar com `go version` e `go env`
- [ ] Perceber `GOROOT`, `GOPATH`, `GOBIN`, `GOMODCACHE`
- [ ] Editor: VS Code + extensão oficial Go (ou GoLand); `gopls` a funcionar
- [ ] Instalar: `goimports`, `golangci-lint`, `dlv` (debugger), `air` (live reload)
- [ ] Docker + Docker Compose instalados (vais precisar nas fases 5+)
- [ ] Repo Git `go-lab` para todos os exercícios

## M0.2 — Toolchain
- [ ] `go mod init`, `go mod tidy`, `go get`, `go work` (workspaces)
- [ ] `go run`, `go build`, `go install`, `go test`, `go vet`, `go fmt`, `go doc`
- [ ] Cross-compilation: `GOOS=linux GOARCH=amd64 go build`
- [ ] Estrutura mínima: `main.go` + `go.mod`

## M0.3 — Mentalidade Go
- [ ] Ler *Effective Go* (go.dev/doc/effective_go)
- [ ] Ler os *Go Proverbs* (go-proverbs.github.io)
- [ ] Perceber: simplicidade > esperteza, composição > herança, erros são valores
- [ ] Perceber porque Go **não tem** classes, herança, excepções, overloading
- [ ] Ler o guia de estilo da Uber (github.com/uber-go/guide)

## Aulas (cola os links)
- [ ] M0.1 — `colar-link`
- [ ] M0.2 — `colar-link`
- [ ] M0.3 — `colar-link` · *pesquisa: "Go proverbs Rob Pike"*

## Exercícios
- [ ] **EX0.1** — Hello World; compilar para Linux e Windows a partir do teu SO
- [ ] **EX0.2** — CLI que recebe `--nome` e imprime saudação (`flag`)
- [ ] **EX0.3** — Configurar `golangci-lint` com `.golangci.yml` e corrigir os avisos
- [ ] **EX0.4** — Fazer debug de um programa simples com `dlv` ou VS Code

## Dicas
> [!tip] Formatação não se discute
> `gofmt`/`goimports` ao guardar. O estilo é o da comunidade inteira — tira o tema "tabs vs espaços" da mesa.

> [!tip] Lê código da stdlib
> `go doc strings.Builder` e abre o código-fonte. A stdlib é o melhor manual de Go idiomático.

## Critério de conclusão
- [ ] Crio um módulo, compilo e corro sem consultar
- [ ] Sei explicar 3 diferenças entre Go e a linguagem que uso hoje
