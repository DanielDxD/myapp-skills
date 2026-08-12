---
name: golang-microservices
description: Implementa microsserviços em Go com HTTP/gRPC, mensageria, outbox, health, config, clients resilientes e organização por serviço. Use ao criar ou revisar serviços Go distribuídos, workers, consumers Kafka/NATS/Rabbit ou splits a partir de APIs Gin.
---

# Golang microsservices

Um binário Go por serviço (ou worker), limites claros e clients com timeout. Arquitetura geral: `microservices-architecture`. HTTP fino: `golang-apis`. Concorrência: `golang-goroutines`.

## Stack típica

| Peça | Papel |
|------|--------|
| **HTTP** | Gin/chi/stdlib — o do projeto |
| **gRPC** | Contratos internos de baixa latência |
| **Mensageria** | Kafka / NATS / Rabbit / SQS — o adotado |
| **DB** | sqlc/pgx/GORM — um DB por serviço |
| **Obs** | otel + slog/zap + Prometheus |

Preserve libs do repo. Não troque broker/ORM sem pedido.

## Layout por serviço

```text
services/orders/
  cmd/orders/main.go
  internal/
    config/
    http/ ou grpc/
    app/           # use cases
    domain/
    adapter/       # db, queue, clients
  migrations/
  Dockerfile
```

- `main` = wiring + graceful shutdown.
- Domínio sem imports de framework HTTP.
- Clients de outros serviços atrás de interfaces.

## Regras inegociáveis

- `context.Context` em toda borda (handler → use case → DB/RPC).
- Timeout em **todo** client HTTP/gRPC/DB.
- Idempotency keys / dedupe em consumers e POSTs críticos.
- Health: `/live` (processo) e `/ready` (deps críticas).
- Config tipada no boot; fail-fast se inválida.
- Sem shared database com outro serviço.

## Sync (HTTP/gRPC)

- Contratos versionados; DTOs explícitos.
- Middleware: request id, auth, logging, recovery.
- Erros tipados → status/codes estáveis.
- Retries só em idempotentes; backoff + jitter.
- gRPC: deadlines, status codes corretos, sem panic em interceptor.

## Async (mensagens)

```text
Write local DB → Outbox → Publisher → Topic → Consumer (inbox/idempotente)
```

- Outbox na mesma transação do write de negócio.
- Consumer: commit offset/ack só após side effect durável (ou inbox).
- Schema de evento versionado; consumidores tolerantes a campos novos.
- Poison messages: DLQ + alerta, não loop infinito.
- Handlers de consumer curtos; trabalho pesado com limite de concorrência (`errgroup`, semaphores).

## Resiliência

- Circuit breaker (sony/gobreaker ou mesh) em deps críticas.
- Bulkheads: pools/limites por dependência.
- Graceful shutdown: pare de aceitar → drenar → fechar pool/producer.
- `ErrGroup` com cancelamento; não vaze goroutines (`golang-goroutines`).

## Observabilidade

- Trace id / baggage propagados em HTTP e mensagens.
- Métricas: latência, erro, saturação de fila, lag de consumer.
- Logs estruturados sem PII; `service=orders` em todo log.

## Testes

- Unitários de domínio e handlers (`golang-unit-testing`).
- Contract/integration na borda com broker/DB de teste quando o risco exigir.
- Fakes de clients cross-service nos unitários.

## Anti-padrões

- `init()` com I/O global
- Consumer sem idempotência
- `http.DefaultClient` sem timeout
- Um monorepo “micro” que deploya tudo acoplado sem necessidade
- Copiar structs de outro serviço como “API interna” sem versionar

## Critérios de conclusão

- Serviço bootável com config, health e shutdown limpo
- Clients com timeout/idempotência
- Eventos ou RPC alinhados a `microservices-architecture`
- Sem shared write DB
- Métricas/logs/traces mínimos no caminho crítico
