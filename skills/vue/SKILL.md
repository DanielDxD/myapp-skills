---
name: vue
description: Desenvolve SPAs Vue 3.5+ com Composition API, script setup, Vite, Vue Router, Pinia, composables e TypeScript. Use ao criar ou revisar apps Vue (sem Nuxt), componentes .vue, rotas, stores ou quando o usuário pedir Vue 3, Composition API ou Vite+Vue.
---

# Vue 3

SPA Vue **sem** pressupor Nuxt. Código novo: Vue 3.5+, Composition API e `<script setup lang="ts">`. Vue 2 chegou ao EOL — não escreva Options API nova salvo o projeto já viver nela.

## Stack típica

| Peça | Papel |
|------|--------|
| **Vue 3.5+** | UI reativa, SFC |
| **Vite** | Dev server e build |
| **Vue Router 4** | Rotas, guards, layouts |
| **Pinia** | Estado de sessão/UI compartilhado |
| **TypeScript** | Props, emits e composables tipados |

Dados remotos: TanStack Vue Query (ou o fetch já adotado). Não duplique server state no Pinia.

Não imponha libs se o repo já tiver padrão diferente. SSR/híbrido: skill `nuxt`.

## Organização

```
src/
  main.ts              # app, plugins, mount
  App.vue
  router/
  stores/              # Pinia por domínio
  views/               # páginas de rota
  components/
  composables/
  lib/                 # api client, schemas
  assets/
```

- Páginas magras; regra de negócio em composables/services.
- Composables `useXxx` com ciclo de vida explícito (cleanup em `onUnmounted`).
- Auto-import de Vite só se o projeto já usar (`unplugin-auto-import`); senão importe.

## Regras inegociáveis

- `<script setup lang="ts">` em componentes novos.
- Props/emits tipados (`defineProps`, `defineEmits`); `defineModel` para v-model.
- Refs de template: `useTemplateRef` (não `$refs` / `ref="x"` solto sem tipo).
- IDs de a11y estáveis: `useId()`.
- Destructuring de props é reativo no 3.5+; não copie props para `ref` local sem motivo.
- Authz de verdade está no servidor; guards de rota só melhoram UX.
- Vue 3.6 Vapor Mode é opt-in futuro — não exija nem migre o renderer.

## Componentes

```vue
<script setup lang="ts">
import { useId, useTemplateRef } from 'vue'

defineProps<{
  title: string
  disabled?: boolean
}>()

defineEmits<{
  select: [id: string]
}>()

const model = defineModel<string>({ required: true })
const inputRef = useTemplateRef('input')
const headingId = useId()
</script>
```

- Um componente, um trabalho. Slots nomeados em vez de props booleanas explosivas.
- `v-bind="$attrs"` consciente; declare `inheritAttrs` quando wrapping nativo.
- Evite `watch` para derivar estado — use `computed`.
- `shallowRef` / `triggerRef` para objetos grandes que não precisam de deep reactivity.

## Estado e dados

- **Local:** UI efêmera (`ref` no componente).
- **Remoto:** cache de Query/`useFetch` próprio — não clones JSON no Pinia.
- **Sessão:** Pinia mínimo (user, prefs).
- URL como estado para filtros/paginação compartilháveis (`route.query`).

```ts
export const useAuthStore = defineStore('auth', () => {
  const user = ref<User | null>(null)
  const isSignedIn = computed(() => user.value !== null)
  function setUser(next: User | null) {
    user.value = next
  }
  return { user, isSignedIn, setUser }
})
```

Prefira setup stores (`defineStore(() => { ... })`) a options stores.

## Router

- Rotas com `component: () => import(...)` para code-split.
- Layouts via nested routes (`children` + `<RouterView>`), não wrappers ad hoc em cada view.
- Guards: `beforeEnter` / `router.beforeEach` para auth de UX; 404 com rota catch-all.
- Nomes de rota estáveis; links com `RouterLink`, não `window.location`.

## Qualidade

- A11y: landmarks, labels, foco, `aria-*` só quando o nativo não chega.
- Performance: `v-once`/`v-memo` com evidência; listas grandes com virtualização.
- Testes: Vue Test Utils / Testing Library nas features; Playwright nos fluxos críticos.
- `vue-tsc` + ESLint (`eslint-plugin-vue`) no ciclo.

## Anti-padrões

- Options API em arquivo novo num app Composition
- Assumir Nuxt (`useFetch` auto-imported, `navigateTo`, `server/api`)
- Buscar tudo em `onMounted` sem cache
- Pinia para cada response HTTP
- Mutar props; `v-if` + `v-for` no mesmo elemento
- `reactive()` em primitivos; `ref` + `.value` esquecido em helpers

## Integração com outras skills

- Full-stack SSR: `nuxt`
- UI web genérica: `better-web-interface`
- Tipagem: `typescript-strict`
- Testes e2e: `testing-fullstack`

## Critérios de conclusão

- SFCs em `<script setup>` tipado
- Rotas code-split e guards alinhados à API
- Server state fora do Pinia
- Build Vite de produção ok
- Fluxo principal testado ou verificado com checklist a11y
