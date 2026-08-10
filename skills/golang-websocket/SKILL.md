---
name: golang-websocket
description: WebSockets em Go com Gin e gorilla/websocket — upgrade, hub, broadcast, ping/pong, auth no handshake, backpressure e shutdown. Use ao criar ou revisar conexões tempo real, chats, feeds ou canais WS em APIs Go.
---

# Golang WebSocket

Use WebSocket quando o domínio exigir push real. Autentique no handshake; trate cada conexão como um lifecycle com leitura, escrita e saída explícitas.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **Gin** | Rota de upgrade HTTP → WS |
| **gorilla/websocket** | Conn, Upgrader, control frames |
| **Hub** | Registro de clients, broadcast, unregister |
| **goroutines** | Read/write pumps por conexão (ver `golang-goroutines`) |

## Upgrade com Gin

```go
var upgrader = websocket.Upgrader{
    ReadBufferSize:  1024,
    WriteBufferSize: 1024,
    CheckOrigin: func(r *http.Request) bool {
        return allowedOrigin(r.Header.Get("Origin"))
    },
}

func serveWS(hub *Hub, c *gin.Context) {
    // Auth antes do upgrade (cookie/token/query conforme o projeto)
    userID, err := authenticateWS(c)
    if err != nil {
        c.AbortWithStatus(http.StatusUnauthorized)
        return
    }

    conn, err := upgrader.Upgrade(c.Writer, c.Request, nil)
    if err != nil {
        return
    }

    client := NewClient(hub, conn, userID)
    hub.Register(client)
    go client.WritePump()
    go client.ReadPump()
}
```

- `CheckOrigin` explícito; nunca `return true` em produção sem critério.
- Auth **antes** do upgrade quando possível; revalide por mensagem se o domínio exigir.
- Após `Upgrade`, não escreva no `gin.Context` response.

## Modelo hub / client

```
Client --register/unregister--> Hub
Hub --broadcast--> Client.send (channel bufferizado)
Client.WritePump <-- Client.send
Client.ReadPump --> Hub.broadcast / handlers
```

- Um writer por `Conn` (gorilla não é safe para writes concorrentes).
- Read pump: lê → processa → encaminha; ao sair, unregister + close.
- Write pump: recebe do channel → escreve; ping periódico; close ao drenar/cancelar.
- Buffer em `send`: tamanho finito; política de drop ou desconectar se cheio (backpressure).

## Ping, pong e timeouts

- `SetReadDeadline` + `PongHandler` que estende o deadline.
- Write pump envia `PingMessage` em ticker.
- `SetWriteDeadline` por frame.
- Limite `ReadLimit` para frames abusivos.

## Mensagens e autorização

- Contrato de mensagem versionado (JSON com `type` discriminado).
- Authz por canal/room no servidor; nunca confiar só no client.
- Valide e limite tamanho/taxa por conexão.
- Erros de protocolo: feche com close code adequado; logue sem PII.

## Shutdown gracioso

1. Pare de aceitar novos upgrades.
2. Hub broadcast de close / cancele context do hub.
3. Cada client: fechar `send`, flush write pump, `Conn.Close`.
4. `WaitGroup` ou errgroup até zerar conexões (com timeout global).

## Integração

- Rota Gin atrás do mesmo middleware de request id/logging quando útil.
- Não bloqueie o hub loop com I/O pesado; delegue a workers.
- Escala multi-instância: hub in-process não basta — pub/sub (Redis etc.) se o deploy for horizontal.

## Anti-padrões

- `CheckOrigin: func(...) bool { return true }` sem análise
- Várias goroutines escrevendo no mesmo `Conn`
- Channel `send` sem bound e sem política de overflow
- Auth só na primeira mensagem HTTP sem amarrar à conexão
- Ignorar ping/pong (conexões zumbi)
- Usar WS para CRUD simples que seria HTTP

## Critérios de conclusão

- Upgrade seguro com origin e auth definidos
- Um writer por conexão; read/write pumps claros
- Ping/pong e deadlines configurados
- Backpressure ou desconexão sob sobrecarga
- Authz por mensagem/canal quando aplicável
- Shutdown não deixa conexões órfãs
