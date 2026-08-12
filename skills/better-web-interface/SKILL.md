---
name: better-web-interface
description: Melhora design e consistência de interfaces web — tokens semânticos, tipografia, espaçamento, hierarquia, estados, responsividade e paridade light/dark. Use ao criar ou revisar UI web, refinar visual, unificar telas inconsistentes ou quando o usuário pedir interface moderna, polida ou coerente.
---

# Better Web Interface

Desenhe a web como um sistema, não como páginas soltas. Mesma hierarquia, ritmo e significado visual em todo o produto — desktop e mobile web.

Complementa `design-system` (tokens/componentes de produto), `react-component-engineering` (API de componentes) e `storybook-docs` (documentação viva). Mobile nativo: `better-mobile-interface`.

## Resultado esperado

- Hierarquia óbvia em poucos segundos; uma ação primária clara por view.
- Espaçamento, tipo, raio e cor consistentes entre rotas e breakpoints.
- Light/dark (se o produto tiver) com os mesmos roles semânticos.
- Responsivo sem “versão mobile” inventada com outros padrões.
- Acessível: contraste, foco, teclado, `prefers-reduced-motion`.

## Antes de desenhar

1. **Produto e público** — o que a tela resolve.
2. **Assinatura visual** — um detalhe memorável (tipo, acento, mídia, shape).
3. **Tom** — calmo / energético / editorial / utilitário.
4. **Inventário** — o que já existe no design system / CSS variables / componentes.

Se o projeto já tem design system ou brand guidelines, eles vencem. Complete só o que faltar. Não invente uma segunda linguagem visual na feature.

## Tokens semânticos

Nunca espalhe hex literais nas telas. Use roles (CSS variables, theme object, Tailwind tokens semânticos):

| Role | Função |
|------|--------|
| `--bg` / `background` | Fundo da página |
| `--surface` | Cards, painéis, inputs |
| `--surface-elevated` | Popovers, modais, menus |
| `--text` / `text-primary` | Título e corpo |
| `--text-muted` | Apoio, metadata |
| `--border` | Divisores e outlines |
| `--accent` / `primary` | CTA e foco de marca |
| `--on-accent` | Texto/ícone sobre acento |
| `--success` / `--warning` / `--danger` | Feedback |
| `--focus-ring` | Foco visível |

Regras:

- Um acento (ou família controlada). Não uma cor nova por página.
- Neutros carregam a UI; acento em CTAs, links e seleção.
- Contraste AA: 4.5:1 corpo, 3:1 texto grande / ícones essenciais.
- Dark: superfícies por camada + borda sutil; evite `#000` + cinza ilegível.

## Tipografia

- Escala nomeada limitada: `display`, `title`, `body`, `label`, `caption`.
- No máximo 2 famílias (display + body) ou a stack já do produto.
- Hierarquia por tamanho **e** peso — não só por cor.
- Line-height confortável; evite paredes de texto sem measure (~45–75ch).
- Não misture 6 tamanhos “quase iguais” na mesma view.

## Espaço e layout

- Base 4 ou 8px. Escala: 4, 8, 12, 16, 24, 32, 40, 48, 64…
- Largura de conteúdo e gutters consistentes por breakpoint.
- Um ritmo vertical por página; agrupe relacionados; separe seções com espaço, não com bordas demais.
- Grid/flex alinhados ao sistema; evite magias de `margin` únicas por tela.
- Densidade: app utilitário mais compacto; marketing mais aéreo — escolha e mantenha.

## Forma e profundidade

- Raios nomeados (`sm` / `md` / `lg` / `full`) reutilizados em botões, inputs e cards.
- Elevação só quando comunica stacking (modal > popover > page).
- Evite multi-shadow, glow e glass em tudo.
- Separadores na cor `border`, 1px.

## Consistência entre interfaces

Ao tocar uma tela, alinhe com as irmãs:

| Elemento | Regra |
|----------|--------|
| CTAs | Mesmo estilo primário/secundário/terciário e ordem |
| Forms | Mesmo label, helper, erro, altura de controle |
| Nav | Mesmos itens, ícones e estado ativo |
| Tabelas/listas | Mesma hierarquia título → meta → ação |
| Empty/loading/error | Mesmos padrões de feedback |
| Modais/drawers | Mesmo header, dismiss, foco inicial |

Checklist rápido de inconsistência:

- [ ] Botões com raios/padding diferentes entre páginas
- [ ] Cinzas e acentos “parecidos” mas não iguais
- [ ] Espaçamento 13/18/22 sem token
- [ ] Ícones de famílias misturadas
- [ ] Dark/light só em parte do fluxo

## Responsivo

- Mobile-first ou breakpoints do projeto — sem layout paralelo com outra marca visual.
- Touch targets ≥ 44px onde houver ponteiro grosso.
- Tabelas densas: scroll/cardização consciente, não esmagar.
- Tipografia e spacing podem escalar; roles semânticos não mudam de significado.

## Estados e feedback

- Toda superfície interativa: default, hover, focus-visible, active, disabled.
- Forms: erro junto ao campo + mensagem acionável.
- Empty / skeleton / error são design, não tela em branco.
- Toasts/banners: severidade pelos tokens `success|warning|danger`.

## Movimento

- 150–300ms na maioria das transições de UI.
- Motion reforça hierarquia e feedback — não decora.
- Respeite `prefers-reduced-motion`.
- Evite parallax/bounce teatral em app productivo.

## Anti-padrões (UI “IA genérica”)

- Roxo/indigo gradient + glassmorphism em tudo
- Cream + serif + terracota como default sem brief
- Excesso de pills, badges, stat strips e cards aninhados
- Hero lotado de metadata e CTAs competindo
- Tipografia só Inter/Roboto sem escala pensada + acento genérico
- Consistência sacrificada por “cada página única”

## Fluxo de implementação

1. Ler tokens/componentes existentes (`design-system`) — reutilizar antes de criar.
2. Aplicar só roles semânticos e componentes do sistema na view.
3. Unificar CTAs, forms e espaçamento com telas vizinhas.
4. Revisar breakpoints + light/dark (se houver).
5. Documentar variantes novas no Storybook quando forem reutilizáveis (`storybook-docs`).

## Critérios de conclusão

- Sem cores/spacing literais soltos nas áreas tocadas
- Hierarquia e CTA primário claros
- Consistente com telas irmãs e com o design system
- Estados (hover/focus/disabled/empty/error) cobertos
- Responsivo e acessível no essencial (contraste, foco, teclado)
- Sem regressão de tom visual do produto
