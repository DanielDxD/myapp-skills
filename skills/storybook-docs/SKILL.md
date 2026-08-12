---
name: storybook-docs
description: Documentação viva com Storybook — CSF stories, Docs/MDX, controls, a11y, estados e autods. Use ao criar ou revisar stories, documentar design system, gerar docs de componentes ou padronizar Chromatic/Storybook no projeto.
---

# Storybook Docs

Storybook é a fonte de verdade visual dos componentes. Cada story demonstra um estado real; Docs explica o contrato — não um PDF paralelo.

Complementa `design-system` e `react-component-engineering`. Consistência visual nas apps: `better-web-interface`.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **Storybook 8+** | Runtime de stories (preserve a major do projeto) |
| **CSF 3** | `meta` + named exports |
| **Docs / Autodocs** | Documentação ao lado do canvas |
| **Controls / Args** | Props interativas |
| **Addons** | a11y, viewport, themes — os do projeto |

Não migre major nem troque Vite/Webpack sem pedido. Use o setup já no repo.

## Regras inegociáveis

- Um arquivo de stories por componente público do design system (ou por pasta documentada).
- Stories cobrem estados de risco: default, disabled, loading, error, empty, variantes.
- Args tipados alinhados às props reais — sem props fake só para o Storybook.
- Docs descreve quando usar / quando não usar, não só a API.
- Decorators de theme/router/i18n no preview — não duplicar boilerplate em cada story.
- Não documente componentes internos não exportados como API pública.

## CSF 3

```tsx
import type { Meta, StoryObj } from "@storybook/react";
import { Button } from "./Button";

const meta = {
  title: "Primitives/Button",
  component: Button,
  tags: ["autodocs"],
  args: { children: "Continue", variant: "primary" },
  argTypes: {
    variant: { control: "select", options: ["primary", "secondary", "ghost"] },
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story = {};

export const Secondary: Story = {
  args: { variant: "secondary" },
};

export const Disabled: Story = {
  args: { disabled: true },
};

export const Loading: Story = {
  args: { loading: true },
};
```

- `title`: hierarquia estável (`Primitives/`, `Patterns/`, `Features/`).
- Prefira `args` a JSX duplicado; use `render` só quando a composição exigir.
- `satisfies Meta<typeof Component>` para typecheck dos args.

## O que documentar em Docs

Para cada componente público:

1. **Propósito** — uma frase.
2. **Quando usar / evitar** — limites claros.
3. **Variantes e anatomia** — primary/secondary, slots.
4. **Acessibilidade** — teclado, roles, labels obrigatórias.
5. **Tokens** — quais variáveis de cor/espaço/tipo consome.
6. **Do / Don't** — 1–2 exemplos visuais quando o erro for comum.

MDX só quando autodocs + description na meta não bastarem (guias de marca, overview de foundations).

## Foundations no Storybook

Documente também (como stories ou MDX de overview):

- Cores semânticas / tipografia / spacing / radius / elevação
- Ícones
- Breakpoints e grid
- Tema light/dark (toolbar de theme)

Isso evita o design system existir “só no Figma”.

## Addons e qualidade

- **a11y**: rode no CI ou como gate local nas PRs de UI.
- **viewport**: stories críticas em mobile + desktop.
- **theme**: toggle light/dark se o produto tiver.
- Evite stories que dependem de rede real; mocke data na borda.
- Chromatic/visual regression se o projeto já usar — stories estáveis (sem tempo/random).

## Organização

```
src/components/Button/
  Button.tsx
  Button.stories.tsx
  Button.mdx          # só se necessário além de autodocs
```

- Coloque stories perto do código (`*.stories.tsx`) salvo convenção monorepo diferente.
- Design system package: stories no pacote UI; app pode ter stories de patterns compostos.

## Anti-padrões

- Uma story “KitchenSink” com todas as props misturadas
- Docs desatualizados com props que não existem
- Screenshots no README no lugar de stories
- Story que monta a página inteira da app sem necessidade
- Ignorar estados disabled/loading/error
- Args que não batem com a API TypeScript do componente

## Fluxo ao documentar um componente

1. Confirme API pública e variantes reais no código.
2. Meta + autodocs + args default.
3. Stories por estado/variante relevante.
4. Description / Do-Don't / a11y notes.
5. Verifique theme, viewport e addon a11y.
6. Se for foundation nova, linke na overview do design system.

## Critérios de conclusão

- Stories CSF tipadas para o componente tocado
- Estados principais cobertos e navegáveis por controls
- Autodocs ou MDX com propósito + limites de uso
- Theme/a11y/viewport alinhados ao preview do projeto
- Sem drift óbvio entre props TypeScript e docs
