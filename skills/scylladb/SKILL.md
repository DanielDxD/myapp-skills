---
name: scylladb
description: Modelagem e operações com ScyllaDB — partition/clustering keys, CQL, consistência, TTLs, denormalização e anti-padrões de leitura. Use ao projetar keyspaces/tables, revisar queries CQL, definir consistency levels ou migrar workloads Cassandra-compatíveis.
---

# ScyllaDB

Modele para as queries. Partition key define distribuição e hotspots; clustering define ordem na partição. ScyllaDB não é um SQL relacional com joins.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **CQL** | DDL/DML |
| **Keyspace** | RF, estratégia de replicação |
| **Table** | Partition key + clustering + payload |
| **Materialized views / tabelas extras** | Access patterns alternativos |

Integração em Go: use `golang-scylladb`. Esta skill cobre modelo de dados e operações independentes da linguagem.

## Regras inegociáveis

- Desenhe a table a partir do access pattern (query-first), não do diagrama ER.
- Toda query de produção deve restringir a **partition key** (e clustering de forma prefixada).
- Evite `ALLOW FILTERING` em paths quentes.
- Denormalize e duplique writes quando precisar de múltiplos padrões de leitura.
- Consistency level consciente: trade-off latência vs. durabilidade/visibilidade.
- TTLs e deletes geram tombstones — planeje compaction e tamanho de partição.

## Modelagem

```cql
CREATE KEYSPACE IF NOT EXISTS app
WITH replication = {'class': 'NetworkTopologyStrategy', 'dc1': 3};

CREATE TABLE app.messages_by_room (
  room_id   uuid,
  sent_at   timeuuid,
  message_id uuid,
  author_id uuid,
  body      text,
  PRIMARY KEY ((room_id), sent_at, message_id)
) WITH CLUSTERING ORDER BY (sent_at DESC, message_id ASC);
```

- Partition key `((room_id))`: colocaliza dados lidos juntos.
- Clustering (`sent_at`, `message_id`): ordenação e paginação na partição.
- Um access pattern novo → tabela nova (ou MV só se o trade-off for aceito).
- IDs: `uuid` / `timeuuid` conforme necessidade de ordenação temporal.

## Partition size e hotspots

- Partições grandes demais degradam latência e compaction; quebre por bucket temporal (`day`, `hour`) quando a partição crescer sem bound.
- Hot partition: uma key recebendo a maior parte do tráfego — redistribua (bucketing, sharding lógico).
- Prefira muitas partições moderadas a poucas gigantes.

## Consistência e writes

| Nível típico | Uso |
|--------------|-----|
| `LOCAL_QUORUM` | Default comum em multi-node DC único / NTS |
| `LOCAL_ONE` | Leitura/escrita mais rápida, menos garantia |
| `EACH_QUORUM` / `ALL` | Raro; latência e disponibilidade piores |

- Lightweight transactions (`IF NOT EXISTS` / conditional updates) só quando a correção exigir — custo alto.
- Idempotência: desenhe writes reaplicáveis quando houver retries.
- Contadores: tipo especial com limitações; evite se um modelo append/rollup servir melhor.

## Leituras e paginação

- `SELECT` com equality na partition key; ranges só em clustering columns em ordem de prefixo.
- Paginação com paging state do driver / `LIMIT` consciente.
- Multi-partition fan-out no client: limite concorrência e timeout.
- Secondary indexes: use com parcimônia; muitas vezes tabela denormalizada é melhor.

## TTL, deletes e tombstones

- TTL em colunas/linhas para expiração automática.
- Deletes frequentes + leitura wide = risco de tombstone storms.
- Prefira modelos append + TTL a update/delete intensivos quando o domínio permitir.
- Monitore tombstones e latência de read no Ops/Monitoring do cluster.

## Operações (app-facing)

- Schema changes: compatíveis e forward-safe; evite rewrites destrutivos sem plano.
- RF e DC alinhados ao `NetworkTopologyStrategy` real do deploy.
- Timeouts e retries no client alinhados ao SLA; não retry infinito em writes não idempotentes.
- Capacidade: meça p99 sob o access pattern real, não só throughput sintético.

## Anti-padrões

- Modelar como 3NF e “descobrir joins” no app
- Queries sem partition key / `ALLOW FILTERING` no hot path
- Uma única partição por tenant gigante sem bucketing
- Secondary index para alta cardinalidade / alta seletividade errada
- LWT em todo write “por precaução”
- Tratar Scylla como fila sem controlo de tombstones/TTL

## Critérios de conclusão

- Cada query de produção casa com uma table/access pattern
- Partition e clustering keys justificadas
- Consistency levels documentados por operação
- Partições com bound razoável (ou bucketing)
- Tombstones/TTL considerados no desenho
- Sem `ALLOW FILTERING` nos caminhos críticos
