---
name: rust-best-practices
description: Aplica melhores práticas Rust — idioms, Clippy, API design, error handling, unsafe consciente, testes e hygiene de crates. Use ao revisar código Rust, endurecer qualidade, refatorar APIs públicas ou quando o usuário pedir best practices, idiomatic Rust ou limpeza de smells.
---

# Melhores práticas Rust

Esta skill afia qualidade e idioms. Para fundamentos da linguagem use `rust`; para Axum use `axum` / `rust-axum-production`.

## Checklist contínuo

Em todo PR/escopo tocado:

1. `cargo fmt`
2. `cargo clippy --all-targets -- -D warnings` (ou o padrão do CI)
3. `cargo test`
4. Sem `TODO`/`unwrap` novos sem tracking

Trate warnings como erros no código novo.

## Idioms

- “Parse, don’t validate”: converta input em tipos válidos cedo.
- Prefira iterators e combinators claros a índices manuais quando legível.
- `match` exaustivo; `if let` só para um braço relevante.
- Evite `ref` legado; use destructuring moderna.
- `Default`, builders ou funções `new` com pré-condições claras.
- Naming: `snake_case`, tipos `UpperCamelCase`, consts `SCREAMING_SNAKE` quando convencional.
- Documente `pub` items com `///` e exemplos quando a API for externa.

## Design de API (crates/libs)

- Minimize superfície `pub`.
- Breaking changes conscientes em semver; features Cargo para opcionais.
- Aceite `impl AsRef<Path>` / `impl Into<String>` só quando ergonomia real valer a complexidade.
- Retorne tipos concretos ou `impl Trait` estável; evite `Box<dyn Error>` em libs públicas se puder tipar.
- Não exponha tipos de dependências internas sem necessidade (evita coupling semver).
- `#[non_exhaustive]` em enums/structs públicos que possam crescer.

## Ownership idiomático

- API que precisa guardar dado: aceite `impl Into<T>` ou ownership clara, não `&String`.
- Prefira `&str` a `&String`; `&[T]` a `&Vec<T>`.
- Interior mutability (`Cell`/`RefCell`/`Mutex`) só com razão; documente thread-safety (`Send`/`Sync`).
- Evite `clone` em hot paths; meça antes de otimizar micro.

## Erros e panics

| Contexto | Preferência |
|----------|-------------|
| Lib | `thiserror`, erros enum |
| App borda | `anyhow` / `eyre` se já adotado |
| Invariante interna | `expect` com mensagem que diga a invariante |
| Input externo | `Result`, nunca panic |

- Encadeie contexto; não descarte `source`.
- `clippy::unwrap_used` / `expect_used` em áreas sensíveis quando o projeto permitir.

## Unsafe

- Último recurso. Cada bloco `unsafe` com comentário de **invariante de segurança**.
- Encapsule em API safe mínima.
- Não copie `unsafe` de Stack Overflow sem entender aliasing/lifetimes.
- Preferível: crates auditadas (`bytes`, `zerocopy`, etc.) a `unsafe` caseiro.

## Concorrência

- Compartilhe com `Arc`; mute com `Mutex`/`RwLock` ou canais.
- Prefira message passing quando reduzir contensão.
- Evite `std::thread::sleep` em async; use `tokio::time`.
- Detecte deadlocks potenciais (lock ordering).
- `Send`/`Sync` errors: corrija o design, não “escape” com wrappers injustificados.

## Dependências

- Poucas, conhecidas, mantidas.
- `cargo deny` / `cargo audit` quando o repo já usa; senão, audite upgrades críticos.
- Feature flags: ative só o necessário (`rt-multi-thread`, TLS stack, etc.).
- Evite duplicar crates semânticos (dois parsers JSON, dois TLS).

## Testes e qualidade

- Unit + integração; property tests (`proptest`) para parsers/invariantes.
- Snapshot tests só se o time já usa e snapshots são revisados.
- Benches com `criterion` apenas para regressões reais.
- CI: fmt + clippy + test (+ MSRV se declarado).

## Smells a eliminar

- `String` como “tipo universal”
- `mut` e reassigns onde binding nova seria mais clara
- Módulos `utils` dump
- Comentários que narram o óbvio em vez de invariantes
- `#[allow(clippy::...)]` amplo sem justificativa
- Código comentado “por se acaso”

## Critérios de conclusão

- Clippy limpo no escopo (ou allows locais justificados)
- API/`pub` mínima e documentada quando externa
- Erros idiomáticos; panics só em invariantes
- `unsafe` ausente ou encapsulado com invariantes
- Testes cobrem regressões do refactor
