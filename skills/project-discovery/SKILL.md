---
name: project-discovery
description: Inspeciona stack, arquitetura, convenções e riscos antes de alterar um projeto. Use no início de qualquer tarefa em código desconhecido, monorepo, migração ou quando o usuário pedir exploração, onboarding ou mapa do sistema.
---

# Project Discovery

Antes de editar, mapeie o sistema. Não proponha mudanças estruturais sem evidência no repositório.

## Fluxo

1. Identifique o tipo de projeto: app web, API, mobile, monorepo ou biblioteca.
2. Localize entrypoints, configs e scripts de build/test/lint.
3. Extraia a stack real a partir de manifests e imports, não de suposições.
4. Mapeie pastas de domínio, camadas e limites (UI, API, dados, auth).
5. Liste padrões existentes: naming, state, erros, testes, CI.
6. Liste riscos: dívida técnica, arquivos críticos, áreas sem testes.
7. Só então proponha o menor plano alinhado às convenções encontradas.

## O que inspecionar

- Manifests: `package.json`, `pnpm-workspace.yaml`, `Cargo.toml`, `Podfile`, `Package.swift`, `.xcodeproj`
- Frameworks: Next.js, Vite, Expo, SwiftUI, Nest, Express, Prisma
- Rotas e entrypoints: `app/`, `pages/`, `src/main`, `App.tsx`, `ContentView`
- Dados: schemas Prisma/Drizzle, migrations, clients HTTP
- Auth e middlewares
- Testes e CI: `*.test.*`, Playwright, GitHub Actions
- Variáveis de ambiente e secrets (sem expor valores)

## Relatório mínimo

Entregue um resumo curto:

- Stack e versão aproximada
- Arquitetura em 3–6 bullets
- Convenções a respeitar
- Hotspots de risco
- Próximo passo recomendado

## Regras

- Prefira evidência de arquivos a memória de padrões genéricos.
- Se houver conflito entre docs e código, o código vence.
- Não refatore “de passagem” durante discovery.
- Preserve o estilo local; não imponha outra arquitetura sem pedido explícito.
- Em monorepos, declare qual pacote/app está no escopo.

## Critérios de conclusão

- Stack e entrypoints confirmados
- Convenções principais listadas
- Riscos e dependências críticas identificados
- Plano de mudança proporcional ao pedido
