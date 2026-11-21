---
tags: [golang, roadmap, fase-11, mensageria]
fase: 11
status: por-começar
semanas: 2
---

# Fase 11 — Mensageria e Event-Driven

← [[Fase 10 - Microservices]] | [[00 - HOME|Home]] | [[Fase 12 - Observabilidade, Segurança e DevOps]] →

> [!abstract] Objectivo
> Comunicação assíncrona fiável entre serviços: brokers, garantias de entrega e desenho de eventos.

## M11.1 — Conceitos
- [ ] Mensagem vs evento vs comando
- [ ] Pub/Sub, filas, streams, consumer groups
- [ ] Garantias: *at-most-once*, **at-least-once**, *exactly-once* (e porque é um mito prático)
- [ ] Ordenação, partições, *backpressure*

## M11.2 — Brokers e clientes Go
- [ ] **RabbitMQ** (`rabbitmq/amqp091-go`): exchanges, queues, bindings, acks
- [ ] **Kafka** (`twmb/franz-go` ou `segmentio/kafka-go`): topics, partitions, offsets
- [ ] **NATS / JetStream** (`nats.go`): simples e leve
- [ ] Redis Streams (noção)
- [ ] **Watermill** (ThreeDotsLabs): abstracção Go sobre vários brokers

## M11.3 — Fiabilidade
- [ ] **Outbox** (produtor) + **Inbox / dedupe** (consumidor)
- [ ] Consumidores **idempotentes**
- [ ] Retries com backoff e **Dead Letter Queue**
- [ ] Poison messages
- [ ] Shutdown gracioso do consumidor (terminar mensagem em curso)

## M11.4 — Desenho de eventos
- [ ] Nomes no passado (`OrderPlaced`), payload estável, `event_id`, `occurred_at`, `version`
- [ ] Schemas: Protobuf / Avro / JSON Schema
- [ ] Evolução compatível (backward/forward)
- [ ] Event notification vs event-carried state transfer

## Aulas
- [ ] M11.1 — `colar-link` · *pesquisa: "event driven architecture explained"*
- [ ] M11.2 RabbitMQ — `colar-link`
- [ ] M11.2 Kafka — `colar-link`
- [ ] M11.2 NATS — `colar-link`
- [ ] M11.3 — `colar-link` · *pesquisa: "idempotent consumer outbox inbox"*

## Exercícios
- [ ] **EX11.1** — Producer/consumer básico com NATS **ou** RabbitMQ
- [ ] **EX11.2** — Mesmo fluxo com Kafka (consumer group, 3 partições)
- [ ] **EX11.3** — Retry com backoff + DLQ para mensagens "venenosas"
- [ ] **EX11.4** — Outbox em Postgres + relay → broker (provar sem perda com crash simulado)
- [ ] **EX11.5** — Consumidor idempotente (tabela `processed_events`)
- [ ] **EX11.6** — Reescrever a comunicação com **Watermill**
- [ ] ** Mini-projecto:** Ligar `orders` → `payments` → `notifications` por eventos, com DLQ e dashboard de lag

## Erros comuns
- Assumir *exactly-once*
- Consumidor que não é idempotente
- Publicar no broker **fora** da transacção da BD (dual write)
- Eventos com payload gigante / acoplados à tabela

## Dicas
> [!tip] Começa com NATS ou RabbitMQ
> Menos operação. Kafka compensa quando precisas de **log replayable** e alto volume.

## Critério de conclusão
- [ ] Derrubar o consumidor a meio **não** perde nem duplica efeitos
- [ ] DLQ com mensagens inspeccionáveis
