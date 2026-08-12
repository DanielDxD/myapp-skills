---
name: axum
description: Constrói APIs e serviços HTTP em Rust com Axum, Tower, extractors, estado compartilhado, middleware, roteamento e integração Tokio. Use ao criar ou revisar handlers Axum, routers, autenticação de rotas, WebSockets ou serviços web Rust.
---

# Axum (Rust)

Axum é roteamento + extractors sobre Tower/Hyper/Tokio. Handlers pequenos, estado explícito, composição via `Router` e middleware Tower.

## Stack típica

| Crate | Papel |
|-------|--------|
| `axum` | Router, extractors, response |
| `tokio` | Runtime async |
| `tower` / `tower-http` | Middleware (trace, cors, compression, timeout) |
| `serde` | JSON / form |
| `tracing` | Logs estruturados |
| DB | `sqlx` ou ORM já no projeto |

Preserve a stack existente. Não imponha ORMs novos sem pedido.

## Organização

```
src/
  main.rs          # boot, config, serve
  lib.rs           # app() testável
  routes/          # Router por domínio
  handlers/
  extractors/      # AuthUser, Pagination, etc.
  error.rs         # AppError + IntoResponse
  state.rs         # AppState
  services/
```

- `app()` retorna `Router` para testes sem bind de socket.
- Rotas por domínio; `main` só configura e serve.
- Estado em `State<AppState>` (`Arc` interno quando preciso).

## Regras inegociáveis

- Handlers `async` retornando `impl IntoResponse` ou `Result<T, AppError>`.
- Input via extractors tipados (`Json`, `Query`, `Path`, `State`, custom).
- Um `AppError` central com status HTTP e body estável.
- Authn/authz no servidor (extractor/middleware), nunca só no client.
- Timeouts e limites de body conscientes.
- Não bloqueie o runtime Tokio em handlers.

## Routing

```rust
Router::new()
    .nest("/api/v1", api_router)
    .route("/health", get(health))
    .layer(TraceLayer::new_for_http())
    .with_state(state)
```

- Agrupe por prefixo (`nest` / `merge`).
- Middleware de auth em `Router` aninhado, não em toda rota pública.
- Method routing explícito: `get`, `post`, `put`, `patch`, `delete`.

## Extractors

Ordem importa: extractors consomem partes do request.

- `Path<T>`, `Query<T>`, `Json<T>`, `Form<T>`
- `State<S>`, `Extension<T>` (prefira `State` para app state)
- Custom extractor para `AuthUser` / `CurrentUser`
- Rejeições de extractor → resposta 4xx coerente com `AppError`

Valide após o decode (tamanho, ranges, enums). Serde não substitui regras de negócio.

## Estado e dependências

```rust
#[derive(Clone)]
struct AppState {
    db: PgPool,
    // configs baratas / Arc<Config>
}
```

- Clone barato (handles, `Arc`).
- Sem globals mutáveis; sem `lazy_static` para DB.
- Services recebem o que precisam (pool, clients), não o `Request` inteiro.

## Erros

```rust
enum AppError {
    NotFound,
    Unauthorized,
    Forbidden,
    Validation(String),
    Internal(anyhow::Error),
}

impl IntoResponse for AppError { /* status + JSON { code, message } */ }
```

- 4xx para cliente; 5xx para falhas internas.
- Logue `Internal` com tracing; mensagem externa genérica.
- Não vaze `Debug` de anyhow ao client em produção.

## Middleware Tower

Use `tower-http` para o comum:

- `TraceLayer`, `CorsLayer`, `CompressionLayer`, `TimeoutLayer`
- `RequestBodyLimitLayer` / default body limits
- `SetSensitiveHeadersLayer` para Authorization

Middleware custom: `from_fn` / services Tower quando precisar de auth ou correlation id.

## Testes

- `tower::ServiceExt::oneshot` ou helpers do projeto sobre `Router`.
- DB de teste isolado ou mocks na borda de infra.
- Cubra 200, 401, 403, 404, validação.

## Anti-padrões

- Lógica de negócio monólito em `main.rs`
- `unwrap` em handlers
- Compartilhar `Request` parts entre extractors de forma frágil
- CORS `*` com cookies/credenciais
- Ignorar cancelamento/timeouts em clients HTTP internos
- Misturar Actix/Rocket patterns sem necessidade

## Integração

- Linguagem e crates: `rust`
- Idiomas: `rust-best-practices`
- Produção: `rust-axum-production`
- Contratos: `api-design`

## Critérios de conclusão

- `Router` composável e testável via `app()`
- Extractors e `AppError` consistentes
- Authz nas rotas sensíveis
- Middleware de trace/timeout/limites quando aplicável
- Testes dos fluxos críticos passando
