---
name: nextjs-architecture
description: Arquitetura Next.js App Router com limites server/client, rendering, caching, route handlers e organização por feature. Use ao criar ou refatorar apps Next.js, Server Components, Server Actions, middleware ou estratégias de cache.
---

# Next.js Architecture

Trate o App Router como o padrão. Só use Pages Router se o projeto já estiver nele.

## Regras inegociáveis

- Preferir Server Components por padrão; marcar `"use client"` só quando necessário.
- Não importar módulos de servidor em Client Components.
- Validar inputs na borda (route handlers, Server Actions) com schemas.
- Autorização sempre no servidor; nunca confiar só em UI.
- Caching e revalidação são decisões explícitas, não acidentes.

## Organização

```
app/                 # rotas, layouts, loading, error
features/<domínio>/  # UI, hooks, schemas, services do domínio
components/ui/       # primitivos reutilizáveis
lib/                 # auth, db, env, utils
```

- Coloque data fetching perto da rota que precisa dos dados.
- Extraia lógica de domínio para `features/`, não para “utils” genéricos.
- Mantenha layouts finos; lógica pesada fica em modules/services.

## Server vs Client

Use Client Components apenas para:

- estado interativo e event handlers
- browser APIs
- libs que exigem DOM

Mantenha no servidor:

- secrets, DB, auth
- aggregations e permissões
- SEO-critical content

Padrão: server fetch → props tipadas → client island mínima.

## Data e caching

- Escolha conscientemente entre static, dynamic e streaming.
- Use `revalidate` / `tags` / `updateTag` de forma nomeada e documentada.
- Evite `force-dynamic` global sem motivo.
- Deduplique fetches; não dispare a mesma query em cascata de componentes.
- Prefira Server Actions para mutações co-localizadas; route handlers para APIs públicas/webhooks.

## Route handlers e middleware

- Handlers: contratos claros, status HTTP corretos, erros tipados.
- Middleware: auth gate leve, redirects, headers; não faça DB pesado ali.
- Proteja rotas sensíveis no servidor mesmo com middleware.

## Anti-padrões

- `"use client"` no root layout sem necessidade
- Buscar dados no client só porque “é mais fácil”
- Passar objetos não serializáveis do server para o client
- Misturar lógica de negócio em JSX de página
- Cache stale sem invalidação após mutação

## Critérios de conclusão

- Limites server/client corretos e auditáveis
- Rotas organizadas por feature/domínio
- Mutações validadas e autorizadas no servidor
- Estratégia de cache/revalidate explícita
- Sem vazamento de secrets para o client bundle
