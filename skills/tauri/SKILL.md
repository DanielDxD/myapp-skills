---
name: tauri
description: Apps desktop com Tauri 2 — src-tauri, commands IPC, events, capabilities/permissions, plugins, frontend web e build seguro. Use ao criar, revisar ou depurar apps Tauri, comandos Rust, janelas ou integração webview.
---

# Tauri 2

Tauri 2 isola o privilegiado no Rust e expõe superfície mínima ao frontend. Capabilities e permissions são o contrato de segurança — não um detalhe de config.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **Tauri 2** | Runtime, janelas, IPC, plugins |
| **src-tauri** | Rust: commands, state, capabilities |
| **Frontend** | Web (React/Vue/Svelte/etc. do projeto) |
| **Plugins** | FS, dialog, shell, http, etc. com permissões explícitas |

Não escreva padrões de Tauri 1 (`allowlist` legado) em apps novos. Preserve a estrutura já adotada pelo projeto.

## Organização

```
src/                 # frontend
src-tauri/
  src/
    lib.rs           # #[cfg_attr] run, plugins, invoke_handler
    main.rs
    commands/        # commands por domínio
  capabilities/      # permissões por janela/contexto
  tauri.conf.json
  Cargo.toml
```

- Commands finos; lógica pesada em módulos Rust testáveis.
- Estado compartilhado: `tauri::State` / `Mutex` / `RwLock` conscientes de blocking.
- Frontend fala só via `invoke` / events — sem assumir Node APIs.

## Commands e IPC

```rust
#[tauri::command]
fn greet(name: String) -> Result<String, String> {
    if name.is_empty() {
        return Err("name required".into());
    }
    Ok(format!("Hello, {name}!"))
}

// lib.rs
tauri::Builder::default()
    .plugin(tauri_plugin_shell::init())
    .invoke_handler(tauri::generate_handler![greet])
    .run(tauri::generate_context!())
    .expect("error while running tauri application");
```

```ts
import { invoke } from "@tauri-apps/api/core";

const msg = await invoke<string>("greet", { name: "Ada" });
```

- Commands: `Result<T, E>` serializável; erros seguros para o UI.
- Valide input no Rust; o webview é hostil.
- Prefer `async` commands para I/O; não bloqueie o runtime à toa.
- Events (`emit` / `listen`) para push; commands para request/response.

## Capabilities e permissões

- Declare em `capabilities/` o que cada janela/label pode fazer.
- Princípio do menor privilégio: FS, shell e HTTP só nos paths/comandos necessários.
- Plugins só entram se houver permission correspondente.
- Revise capabilities em todo feature que toque disco, rede ou processos.

## Frontend

- Use APIs `@tauri-apps/api` (v2) e plugins oficiais alinhados.
- Detecte runtime Tauri antes de chamar IPC (SSR/preview web).
- UX desktop: menus, atalhos, close-to-tray só se o produto pedir — sem inflar o shell.
- Assets e CSP: configure em `tauri.conf.json`; evite `unsafe-inline` sem necessidade.

## Config e build

- Identifiers, janelas e security em `tauri.conf.json` / arquivos de capabilities.
- Secrets e signing: fora do repo; CI com secrets do store.
- Dev: `tauri dev` com URL do bundler do front.
- Prod: `tauri build`; teste o artefato instalável no OS alvo.
- Multiplataforma: paths e FS via APIs Tauri, não hardcode de Unix/Windows.

## Segurança

- Nunca exponha command genérico de shell/FS sem allowlist rígida.
- Normalize e valide paths; bloqueie path traversal.
- Trate o frontend como não confiável (XSS no webview = RCE se as permissões forem amplas).
- Atualizações: prefer mechanism oficial/plugin de updater com assinatura.

## Anti-padrões

- Portar mentalidade de Electron/Node para o core Rust
- Capabilities permissivas “para funcionar” deixadas no merge
- Commands que aceitam path/comando arbitrário do UI
- Bloquear UI thread com I/O síncrono longo
- Misturar padrões Tauri 1 allowlist em projeto v2
- Logar tokens/PII no lado Rust ou no webview

## Fluxo ao implementar uma feature

1. Inspecione `tauri.conf.json`, capabilities e commands existentes.
2. Defina command/event + tipos de payload.
3. Restrinja permissions/capabilities ao mínimo.
4. Implemente Rust + chamada `invoke` no front.
5. Teste em `tauri dev` e valide o build se a superfície nativa mudou.
6. Revise paths, rede e shell expostos.

## Critérios de conclusão

- IPC tipado com erros tratados no UI
- Capabilities mínimas para a feature
- Input validado no Rust
- Sem APIs privilegiadas desnecessárias no frontend
- Build/dev do projeto funcionando no escopo da mudança
- Sem regressão de superfície de ataque (FS/shell/http)
