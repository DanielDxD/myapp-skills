---
name: nuxt
description: Desenvolve apps Nuxt 4 com Vue 3, diretório app/, Nitro, SSR/híbrido, useFetch, routeRules e server routes. Use ao criar ou revisar projetos Nuxt, pages, layouts, middleware, server/api, modules ou quando o usuário pedir Nuxt 4 / Nuxt 3.
---

# Nuxt 4

Full-stack Vue com SSR por padrão. Código novo: **Nuxt 4**. Nuxt 3 chegou ao EOL (jul/2026) — preserve projetos 3.x e só migre se pedido. Não trate o app como SPA Vite: data fetching e secrets vivem no servidor.

SPA Vue sem SSR: skill `vue`.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **Nuxt 4** | Routing, SSR, modules, DX |
| **Vue 3.5+** | SFCs, Composition API |
| **Nitro** | `server/api`, middleware, deploy universal |
| **Vite** | Bundling do client |
| **TypeScript** | Tipos gerados (`.nuxt/`) |

Preserve modules e estrutura já adotados. `ssr: false` só com motivo explícito.

## Organização (Nuxt 4)

```
app/
  pages/
  components/
  composables/
  layouts/
  middleware/
  plugins/
  utils/
  app.vue
  app.config.ts
server/
  api/
  middleware/
  plugins/
  utils/
public/
nuxt.config.ts
```

- File-based routing em `app/pages/`.
- Auto-imports de `components/`, `composables/`, `utils/` — não reimporte o que o Nuxt já injeta.
- `server/` é Nitro: sem Vue, sem `window`.
- Em apps Nuxt 3 legados o root ainda pode ser `pages/` na raiz — não mova pastas sem pedido de migração.

## Regras inegociáveis

- Secrets, DB e tokens só em `server/` ou runtime config **private**.
- `useRuntimeConfig()`: `runtimeConfig` privado vs `runtimeConfig.public`.
- Authz no servidor (server routes, `server/middleware`, `getServerSession`/equiv.); middleware de rota é UX.
- Fetch SSR-aware: `useFetch` / `useAsyncData` / `$fetch` no server — não `onMounted` + `fetch` para conteúdo SEO-critical.
- Validar input em `server/api` (schemas). Contratos: skill `api-design`.
- Payload serializável entre server e client; nada de class instances / `Date` sem transformação consciente.

## Data fetching

```vue
<script setup lang="ts">
const route = useRoute()
const { data, error, pending, refresh } = await useFetch(`/api/items/${route.params.id}`, {
  key: `item-${route.params.id}`,
})
</script>
```

- `await useFetch` / `useAsyncData` no setup para bloquear a navegação até o HTML ter dados.
- `key` estável para dedupe e cache.
- Mutações: `$fetch` + `refresh`/`refreshCookie`/invalidação explícita — não confie em refetch ad hoc.
- Lazy: `lazy: true` ou `useLazyFetch` só quando o skeleton for intencional.
- Evite duplicar o mesmo fetch em layout + page; busque no ancestral comum ou use `useAsyncData` com a mesma `key`.

## Server (Nitro)

```ts
// server/api/items/[id].get.ts
export default defineEventHandler(async (event) => {
  const id = getRouterParam(event, 'id')
  const item = await loadItem(id)
  if (!item) {
    throw createError({ statusCode: 404, statusMessage: 'Not found' })
  }
  return item
})
```

- Convenção de arquivo: `[id].get.ts`, `index.post.ts`.
- `createError` com status HTTP corretos; não vaze stack em produção.
- Body/query: `readValidatedBody` / `getQuery` + schema.
- Cookies/headers via h3 (`getCookie`, `setCookie`, flags Secure/HttpOnly).

## Rendering e cache

- Default: SSR. Estático: `nuxt generate` ou `routeRules` com `prerender: true`.
- Híbrido via `routeRules` (`ssr`, `swr`, `isr`, `cache`, redirects) — decisões nomeadas, não globais cegas.
- Evite `ssr: false` no `nuxt.config` inteiro sem motivo.
- Islands / `<NuxtIsland>` quando um bloco for caro e isolável.

## Router, layouts, middleware

- Layouts em `app/layouts/`; `definePageMeta({ layout, middleware, alias })`.
- Route middleware (`app/middleware`) para redirects de auth no client; **revalide** no server.
- `navigateTo` para redirects; `defineNuxtRouteMiddleware`.
- Error pages: `error.vue`; loading: `loading`/skeletons conscientes.

## Config

```ts
export default defineNuxtConfig({
  compatibilityDate: '2026-01-01',
  runtimeConfig: {
    apiSecret: '',
    public: { apiBase: '' },
  },
  routeRules: {
    '/': { prerender: true },
    '/dashboard/**': { ssr: true },
  },
})
```

- Modules oficiais/maduros em vez de plugins manuais duplicados.
- Env: `NUXT_*` mapeado ao runtime config; nunca `import.meta.env.SECRET` no client.

## Qualidade

- `nuxi typecheck` / `vue-tsc`.
- Testes: Vitest em composables/utils; Playwright nos fluxos SSR (HTML inicial importa).
- A11y e performance: skills `better-web-interface` e `web-performance`.

## Anti-padrões

- `onMounted` + `fetch` para conteúdo indexável
- Expor `NUXT_SECRET` / `runtimeConfig` privado no bundle
- Tratar Nuxt como Vite SPA (router manual, Pinia para todo GET)
- Editar `.nuxt/` ou commitar artefatos gerados
- Migrar Nuxt 3 → 4 “de passagem” numa feature
- `useState` como cache global de API

## Integração com outras skills

- SPA Vue sem Nuxt: `vue`
- Contratos HTTP: `api-design`
- Authz: `authentication-authorization`
- Go-live: `production-readiness`

## Critérios de conclusão

- Estrutura `app/` + `server/` (ou legado Nuxt 3 preservado de propósito)
- Fetch SSR-aware com keys estáveis
- Secrets só no server / runtime private
- `routeRules` explícitas se o rendering for híbrido
- Typecheck ok; fluxo principal visível no HTML SSR ou documentado como client-only
