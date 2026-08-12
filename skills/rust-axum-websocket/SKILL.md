---
name: rust-axum-websocket
description: WebSockets em Rust com Axum — upgrade, Message loop, broadcast hub, ping/pong, auth no handshake, backpressure e shutdown. Use ao criar ou revisar conexões tempo real, chats, feeds ou canais WS em APIs Axum.
---

# Rust Axum WebSocket

Use WebSocket quando o domínio exigir push real. Autentique no handshake; cada conexão é um lifecycle async com leitura, escrita e cancelamento via `CancellationToken`/drop do channel.

HTTP geral Axum: skill `axum`. Produção (shutdown, limites): `rust-axum-production`.

## Stack e escopo

| Crate | Papel |
|-------|--------|
| **axum** | `WebSocketUpgrade`, `Message`, rota WS |
| **tokio** | Tasks, `mpsc` / `broadcast` |
| **futures** | `StreamExt` / `SinkExt` no socket |
| **Hub** | Registro de clients, fan-out |

Feature Axum: `ws`. Preserve a stack do projeto.

## Upgrade e handler

```rust
use axum::{
    extract::{
        ws::{Message, WebSocket, WebSocketUpgrade},
        State,
    },
    response::IntoResponse,
};

async fn ws_handler(
    ws: WebSocketUpgrade,
    State(hub): State<Hub>,
    // extratores de auth (Query/Header/Extension) ANTES do upgrade
) -> impl IntoResponse {
    ws.on_upgrade(move |socket| client_session(socket, hub))
}

async fn client_session(socket: WebSocket, hub: Hub) {
    let (mut sender, mut receiver) = socket.split();
    // registrar no hub; spawn write loop; ler até Close/erro
}
```

- Auth **antes** de `on_upgrade` (cookie/header/query validados no handler HTTP).
- Após o upgrade, não misture response HTTP normal no mesmo fluxo.
- `SameOrigin` / checagem de `Origin` quando o browser for cliente.

## Modelo hub / client

```
Client --subscribe--> broadcast::Sender / Hub
Hub --fan-out--> Client mpsc (bounded)
Write task <-- mpsc / broadcast
Read task --> Hub / domain handlers
```

- Um writer por conexão (serialize writes numa task).
- Channel **bounded** para backpressure; política se cheio: drop mensagem ou desconectar.
- `tokio::select!` entre leitura, eventos do hub e shutdown.

## Ping, pong e mensagens

- Trate `Message::Ping` / `Pong` / `Close`.
- Envie pings periódicos se o domínio/proxy exigir keepalive.
- Texto JSON com `type` discriminado (`serde`); binário só com contrato claro.
- Limite tamanho de frame (config do server / rejeição no app).
- Revalide authz por mensagem/canal quando o risco existir.

## Authz e multi-tenant

- Identity amarrada à conexão no upgrade; não confie em `room_id` do client sem checar membership.
- Rooms: mapa `room → subscribers` no hub, ou canais `broadcast` por room.
- Não vaze eventos cross-tenant.

## Shutdown gracioso

1. Pare de aceitar upgrades (drain do server Axum/Hyper).
2. Sinalize hub (`CancellationToken` / fechar sender).
3. Clients: enviar `Close`, flush, drop tasks.
4. `JoinSet` / contagem até zerar (com timeout global).

## Integração

- Compartilhe `State<Hub>` no `Router` Axum.
- Não bloqueie o hub com I/O pesado — `spawn` workers / `spawn_blocking` só se inevitável.
- Multi-instância: hub in-process não basta; use Redis/NATS/etc. para fan-out horizontal.

## Anti-padrões

- Upgrade sem auth nem checagem de Origin
- `UnboundedSender` sem limite sob load
- Várias tasks escrevendo no mesmo `SplitSink` sem mutex/canal único
- Ignorar `Message::Close` e tasks órfãs
- WS para CRUD simples que seria HTTP
- `unwrap` em path de conexão

## Critérios de conclusão

- Upgrade com auth (e Origin quando browser)
- Read/write loops claros; um writer por socket
- Backpressure ou desconexão sob sobrecarga
- Authz por room/canal quando aplicável
- Shutdown não deixa tasks órfãs
- Contrato de mensagem versionado e validado
