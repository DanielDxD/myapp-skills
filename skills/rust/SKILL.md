---
name: rust
description: Desenvolve software em Rust com ownership, lifetimes, traits, error handling, async, crates e tooling Cargo. Use ao criar ou revisar projetos Rust, bibliotecas, CLIs, serviços, ou quando o usuário pedir código, refactors ou arquitetura em Rust.
---

# Rust

Escreva Rust idiomático, seguro e explícito. O compilador é aliado: modele o domínio em tipos para que estados inválidos sejam difíceis de representar.

## Tooling

- `cargo new` / `cargo init`, workspaces quando houver múltiplos crates.
- `cargo check` no ciclo rápido; `cargo test`, `cargo clippy`, `cargo fmt` antes de concluir.
- Edição 2021+ (ou a do projeto). Respeite `rust-version` / MSRV se existir.
- Features Cargo explícitas; evite ativar tudo “por precaução”.
- Prefira crates maduros já adotados pelo projeto; não troque ecossistema sem motivo.

## Organização

```
crate/
  Cargo.toml
  src/
    main.rs | lib.rs
    ...
  tests/          # integração
  examples/
  benches/        # só com critério
```

Em apps maiores:

```
crates/
  api/
  domain/
  infra/
```

- `lib.rs` / `main.rs` finos; módulos por domínio.
- Separe domínio puro de I/O (DB, HTTP, FS).
- Visibilidade mínima: `pub(crate)` > `pub` desnecessário.

## Ownership e borrowing

- Prefira emprestar (`&T` / `&mut T`) a clonar.
- Clone só quando a semântica exigir ownership independente.
- `Arc`/`Rc` para compartilhamento consciente; `Mutex`/`RwLock` com seções críticas curtas.
- Evite ciclos `Rc`/`RefCell` sem `Weak`.
- Lifetimes: nomeie quando esclarecer; elida quando óbvio.
- Não lute com o borrow checker com `unsafe` — reestruture dados/APIs.

## Tipos e API

- Newtypes para IDs e valores de domínio (`UserId(Uuid)`).
- Enums para estados fechados; evite `bool` + strings mágicas.
- `Option`/`Result` em vez de sentinelas ou panics em API de biblioteca.
- Traits para comportamento compartilhado; derive (`Debug`, `Clone`, `serde`) com parcimônia.
- Generics com bounds claros; evite generics decorativos.

## Erros

- Bibliotecas: tipos de erro próprios ou `thiserror`.
- Aplicações: `anyhow` (ou equivalente do projeto) nas bordas.
- Converta erros nas fronteiras; preserve contexto (`with_context`).
- `panic!` só para invariantes de programação, nunca para input de usuário ou I/O esperado.
- Não use `unwrap`/`expect` em caminhos de produção sem justificativa local.

## Async

- Runtime do projeto (`tokio` é o padrão comum).
- `.await` em I/O; não bloqueie o runtime com CPU pesada — use `spawn_blocking` ou thread dedicada.
- `Send + 'static` em tasks spawnadas; entenda bounds antes de “corrigir” com `Mutex` demais.
- Cancele com respeito a cooperative cancellation; timeouts em I/O externo.
- Prefira `async fn` e `Future` explícitos a macros obscuras.

## Persistência e I/O

- Encapsule acesso a DB/FS atrás de traits ou módulos de infra.
- Migrations versionadas quando houver schema.
- SQL: statements parametrizados; nunca concatene input em query.
- Serde para JSON/config; valide na borda.

## Testes

- Unitários ao lado do código ou em `#[cfg(test)]`.
- Integração em `tests/`.
- Prefira testes de comportamento e propriedades de domínio a detalhes privados.
- `cargo test` deve passar nas mudanças tocadas.

## Anti-padrões

- `unwrap` em cascata
- `clone` para silenciar borrow checker
- `unsafe` sem invariante documentada
- Crates abandonados ou overlapping (dois HTTP clients, dois runtimes)
- Ignorar warnings e Clippy no CI local
- Modelar tudo como `String` / `HashMap<String, Value>`

## Integração com outras skills

- Idiomas e qualidade contínua: `rust-best-practices`
- Testes unitários e de integração: `rust-unit-testing`
- HTTP com Axum: `axum`
- Axum endurecido para produção: `rust-axum-production`
- Contratos HTTP genéricos: `api-design`

## Critérios de conclusão

- Código compila com warnings relevantes tratados
- Ownership/API claros; sem `unsafe` injustificado
- Erros tipados nas bordas; sem panic em input externo
- Testes dos caminhos críticos passando
- `clippy` e `fmt` ok no escopo alterado
