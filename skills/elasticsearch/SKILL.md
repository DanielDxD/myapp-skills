---
name: elasticsearch
description: Indexação e busca com Elasticsearch — mappings, queries DSL, aggregations, paginação, bulk e performance. Use ao modelar índices, escrever buscas, depurar relevance/latência ou integrar Elasticsearch em APIs.
---

# Elasticsearch

Modele o índice para as queries. Mapping é contrato; `text` vs `keyword` e analyzers definem o que a busca consegue fazer bem.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **Index / mapping** | Schema de campos, analyzers, norms |
| **Query DSL** | `bool`, `match`, `term`, `range`, filters |
| **Aggregations** | Facets, métricas, buckets |
| **Bulk API** | Ingestão em lote |
| **Client** | Oficial da linguagem do projeto (JS/Java/Go/Python/etc.) |

OpenSearch compartilha grande parte do DSL — preserve o produto já adotado. Não misture APIs de versões incompatíveis sem checar.

## Regras inegociáveis

- Defina mapping explícito para campos críticos; não dependa só de dynamic mapping em produção.
- Filters (`filter`/`must_not` em `bool`) para critérios exatos — cacheáveis e sem score.
- Queries full-text em `text`; IDs, status, tags em `keyword` (ou `.keyword`).
- Paginação profunda com `search_after` / point-in-time; evite `from` alto em índices grandes.
- Bulk com tamanho consciente; retries idempotentes quando o domínio permitir.
- Nunca exponha o cluster à internet sem auth; queries vindas do client devem ser construídas no servidor.

## Mapping

```json
{
  "mappings": {
    "properties": {
      "title": { "type": "text", "fields": { "raw": { "type": "keyword" } } },
      "status": { "type": "keyword" },
      "created_at": { "type": "date" },
      "tenant_id": { "type": "keyword" }
    }
  }
}
```

- Multi-fields (`text` + `keyword`) quando ordenar/agregar e buscar full-text no mesmo campo.
- Nested / `join` só com necessidade real — custo de query e memória.
- Aliases para blue/green de reindex; apps apontam ao alias, não ao nome cru versionado.

## Query DSL

```json
{
  "query": {
    "bool": {
      "must": [{ "match": { "title": "rust async" } }],
      "filter": [
        { "term": { "tenant_id": "org_123" } },
        { "term": { "status": "published" } }
      ]
    }
  },
  "sort": [{ "created_at": "desc" }, { "_id": "asc" }],
  "size": 20
}
```

- Multi-tenant: sempre filtrar `tenant_id` (ou equivalente) no servidor.
- Prefira `bool` composto a scripts ad hoc no hot path.
- `minimum_should_match`, fuzziness e boosts só com critério de relevance medido.
- Source filtering (`_source`) — não traga campos enormes sem necessidade.

## Ingestão e updates

- Bulk: batches (ex. alguns MB / N docs), flush periódico, trate `items[].error`.
- Updates parciais vs reindex: documentos imutáveis + versão nova quando o modelo permitir.
- Pipelines de ingest para normalize/enrich na borda do cluster.
- Reindex + alias swap para mudanças de mapping breaking.

## Aggregations e analytics

- Aggs em campos `keyword` / numéricos / date com calendário explícito.
- Limite `size` de terms aggs; use `composite` para paginar buckets grandes.
- Não use aggs como substituto de warehouse sem medir custo de heap/CPU.

## Performance e ops (app-facing)

- Timeouts e `track_total_hits` conscientes (evitar `true` sempre em listas enormes).
- Circuit breakers e rejeição: trate `429`/`503` com backoff.
- Monitore p99 de search/index, refresh lag, rejected bulk.
- Shards: comece simples; oversharding piora latência — ajuste com evidência.

## Anti-padrões

- Dynamic mapping livre em produção sem revisão
- `query_string` cru com input do usuário
- `from`/`size` para deep pagination
- Wildcard líder (`*foo`) em hot path
- Um índice “faz tudo” sem aliases/estratégia de rollover quando o volume exige
- Ignorar isolamento multi-tenant na query

## Critérios de conclusão

- Mapping alinhado aos access patterns
- Queries com filter de tenancy/authz
- Paginação segura para o volume esperado
- Bulk/erros de ingestão tratados
- Relevance/latência aceitáveis nos caminhos críticos
- Cliente e credenciais só no servidor
