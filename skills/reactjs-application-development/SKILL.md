---
name: reactjs-application-development
description: Desenvolve aplicações React (SPA) com Vite, React Router, estado local/remoto, formulários, acessibilidade e performance, sem pressupor Next.js. Use ao criar ou evoluir SPAs React, dashboards, painéis Vite/CRA ou apps client-side.
---

# ReactJS Application Development

Para apps React completas fora do Next.js. Componentes reutilizáveis detalhados ficam na skill `react-component-engineering`.

## Stack típica

- Bundler: Vite (preferência) ou o do projeto
- Roteamento: React Router
- Dados remotos: TanStack Query (ou equivalente já adotado)
- Forms: React Hook Form + schema Zod
- Estilo: o sistema do projeto (Tailwind, CSS Modules, etc.)

Não imponha libs se o repo já tiver padrão diferente.

## Arquitetura

```
src/
  app/          # providers, router, shell
  routes/       # páginas por rota
  features/     # domínio: api, hooks, ui, model
  shared/       # ui kit, lib, hooks genéricos
```

- Rotas preguiçosas (`lazy` + `Suspense`) para features pesadas.
- Providers na borda: query client, theme, auth.
- Erros de rota com error boundaries.

## Estado

- **Local**: UI efêmera.
- **Remoto**: cache do server state (Query), não duplique em Redux sem necessidade.
- **Global de sessão**: auth/user mínimo.
- URL como estado para filtros/paginação compartilháveis.

## Dados e forms

- Camada API tipada + schemas na borda.
- Mutations com invalidação de cache explícita.
- Forms: validação, erros por campo, estados pending/disabled.
- Tratamento de 401/403 centralizado (redirect ou evento).

## Roteamento e auth

- Route guards no loader/wrapper, não só esconder botões.
- Layouts aninhados para shell autenticado vs público.
- Deep links e fallback 404.

## Qualidade

- A11y: landmarks, foco, labels, contraste.
- Performance: code-split, listas virtuais se grandes, memo com evidência.
- Testes: Testing Library nas features; Playwright nos fluxos críticos.

## Anti-padrões

- Buscar tudo em `useEffect` ad-hoc sem cache
- Prop drilling extremo sem composição/context pontual
- Store global para server state
- Misturar lógica de domínio em componentes de página gigantes
- Assumir Next.js APIs (`getServerSideProps`, RSC) nesta skill

## Critérios de conclusão

- Rotas e features organizadas
- Server state e forms previsíveis
- Auth guard no client alinhado à API
- Build de produção ok
- Fluxo principal testado ou verificado manualmente com checklist
