---
name: design-system
description: Cria e evolui design systems — foundations (tokens), componentes, padrões, versionamento, acessibilidade e governança. Use ao estruturar UI kit, definir tokens, padronizar componentes, alinhar Figma/código ou escalar consistência visual no produto.
---

# Design System

O design system é o contrato visual e de interação do produto. Tokens e componentes vencem one-offs de tela. Código e Figma (se houver) devem contar a mesma história.

Consumo visual em apps: `better-web-interface`. API de componentes React: `react-component-engineering`. Documentação viva: `storybook-docs`. Mobile nativo: `better-mobile-interface`.

## Camadas

```
Foundations (tokens) → Primitives → Components → Patterns → Product screens
```

| Camada | Exemplos | Regra |
|--------|----------|--------|
| **Foundations** | cor, tipo, espaço, raio, sombra, z-index, motion | Sem UI; só tokens |
| **Primitives** | Button, Input, Checkbox, Icon | API mínima, altamente reutilizável |
| **Components** | Field, Modal, DataTable | Compõem primitives |
| **Patterns** | LoginForm, PageHeader, EmptyState | Receitas de produto, ainda reutilizáveis |

Não pule camadas: tela de produto não redefine `padding: 13px` se o token é `space.16`.

## Foundations (tokens)

Defina roles semânticos, não só paletas brutas:

- **Color**: `background`, `surface`, `text`, `border`, `accent`, feedback…
- **Typography**: famílias, sizes, weights, line-heights nomeados
- **Space**: escala 4/8
- **Radius / border / elevation**
- **Motion**: duration/easing
- **Breakpoints**

Formato: CSS variables, Style Dictionary, theme JS/TS ou tokens do toolchain do projeto — preserve o que já existe.

```css
:root {
  --color-bg: #f7f7f5;
  --color-surface: #ffffff;
  --color-text: #121212;
  --color-accent: #0b6e4f;
  --space-4: 1rem;
  --radius-md: 8px;
}
```

- Temas (light/dark / brand): trocam valores, **não** os nomes dos roles.
- Documente aliases (`accent` → CTA) para design e eng usarem a mesma língua.

## Componentes

- Um trabalho por componente; variantes via props discriminadas ou composição.
- Estados obrigatórios: default, hover, focus-visible, disabled; + loading/error quando couber.
- Acessibilidade de fábrica: teclado, foco, labels, contraste.
- Asleeping API: prefira `variant` + `size` a dezenas de booleans.
- Estilos só via tokens do system; proibido hex solto no primitive.

```tsx
type ButtonProps = {
  variant?: "primary" | "secondary" | "ghost" | "danger";
  size?: "sm" | "md" | "lg";
  loading?: boolean;
  disabled?: boolean;
  children: React.ReactNode;
};
```

## Patterns

- Page headers, forms de settings, empty states, confirm dialogues — documente como pattern se se repetirem 3+ vezes.
- Patterns podem opinar em copy estrutural (“Título + descrição + CTA”), não em copy de feature específica.
- Se um pattern só serve a uma tela, ele ainda não é system — deixe no app.

## Governança

- **Source of truth**: pacote UI / pasta `packages/ui` (ou equivalente) + Storybook.
- Mudança breaking de token/componente → versionamento semver + changelog.
- Processo: propor → documentar story → adotar em 1–2 telas → expandir.
- Deprecar com caminho de migração; não apagar silentemente.
- Figma (se houver): mesmos nomes de tokens/componentes; drift é bug.

## Adoção no produto

1. Inventariar one-offs (botões/inputs/cores duplicados).
2. Extrair primitive + tokens.
3. Substituir call sites sem mudar UX sem intenção.
4. Story + docs (`storybook-docs`).
5. Proibir novos literais via lint/review quando o time estiver pronto.

Apps devem importar do design system — não copiar o Button para `features/`.

## Acessibilidade e i18n

- Contraste e foco nos tokens/componentes, não “depois na tela”.
- Não embutir strings de produto em primitives; patterns podem ter defaults sobrescrevíveis.
- Suporte a densidades / zoom / RTL se o produto exigir — planeje no primitive.

## Anti-padrões

- Design system = só biblioteca de botões sem tokens
- Tokens semânticos misturados com `blue-500` nas telas
- Variantes infinitas para um caso de uma feature
- Documentação só no Figma, código divergente
- Quebrar APIs sem major/changelog
- Reestilizar primitive com CSS global frágil por página

## Fluxo ao evoluir o system

1. Qual camada? (token vs primitive vs pattern)
2. Existe algo reutilizável? Estenda antes de criar.
3. Implemente com tokens + a11y + testes leves se o repo tiver.
4. Stories/docs com do/don't.
5. Migre call sites críticos; anuncie deprecações.

## Critérios de conclusão

- Tokens semânticos cobrindo a mudança
- Componente/pattern na camada certa, sem one-off disfarçado
- Estados e a11y básicos cobertos
- Documentado no Storybook (ou docs do monorepo)
- Apps consumidores usam o system sem hex/spacing soltos novos
- Breaking changes versionados ou evitados
