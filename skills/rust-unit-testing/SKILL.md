---
name: rust-unit-testing
description: Testes unitários e de integração em Rust com cargo test, #[cfg(test)], asserts, table-driven, async Tokio, fakes por traits e testes de Router Axum. Use ao adicionar testes Rust, cobrir domínio/handlers, eliminar flakiness ou definir estratégia de teste.
---

# Rust Unit Testing

Teste comportamento e invariantes, não detalhes privados de implementação. Prefira testes determinísticos e table-driven a suítes frágeis.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **`cargo test`** | Unit, doc e integração |
| **`#[cfg(test)]`** | Módulos de teste no mesmo arquivo/crate |
| **`tests/`** | Integração contra a API pública do crate |
| **`tokio::test`** | Handlers e serviços async |
| **traits + fakes** | Isolar DB, clock, HTTP clients |

Use `pretty_assertions`, `rstest`, `mockall` ou `proptest` **somente** se o projeto já depender deles. Não introduza framework de mock sem necessidade — fakes manuais bastam na maioria dos casos.

## O que priorizar

- Regras de domínio e policies (authz, ownership)
- Parsing/validação na borda (`Result`, códigos de erro)
- Mutações e estados inválidos que o tipo deveria impedir
- Regressões já ocorridas (bug → teste)
- Handlers HTTP críticos quando o crate expõe `Router` / `app()`

## Layout

```text
src/lib.rs
src/domain/foo.rs          # #[cfg(test)] mod tests { ... }
tests/
  api_todos.rs             # integração / HTTP
```

- Unitários: perto do código, acessam `pub(crate)` quando útil.
- Integração: só API `pub`; compilam como crates separados.
- Doc tests (`/// ````) para exemplos públicos que devem continuar compilando.

## Anatomia de um teste

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn parse_cursor_rejects_invalid_input() {
        let err = parse_cursor("%%%").unwrap_err();
        assert!(matches!(err, CursorError::Invalid));
    }
}
```

- Nome: `sujeito_comportamento_condição` ou frase clara em snake_case.
- Um comportamento principal por teste (ou por linha da tabela).
- Prefira `assert_eq!` / `assert_ne!` / `assert!` / `matches!` a prints.
- `#[should_panic]` só para panics intencionais de invariante — não para erros de domínio (`Result`).

## Table-driven

```rust
#[test]
fn parse_cursor_cases() {
    struct Case {
        name: &'static str,
        input: &'static str,
        expected: Result<u64, CursorError>,
    }

    let cases = [
        Case { name: "empty", input: "", expected: Err(CursorError::Invalid) },
        Case { name: "ok", input: "10", expected: Ok(10) },
    ];

    for case in cases {
        let got = parse_cursor(case.input);
        assert_eq!(got, case.expected, "case: {}", case.name);
    }
}
```

- Cases independentes; sem ordem global implícita.
- Mensagem de assert com o nome do case.
- Extraia helpers com cuidado para não esconder o arrange/act/assert.

## Async

```rust
#[tokio::test]
async fn create_todo_requires_auth() {
    let state = test_state().await;
    let res = create_todo(State(state), /* ... */).await;
    assert!(matches!(res, Err(AppError::Unauthorized)));
}
```

- Use o runtime do projeto (`#[tokio::test]`).
- Evite sleeps; espere condições ou use fakes síncronos.
- Em testes de concorrência, prefira canais/barriers a timing frágil.

## Fakes e isolation

- Defina traits na borda (`TodoRepo`, `Clock`, `EmailSender`).
- Em teste, injete structs fake em memória.
- Não mocke o sistema sob teste — mocke dependências externas.
- Arquivos: `tempfile::TempDir` quando o projeto já usa; limpe ao dropar.
- Relógio: injete `Clock` em vez de `Utc::now()` espalhado.

## Axum / HTTP

Quando o app expõe `fn app(state) -> Router`:

```rust
#[tokio::test]
async fn health_ok() {
    let app = app(test_state().await);
    let response = app
        .oneshot(
            Request::builder()
                .uri("/health")
                .body(Body::empty())
                .unwrap(),
        )
        .await
        .unwrap();

    assert_eq!(response.status(), StatusCode::OK);
}
```

- Use `tower::ServiceExt::oneshot` (ou helpers do projeto).
- Cubra status e corpo relevante (`401`, `403`, `404`, validação), não só o happy path.
- DB de teste isolado (SQLite, container, ou fake de repo) — sem compartilhar estado sujo entre testes.

## Asserts e erros

- Compare com `PartialEq` sempre que possível.
- Para erros: `assert!(matches!(err, ...))` ou `assert_eq!(err.to_string(), ...)`.
- Evite `unwrap` sem contexto em setup; use `expect("fixture ...")` se precisar falhar rápido.
- Não engula `Result` com `let _ =`.

## Flakiness e CI

- Sem rede real, relógio real ou ordem de hashmap não determinística em asserts.
- Seeds fixos em aleatoriedade (`proptest` / RNG).
- `cargo test` no escopo alterado; no CI: `cargo test` (+ `--workspace` se aplicável).
- Paralelo do Cargo é ok; se houver estado global, isole com mutex ou desabilite parallel só no teste problemático (`--test-threads=1` como último recurso).

## Anti-padrões

- Testar só getters/setters triviais
- Snapshots gigantes sem revisão
- `unwrap` em cascata que torna falhas ilegíveis
- Dormir (`sleep`) para “esperar” async
- Cobrir linhas em vez de riscos
- Duplicar lógica de produção no teste em vez de observar comportamento

## Integração com outras skills

- Linguagem e tooling: `rust`
- Idioms / Clippy: `rust-best-practices`
- Handlers e `Router`: `axum`
- Serviço endurecido: `rust-axum-production`

## Critérios de conclusão

- Casos de risco da mudança cobertos por `cargo test`
- Nomes legíveis; asserts claros
- Fakes nas bordas; sem flakiness óbvia
- Async/HTTP testados com runtime e `Router` do projeto quando aplicável
- CI ou comando local documentado passa no escopo tocado
