---
name: rust-microservices
description: Implementa microsserviços em Rust com Axum/tonic, mensageria, outbox, health, clients resilientes, tracing e deploy Linux. Use ao criar ou revisar serviços Rust distribuídos, workers, gRPC ou splits a partir de APIs Axum.
---

# Rust microsservices

Serviços Rust: um binário por bounded context, async Tokio, falhas tipadas. Arquitetura: `microservices-architecture`. HTTP: `axum`. Produção HTTP: `rust-axum-production`. Linguagem: `rust` / `rust-best-practices`.

## Stack típica

| Peça | Papel |
|------|--------|
| **Axum** | HTTP externo/BFF |
| **tonic** | gRPC interno (se o projeto usar) |
| **Mensageria** | `rdkafka`, NATS, SQS — crates do repo |
| **DB** | sqlx / sea-orm — um DB por serviço |
| **Obs** | tracing + otel + metrics |

Não introduza runtime além de Tokio sem necessidade.

## Layout

```text
services/orders/
  Cargo.toml
  src/
    main.rs
    lib.rs          # app() testável
    config.rs
    http/ | grpc/
    app/
    domain/
    adapters/       # db, bus, clients
  migrations/
  Dockerfile
```

Workspace Cargo com `crates/contracts` para protos/DTOs versionados quando multi-serviço.

## Regras inegociáveis

- Handlers/`async fn` com `Result<T, AppError>`; sem `unwrap` em borda.
- Timeouts em clients (`tower::timeout`, reqwest, tonic deadlines).
- Idempotência em consumers e POSTs críticos.
- `/health` e `/ready`; graceful shutdown (`rust-axum-production`).
- Config tipada no boot (`figment`/env); fail-fast.
- Sem shared write DB entre serviços.
- Tipos `Send` nas tasks spawnadas; sem bloquear o runtime.

## Sync

- Axum routers por serviço; DTOs serde.
- gRPC: proto versionado; status codes corretos; deadlines.
- Tower layers: trace, concurrency limit, load shed quando necessário.
- Retries só em idempotentes; backoff + jitter.

## Async / mensagens

```text
DB write + outbox (mesma tx) → dispatcher → broker → consumer (inbox)
```

- Outbox/inbox em sqlx transaction.
- Consumer: ack após persistência; DLQ para poison.
- Payload versionado (`version` / topic por major).
- Limite de concorrência no consume (`Semaphore` / config do crate).

## Resiliência

- `CancellationToken` / broadcast shutdown para tasks de background.
- Circuit breaker (tower ou lib do projeto) em deps críticas.
- `JoinHandle` tracked; abort no shutdown.
- Pool DB com `max_connections` consciente de réplicas.

## Observabilidade

- `tracing` com `service.name`, request/message id.
- Propagação W3C tracecontext em HTTP e headers de mensagem.
- Métricas: HTTP, consumer lag, erros por causa.

## Testes

- `rust-unit-testing`: domínio + `Router::oneshot`.
- Integração com testcontainers/broker local só nos riscos.
- Fakes de ports cross-service.

## Anti-padrões

- Um workspace que vira monólito linkado sem limites de deploy
- `unwrap` em consumer loop
- Esquecer cancelamento de tasks no SIGTERM
- Clonar `AppState` pesado em vez de handles/`Arc`
- Eventos acoplados a structs internas sem DTO estável

## Critérios de conclusão

- Binário com config, health, timeout e shutdown
- Comunicação sync/async alinhada a `microservices-architecture`
- Consumers idempotentes; outbox quando write+evento
- Tracing/métricas no caminho crítico
- Testes dos riscos do serviço passando
