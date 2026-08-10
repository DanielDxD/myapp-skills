---
name: golang-scylladb
description: Integração Go com ScyllaDB via gocql/gocqlx — session shard-aware, prepared statements, paging, retries, mapping e repositórios. Use ao conectar apps Go a Scylla, escrever repositórios CQL, tunar consistency ou depurar latência/erros do driver.
---

# Golang + ScyllaDB

O driver é parte do contrato de performance. Session única, queries preparadas, paging explícito e modelagem conforme `scylladb`.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **github.com/scylladb/gocql** | Driver (fork Scylla, token/shard-aware) |
| **gocqlx** | Mapping structs ↔ CQL, builders (se o projeto usar) |
| **Session** | Pool de conexões; compartilhada no processo |
| **Repository** | Encapsula CQL; handlers não montam query solta |

Preserve o driver já adotado. Em apps novos: `scylladb/gocql` (+ `gocqlx` quando mapping ajudar). Modelagem de tables: skill `scylladb`.

## Session e config

```go
cluster := gocql.NewCluster(hosts...)
cluster.Keyspace = "app"
cluster.Consistency = gocql.LocalQuorum
cluster.PoolConfig.HostSelectionPolicy = gocql.TokenAwareHostPolicy(
    gocql.RoundRobinHostPolicy(),
)
cluster.Timeout = 2 * time.Second
cluster.ConnectTimeout = 5 * time.Second

session, err := cluster.CreateSession()
if err != nil {
    return err
}
defer session.Close()
```

- Uma `Session` por processo/app; injete via deps — não abra session por request.
- Token-aware (+ shard-aware quando suportado/configurado) para ir ao nó certo.
- Timeouts alinhados ao SLA; consistency por operação quando diferir do default.
- Auth/TLS via config do cluster em produção.

## Prepared statements e queries

```go
stmt := `SELECT message_id, author_id, body, sent_at
         FROM messages_by_room
         WHERE room_id = ? AND sent_at < ?
         LIMIT ?`

iter := session.Query(stmt, roomID, before, limit).
    WithContext(ctx).
    PageSize(100).
    Iter()

var row Message
for iter.Scan(&row.MessageID, &row.AuthorID, &row.Body, &row.SentAt) {
    // ...
}
if err := iter.Close(); err != nil {
    return err
}
```

- Sempre `WithContext` / cancelamento do request.
- Prepared statements (reutilização automática do driver) para hot paths.
- Bind por posição ou gocqlx `BindStruct` / `GetRelease` — evite concatenar CQL.
- `PageSize` + paging state para listas; não carregue partições inteiras na memória.

## Writes e idempotência

```go
err := session.Query(
    `INSERT INTO messages_by_room (room_id, sent_at, message_id, author_id, body)
     VALUES (?, ?, ?, ?, ?)`,
    roomID, sentAt, messageID, authorID, body,
).WithContext(ctx).Exec()
```

- Retries só com política clara; writes devem ser idempotentes ou usar IDs estáveis.
- LWT (`ScanCAS` / `MapScanCAS`) só quando o domínio exigir compare-and-set.
- Batch: prefira single-partition; batches multi-partition não são atômicos como “transação SQL”.
- TTL: `USING TTL ?` quando o modelo pedir expiração.

## Organização

```
internal/
  scylla/
    session.go      # cluster/session wiring
    messages.go     # repository por tabela/access pattern
  domain/
```

- Um arquivo/repo por access pattern ou agregado — nomes alinhados à table.
- Não exponha `*gocql.Session` aos handlers HTTP; exponha métodos de domínio.
- Erros do driver → erros de domínio (`not found`, timeout, unavailable).

## Observabilidade e erros

- Distinga timeout, unavailable, overloaded; mapeie para `503`/`504` na API quando couber.
- Métricas: latência p99 por query/table, timeouts, retries.
- Tracing do driver só em debug; não logue payloads sensíveis.
- Testes: Testcontainers/Scylla local ou cluster de CI; fakes do repositório nos unit tests (`golang-unit-testing`).

## Anti-padrões

- `CreateSession` por request ou por handler
- CQL montado com `fmt.Sprintf` e input de usuário
- `ALLOW FILTERING` escondido no repositório
- Ignorar `context` / paging e fazer full partition scan
- Consistency `All` como default “seguro”
- Tratar batch multi-partition como transação ACID
- Modelar no Go sem table query-first (ver `scylladb`)

## Fluxo ao implementar uma feature

1. Defina access pattern e table CQL (`scylladb`).
2. Adicione método no repository com statement preparado + context.
3. Escolha consistency/TTL/paging conscientes.
4. Trate erros e timeouts na borda HTTP/service.
5. Teste com Scylla real ou fake do port; meça p99 se for hot path.

## Critérios de conclusão

- Session singleton token-aware configurada
- Queries com partition key e context
- Paging/limites em listas
- Retries/idempotência pensados nos writes
- Repositório encapsula o driver
- Erros de cluster mapeados para o domínio/API
