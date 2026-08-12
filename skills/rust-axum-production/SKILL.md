---
name: rust-axum-production
description: Endurece serviços Axum/Rust para produção com config tipada, graceful shutdown, health/readiness, timeouts, limites, tracing/métricas, segurança HTTP, Docker e deploy. Use antes de release, hardening, revisão operacional ou quando o usuário pedir Axum production-ready.
---

# Rust Axum em produção

Um serviço Axum pronto para produção é previsível sob falha, observável e seguro por padrão. Aplique junto com `axum` e, no go-live geral, `production-readiness`.

## Boot e configuração

- Carregue config de env (e arquivos se o projeto usar) **uma vez** no startup.
- Valide com tipos (`figment`, `config`, `envy`, ou parse manual) — falhe rápido se faltar secret/DB URL.
- Separe `development` / `staging` / `production` (logs, auto-migrate, verbosidade de erro).
- Não leia env dentro de handlers.

```text
APP_ENV=production
DATABASE_URL=...
RUST_LOG=info,tower_http=info,my_app=info
LISTEN_ADDR=0.0.0.0:8080
```

## Processo e shutdown

- Bind explícito (`0.0.0.0` em container; localhost só em dev se desejado).
- **Graceful shutdown** em SIGTERM/SIGINT: pare de aceitar conexões, drenar requests, fechar pool.
- Timeout de shutdown definido (ex.: 30s); logue o que não drenou.
- `tokio::spawn` de background tasks com cancelamento no shutdown (`CancellationToken` / `watch` / `AbortHandle`).

## Health e readiness

| Endpoint | Significado |
|----------|-------------|
| `/health` ou `/live` | Processo up |
| `/ready` | Dependências críticas ok (DB ping) |

- Liveness não deve falhar só porque o DB está lento (evita kill loop), salvo política do orquestrador.
- Readiness falha se não puder servir tráfego.
- Não exige auth; não vaza internals.

## Timeouts, limites e resiliência

- `TimeoutLayer` global ou por grupo de rotas.
- Limite de body (`DefaultBodyLimit` / `RequestBodyLimitLayer`).
- Timeouts em **todos** os clients HTTP/DB externos.
- Retries só em erros transitivos idempotentes, com backoff e jitter.
- Circuit breaker / degradação quando o domínio exigir.
- Pool DB: `max_connections` alinhado a CPU/latência; não copie defaults cegamente em multi-réplica.

## Observabilidade

- `tracing` + `tracing-subscriber` (JSON em produção).
- `TraceLayer` do `tower-http` com request id / correlation id.
- Propague `traceparent` / request id se o ecossistema usar.
- Métricas: latência, taxa de erro, saturação (Prometheus via `metrics` ou o padrão do projeto).
- Spans em handlers/services relevantes — sem logar body com PII/secrets.
- Erros 5xx: log com causa; resposta externa genérica.

## Segurança

- TLS na borda (proxy) ou no processo se for o padrão do deploy.
- CORS explícito: origins allowlist; nunca `*` com credentials.
- Headers: `Strict-Transport-Security`, `X-Content-Type-Options`, frame policy conforme o app.
- Cookies: `HttpOnly`, `Secure`, `SameSite` adequados.
- Auth: JWT/session validados em extractor; authz por recurso.
- Secrets só em env/secret manager; `cargo audit` / SCA no CI.
- Rate limit em login e endpoints caros (middleware ou API gateway).

## HTTP e contratos

- JSON de erro estável: `{ "code", "message", "request_id"? }`.
- Status corretos; não mascare 500 como 200.
- Paginação com teto; rejeite `limit` abusivo.
- Uploads: tipo/tamanho validados; armazene fora do disco efêmero sem plano.

## Banco e migrations

- Migrations no pipeline **antes** de liberar tráfego (job ou init controlado).
- Evite `migrate on boot` em multi-réplica sem lock/liderança.
- Conexões com runtime + statement timeouts quando o driver permitir.
- Rollback/forward strategy documentada para mudanças destrutivas.

## Container e deploy

- Docker multi-stage: build `cargo build --release`, runtime distroless/slim com certs.
- Usuário não-root; binário estático ou deps mínimas.
- `WORKDIR`, `EXPOSE`, `STOPSIGNAL SIGTERM`.
- Healthcheck do orquestrador apontando para `/ready` ou `/health`.
- Recursos: CPU/memory requests/limits; graceful period > timeout de shutdown.
- Build: cache de registry Cargo no CI; `Cargo.lock` commitado em bins.

## Performance em produção

- Release builds (`--release`); LTO/codegen só com evidência.
- Evite allocations quentes desnecessárias; meça com profiling sob carga.
- Compressão HTTP se CPU sobrar e payload for grande.
- Conexões keep-alive atrás do proxy; HTTP/2 quando a borda suportar.

## Checklist go-live

- [ ] Config tipada; boot falha se inválida
- [ ] Graceful shutdown testado (SIGTERM)
- [ ] `/health` e `/ready` corretos
- [ ] Timeouts + body limits ativos
- [ ] Tracing JSON + request id
- [ ] Métricas/alertas básicos
- [ ] CORS/headers/authz revisados
- [ ] Migrations no pipeline
- [ ] Imagem não-root; `cargo audit` limpo o suficiente
- [ ] Smoke test pós-deploy e plano de rollback

## Anti-padrões

- `RUST_LOG=debug` em produção com PII
- Panics não capturados derrubando o processo em request handling (prefira `Result`)
- Uma única instância com estado em memória sem sticky/session store
- Abrir CORS no mundo para “testar rápido” e esquecer
- Deploy sem readiness (tráfego em app ainda migrando)

## Critérios de conclusão

- Serviço sobe, serve health/ready e derruba limpo com SIGTERM
- Falhas de dependência aparecem em logs/métricas, não como silêncio
- Superfície HTTP endurecida (limites, timeouts, CORS, authz)
- Pipeline de migrate + deploy + smoke definido
- Sem secrets no artefato ou logs
