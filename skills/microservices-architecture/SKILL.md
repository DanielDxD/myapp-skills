---
name: microservices-architecture
description: Desenha e revisa arquitetura de microsserviços — bounded contexts, comunicação síncrona/assíncrona, dados, consistência, gateways, observabilidade e trade-offs frente a monolito. Use ao planejar serviços, splits de domínio, eventos, sagas ou revisar acoplamento entre serviços.
---

# Arquitetura de microsserviços

Microsserviço é um limite de negócio deployável de forma independente — não um CRUD por tabela. Prefira o menor número de serviços que preserve autonomia e clareza.

Para implementação: `golang-microservices`, `nestjs-microservices`, `rust-microservices`. Contratos HTTP: `api-design`. Go-live: `production-readiness`.

## Quando (não) usar

**Faz sentido quando:** times autônomos, escalas distintas, ciclos de release diferentes, falhas isoláveis.

**Evite quando:** domínio incerto, time pequeno, latência forte entre “serviços”, operação sem observabilidade. Comece modular no monolito e extraia com evidência.

## Bounded contexts

- Um serviço = um contexto com linguagem ubíqua própria.
- API pública estável; modelo interno privado.
- Proibido: dois serviços compartilhando o mesmo banco de escrita.
- Evite “distributed monolith”: chamadas síncronas em cascata para todo write.

## Comunicação

| Estilo | Uso |
|--------|-----|
| **HTTP/gRPC sync** | Queries, comandos que precisam de resposta imediata |
| **Mensageria** (fila/bus) | Eventos de domínio, desacoplar writers |
| **Outbox / inbox** | Publicar eventos de forma confiável com a mesma transação local |

Regras:

- Timeouts, retries com backoff/jitter e idempotência em todo client.
- Circuit breaker / bulkhead onde a falha do downstream derruba o produto.
- Contratos versionados (`v1`, schema registry para eventos).
- Correlation / trace id em todas as bordas.
- Não invente RPC interno sem autenticação serviço-a-serviço.

## Dados e consistência

- Database per service (schema ou instância).
- Consistência eventual entre contextos; documente o modelo mental.
- Sagas/process managers para workflows multi-serviço; evite 2PC distribuído.
- Leituras compostas: BFF, vista materializada ou orquestração explícita — não N+1 de HTTP no client mobile.
- Migrações de dados entre serviços: dual-write só com plano e métricas.

## Topologia

```text
Client → Gateway/BFF → Serviços → (DB | Queue | Cache)
                         ↘ workers / consumers
```

- Gateway: auth de borda, rate limit, roteamento — lógica de negócio mínima.
- BFF quando canais (web/mobile) precisam de agregação diferente.
- Sidecars/mesh só se a plataforma já operar isso; não adicione complexidade cedo.

## Observabilidade e operação

- Logs estruturados + trace distribuído + métricas RED/USE.
- Health `/live` e `/ready` por serviço.
- SLOs e alertas por serviço crítico, não só CPU.
- Deploy independente com rollback; feature flags para mudanças arriscadas.
- Contratos testados (pact/contract ou suite de integração na borda).

## Segurança

- Zero trust interno: mTLS ou tokens serviço-a-serviço.
- Authz no serviço dono do recurso (não só no gateway).
- Secrets em vault/env; sem credenciais em imagens.
- Blast radius: least privilege em IAM/rede/filas.

## Anti-padrões

- Serviço por entidade CRUD (“UserService”, “OrderItemService” sem contexto)
- Shared DB e joins cross-service
- Chatty sync sem cache/agregação
- Eventos que são só “RPC assíncrono” acoplado
- Distribuição prematura sem domínio estável

## Critérios de conclusão

- Limites de serviço alinhados a domínio, não a tech
- Comunicação, timeouts e idempotência definidos
- Dados sem shared write DB
- Observabilidade e health por serviço
- Trade-off vs monolito/modular documentado quando relevante
