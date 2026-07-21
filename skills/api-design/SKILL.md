---
name: api-design
description: Design de APIs HTTP com contratos, schemas, erros, paginação, idempotência, autorização e versionamento. Use ao criar ou revisar endpoints REST/JSON, route handlers, BFF ou integrações entre serviços.
---

# API Design

Contratos claros e estáveis vencem “endpoints ad hoc”. Toda mutação passa por validação e autorização.

## Contratos

- Recursos e verbos previsíveis: `GET/POST/PATCH/DELETE`.
- Nomes consistentes: plural de recursos, IDs opacos, query params documentados.
- Request/response com schema (Zod etc.) e tipos TypeScript derivados.
- Não vaze campos internos (password hash, tokens, PII desnecessária).
- Versionamento: path (`/v1`) ou header quando a API for pública/consumida por clientes externos.

## Erros

- Use status HTTP corretos (`400`, `401`, `403`, `404`, `409`, `422`, `429`, `500`).
- Corpo de erro estável: `{ code, message, details? }`.
- `code` machine-readable; `message` segura para humanos.
- Não exponha stack traces em produção.
- Distinga validação (cliente) de falha interna (servidor).

## Paginação e filtros

- Prefira cursor-based para listas grandes; offset só quando simples e limitado.
- Limite page size; rejeite valores abusivos.
- Ordene de forma determinística.
- Documente filtros suportados; ignore silenciosamente o mínimo possível.

## Idempotência e mutações

- `PUT`/`DELETE` idempotentes quando fizer sentido.
- Para `POST` críticos (pagamentos, jobs), aceite `Idempotency-Key`.
- Transações para writes compostos.
- Retorne o recurso atualizado ou `202` + location para async.

## Autorização

- Authn ≠ Authz. Autenticado não significa autorizado.
- Verifique ownership/roles no servidor em toda rota sensível.
- Rate limit em auth e endpoints caros.
- CORS e cookies: configuração explícita; Least privilege.

## Observabilidade

- Logue request id, user id (se houver), rota, status, latência.
- Não logue secrets nem payloads sensíveis completos.
- Métricas de erro e latência por endpoint.

## Anti-padrões

- Um endpoint “faz tudo”
- 200 com erro no body sem convenção
- Quebrar clientes sem versionamento
- Validação só no frontend
- N+1 escondido atrás de um GET “simples”

## Critérios de conclusão

- Schemas de entrada/saída definidos
- Erros e status consistentes
- Authz coberta nas rotas sensíveis
- Paginação/limites seguros
- Contrato utilizável por pelo menos um cliente real do repo
