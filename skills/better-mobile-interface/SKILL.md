---
name: better-mobile-interface
description: Define e aplica regras de estilo para interfaces mobile modernas, bonitas e consistentes em Android (Jetpack Compose) e iOS (SwiftUI), com paridade light/dark, tokens semânticos, tipografia, espaçamento, movimento e hierarquia visual. Use ao criar ou revisar UI mobile, design systems, theming, dark mode ou quando o usuário pedir interface moderna, polida ou consistente entre plataformas.
---

# Better Mobile Interface

Desenhe telas mobile como um sistema, não como telas soltas. A mesma hierarquia, ritmo e significado visual devem funcionar em Compose e SwiftUI — adaptando componentes nativos, sem clonar pixels cegamente.

Implementação de framework: `jetpack-compose` e `swiftui-development`. Arquitetura: `android-development` / `mvvm-architecture`. Para React Native/Expo: `better-mobile-interface-react-native`.

## Resultado esperado

- Interface contemporânea, legível e com hierarquia óbvia em 3 segundos.
- Light e dark com a mesma estrutura semântica (não só “inverter cores”).
- Espaçamento, tipo e raio consistentes em todo o fluxo.
- Parece nativo em cada OS, mas pertencente ao mesmo produto.
- Acessível: contraste, Dynamic Type / fontScale, alvos de toque.

## Antes de desenhar

Defina em poucas linhas:

1. **Produto e público** — o que a tela vende ou resolve.
2. **Uma assinatura visual** — o detalhe memorável (tipo, cor de acento, tratamento de mídia, shape).
3. **Tom** — calmo / energético / editorial / utilitário.
4. **Tokens** — cores semânticas, tipo, espaço, raio (light + dark juntos).

Se o app já tem design system ou brand guidelines, eles vencem. Complete só o que faltar.

## Tokens semânticos (obrigatório)

Nunca espalhe hex/Color literais nas telas. Sempre roles:

| Role | Função |
|------|--------|
| `background` | Fundo da tela |
| `surface` | Cards, sheets, inputs |
| `surfaceElevated` | Modais, menus, FABs |
| `textPrimary` | Título e corpo principal |
| `textSecondary` | Apoio, metadata |
| `textTertiary` | Hints, placeholders |
| `border` / `separator` | Divisores e outlines |
| `accent` / `primary` | CTA e foco de marca |
| `onAccent` | Texto/ícone sobre acento |
| `success` / `warning` / `danger` | Feedback |
| `overlay` | Scrim de modal |

Regras:

- Light e dark definem **pares** para cada role na mesma tabela de tokens.
- Acento único (ou família controlada). Não invente uma cor nova por tela.
- Neutros carregam a UI; o acento aparece em CTAs, seleção e links.
- Elevação no dark: prefira superfícies mais claras por camada, não só sombra preta invisível.
- Contraste texto/fundo ≥ WCAG AA (4.5:1 corpo, 3:1 texto grande/ícones essenciais).

### Compose

- Estenda `MaterialTheme` / `ColorScheme` (Material 3) ou um `AppTheme` próprio com composition locals.
- `isSystemInDarkTheme()` + override de usuário se o produto tiver.
- Dynamic Color só se a marca permitir; senão, palette fixa do produto.

### SwiftUI

- `Color` assets com variantes Any/Dark, ou `Color` semânticos no asset catalog.
- `preferredColorScheme` só quando a tela exigir; o padrão segue o sistema.
- Prefira estilos semânticos (`.primary`, `.secondary`) mapeados aos tokens do app.

## Tipografia

- Escala limitada e nomeada: `display`, `title`, `body`, `label`, `caption`.
- No máximo 2 famílias (ou a stack do sistema: SF / Roboto / brand font).
- Hierarquia por tamanho **e** peso — não só por cor.
- Tracking e line-height confortáveis em mobile; títulos sem truncar sem estratégia (`minScale` / `lineLimit` conscientes).
- Respeite Dynamic Type (iOS) e fontScale (Android); layouts não podem quebrar em 200%.
- Evite caixa alta longa; labels curtos ok.

## Espaço, grade e densidade

- Base 4 ou 8 pt/dp. Escala: 4, 8, 12, 16, 20, 24, 32, 40, 48…
- Margens de tela consistentes (tipicamente 16–20).
- Uma tela: um ritmo vertical — não alterne 10, 14, 22 sem sistema.
- Agrupe o que é relacionado; separe seções com espaço, não com linhas demais.
- Densidade: utilitários podem ser mais compactos; marketing/onboarding mais aéreos — escolha e mantenha.

## Forma, elevação e profundidade

- Raios nomeados: `sm`, `md`, `lg`, `full` — use os mesmos em cards, inputs e chips.
- Bordas sutis; no dark, borda + surface elevada costuma ler melhor que sombra forte.
- Evite multi-shadow “glow” genérico e cards com blur em tudo.
- Separadores: 1 pt/dp na cor `separator`, nunca preto 100%.

## Layout mobile

- Safe areas / system bars / home indicator / cutouts respeitados.
- Alvos de toque ≥ 44–48 pt/dp.
- Primária da tela: uma ação principal óbvia (FAB/botão preenchido).
- Listas: hierarquia título → subtítulo → meta; trailing actions discretas.
- Empty / loading / error são estados de design, não telas em branco.
- Teclado: evitar campos escondidos; scroll insets corretos.
- Não copie chrome de plataforma cruzada (bottom nav iOS idêntica ao Material sem adaptação).

## Light e dark — paridade

Checklist por tela:

- [ ] Mesmos tokens semânticos; nenhum hex só no light ou só no dark
- [ ] Imagens/ícones legíveis nos dois (templates, tint, versões adaptivas)
- [ ] Elevação/superfície distinguível no dark
- [ ] Acento com contraste em ambos os fundos
- [ ] Dividers e borders visíveis sem “sumir” no dark
- [ ] Status bar / nav bar ícones claros ou escuros conforme o fundo
- [ ] Placeholders e skeletons com contraste adequado
- [ ] Splash/onboarding também têm par dark

Proibido:

- Dark mode = `#000` + texto cinza fraco
- Light mode = branco puro com cinza `#999` em corpo longo
- Acento neon no dark e pastel ilegível no light (ou o inverso) sem ajuste de par

## Movimento

- Curto e físico: 200–350 ms na maioria das transições de UI.
- Easing padrão do sistema (`spring` / Material motion) em vez de bounce teatral.
- Motion reforça hierarquia (aparecer conteúdo, feedback de toque), não decora.
- Respeite `Reduce Motion` / `prefers-reduced-motion` / animator duration scale.
- Loading: skeleton ou progress nativo; evite spinners decorativos demais.

## Ícones e mídia

- Uma família de ícones (SF Symbols / Material Symbols / set do app).
- Pesos alinhados ao tipo (regular com body; filled para seleção).
- Avatares e thumbs com tamanho fixo da escala; corner radius do sistema de shapes.
- Fotos: overlay/scrim quando houver texto sobre imagem — testar light e dark.

## Componentes — consistência cruzada

Mapeie papéis, não widgets 1:1:

| Papel | Compose (orientação) | SwiftUI (orientação) |
|-------|----------------------|----------------------|
| CTA primário | `Button` filled / M3 | `.borderedProminent` |
| CTA secundário | outlined / text | `.bordered` / plain |
| Campo | `OutlinedTextField` ou design system | `TextField` + estilo do app |
| Lista | `LazyColumn` + list item tokens | `List` / LazyVStack estilizado |
| Nav principal | NavigationBar / Tab | `TabView` / toolbar |
| Sheet | Modal bottom sheet | `.sheet` / detents |

Mesmos labels, ordem de ações e hierarquia de botões nas duas plataformas.

## Anti-padrões (UI “IA genérica”)

Evite defaults batidos quando o brief não pedir:

- Roxo/indigo gradient + cards glass em tudo
- Preto + acento ácido como único look “premium”
- Excesso de pills, badges e sombras em camadas
- Onboarding com 3 cards idênticos e ícones coloridos aleatórios
- Tipografia só sistema sem escala pensada + acento genérico
- Dark mode colado depois, com cores hardcoded remanescentes

## Fluxo de implementação

1. Tokens light/dark documentados (ou no código de theme).
2. Tema aplicado na raiz (`MaterialTheme` / `AppTheme` / `.environment`).
3. Tela usando só tokens e componentes do sistema.
4. Preview / `@Preview` / `#Preview` em **light e dark**, fonte normal e grande.
5. Passada de consistência: spacing, raios, acento, estados vazios.

## Critérios de conclusão

- Nenhum color literal solto nas screens tocadas
- Light e dark revisados lado a lado
- Tipografia e spacing da escala do app
- Contraste e touch targets ok
- Parece nativo no OS-alvo e alinhado ao produto
- Reduce Motion / font scaling não quebram o layout
- Compose e SwiftUI (se ambos no escopo) compartilham os mesmos papéis semânticos
