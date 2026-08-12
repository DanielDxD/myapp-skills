---
name: nestjs-microservices
description: Implementa microsserviços com NestJS — transporters (TCP, Redis, NATS, Kafka, RabbitMQ, gRPC), híbrido HTTP+micro, patterns, pipes, guards e outbox. Use ao criar ou revisar apps Nest hybrid, message patterns, ClientsModule ou splits de monolito Nest.
---

# NestJS microsservices

Nest trata microsserviços via **transporters** e `@MessagePattern` / `@EventPattern`, frequentemente em app **híbrido** (HTTP + bus). Arquitetura geral: `microservices-architecture`. Contratos HTTP: `api-design`.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **`@nestjs/microservices`** | Server/client micro |
| **Transporter** | NATS, Kafka, RMQ, Redis, TCP, gRPC — o do projeto |
| **HTTP** | Controllers REST/GraphQL na borda |
| **Config** | `@nestjs/config` + validação (Joi/Zod) |
| **ORM** | Prisma/TypeORM/Drizzle — **um** schema por serviço |

Não troque transporter sem alinhamento à plataforma.

## Organização

```text
apps/
  orders-api/          # hybrid ou HTTP
  orders-worker/       # consumer puro (opcional)
libs/
  contracts/           # DTOs/eventos versionados
  common/              # filters, interceptors
```

Ou mono-app com modules por domínio **somente** se ainda for um deploy — ao extrair serviço, separe processo e banco.

## Regras inegociáveis

- ValidationPipe (whitelist) em HTTP e payloads de mensagem.
- Authz no serviço dono do dado; JWT/guards na borda HTTP.
- Timeout e retry explícitos nos `ClientProxy`.
- Idempotência em `@EventPattern` / consumers.
- Config validada no bootstrap; secrets fora do código.
- Sem TypeORM/Prisma apontando para DB de outro serviço.

## Hybrid app

```typescript
const app = await NestFactory.create(AppModule);
app.connectMicroservice<MicroserviceOptions>({
  transport: Transport.NATS,
  options: { servers: [process.env.NATS_URL!] },
});
await app.startAllMicroservices();
await app.listen(process.env.PORT ?? 3000);
```

- HTTP para clientes externos; patterns para internos.
- Shutdown hooks: `enableShutdownHooks()` e close de clients.

## Patterns

| Decorator | Semântica |
|-----------|-----------|
| `@MessagePattern` | Request/response (RPC-style) |
| `@EventPattern` | Fire-and-forget / fan-out |

- Payloads tipados (classes + class-validator ou schemas).
- Evite payloads gigantes; passe IDs e deixe o dono carregar estado.
- Versionamento: prefixo de pattern (`orders.v1.created`) ou campos `version`.
- Erros: filtre e mapeie para RPC exceptions / DLQ — não engula.

## Clients

```typescript
ClientsModule.register([
  { name: 'ORDERS', transport: Transport.NATS, options: { /* ... */ } },
])
```

- Injete `@Inject('ORDERS') client: ClientProxy`.
- `send` / `emit` com timeout (rxjs `timeout`, `retry` conscientes).
- Circuit/fallback na camada de application service, não no controller gordo.

## Dados e outbox

- Module + repository locais.
- Outbox table + publisher (cron/worker) para eventos confiáveis.
- Inbox/dedupe key nos consumers.
- Migrations no pipeline do **serviço**, não shared migrate global.

## Observabilidade

- Interceptors de logging com correlation id (HTTP header → async context).
- OpenTelemetry se o projeto já usa.
- Health: `@nestjs/terminus` — DB, broker, downstream críticos no `/ready`.

## Testes

- Unit: services com clients mockados.
- e2e: app Nest de teste + transporter in-memory/TCP local quando viável.
- Não dependa de cluster real para o caminho feliz unitário.

## Anti-padrões

- Um único Nest “micro” deployado como bola de lama com 20 modules e 1 DB
- `emit` sem garantia/outbox em writes críticos
- Guards só no gateway e serviço interno aberto
- `any` nos payloads de pattern
- Bloquear event loop com CPU sync pesada no handler

## Critérios de conclusão

- Transporter e patterns alinhados ao domínio
- Hybrid/HTTP com validation e authz
- Clients com timeout; consumers idempotentes
- Health + shutdown limpos
- Contratos/eventos versionados e testáveis
