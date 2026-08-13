---
name: better-mobile-interface-react-native
description: Regras de estilo para interfaces React Native/Expo modernas, bonitas e consistentes, com tokens semânticos e paridade light/dark. Use ao criar ou revisar UI RN, theming, design system mobile Expo ou quando pedir interface polida em React Native.
---

# Better Mobile Interface (React Native)

Desenhe o app RN como um sistema de tokens e hierarquia — não telas com hex soltos. Parece nativo em iOS e Android, com a mesma linguagem visual do produto.

Nativo Compose/SwiftUI: `better-mobile-interface`. Navegação: `react-native-expo-navigation`. App geral: `react-native-development`.

## Resultado esperado

- Hierarquia óbvia em 3 segundos (título, apoio, CTA).
- Light e dark com os **mesmos roles** semânticos.
- Espaçamento e raios consistentes em todo o fluxo.
- Touch targets ≥ 44–48 dp; contraste AA.
- Sem “UI de template IA” (roxo genérico, glass em tudo, pills demais).

## Tokens semânticos (obrigatório)

| Role | Função |
|------|--------|
| `background` | Fundo da tela |
| `surface` / `surfaceElevated` | Cards, sheets |
| `textPrimary` / `textSecondary` / `textTertiary` | Hierarquia de texto |
| `border` / `separator` | Divisores |
| `accent` / `onAccent` | CTA e marca |
| `success` / `warning` / `danger` | Feedback |
| `overlay` | Scrim |

- Defina pares light/dark na mesma tabela.
- Consuma via theme (`useTheme`, context, NativeWind tokens ou StyleSheet factory) — **nunca** `#RRGGBB` espalhado nas screens.
- Se o projeto já tem design system, ele vence.

## Tipografia e espaço

- Escala nomeada: `display`, `title`, `body`, `label`, `caption`.
- Prefira fonte do sistema ou a brand font do app; no máximo duas famílias.
- Base 4/8: margens 16–20; gaps da escala.
- `allowFontScaling`; layouts não quebram com fonte grande.
- Truncate consciente (`numberOfLines`) em listas.

## Layout RN

- `SafeAreaProvider` + insets; respeite home indicator e notches.
- Flexbox previsível; evite absolute para estrutura principal.
- Uma ação primária por tela (botão filled / accent).
- Listas: título → subtítulo → meta; separators com `separator` token.
- Empty / loading / error são estados desenhados (skeleton ou copy + CTA).
- Teclado: `KeyboardAvoidingView` / `KeyboardStickyView` do stack do projeto.

## Light e dark

Checklist:

- [ ] Todos os roles têm par light/dark
- [ ] StatusBar / navigation bar contrastam com o fundo
- [ ] Imagens/ícones legíveis nos dois modos (tint / variantes)
- [ ] Borders visíveis no dark (não só sombra)
- [ ] Acento com contraste em ambos

Proibido: dark = `#000` + cinza ilegível; light = branco + `#999` em corpo longo.

## Componentes

- Botões: primary / secondary / tertiary com estados pressed/disabled.
- Inputs: label permanente; erro abaixo; altura mínima tocável.
- Cards: radius `md`/`lg` do sistema; padding interno consistente.
- Tab bar / headers: use tokens, não cores hardcoded do navigator.

NativeWind/Tamagui/Restyle: ok se já forem o padrão — mapeie classes/tokens aos roles acima.

## Movimento

- 200–350 ms; Reanimated/LayoutAnimation alinhados ao projeto.
- Respeite Reduce Motion (`AccessibilityInfo` / `useReducedMotion`).
- Feedback de toque (opacity/haptic) sem exagero.

## Anti-padrões

- Hex literais em cada tela
- Clonar UI web (divs mentais, hover-only)
- Densidade inconsistente entre stacks de navegação
- Dark mode “depois” com leftovers light-only
- Excesso de badges, gradients e shadows

## Critérios de conclusão

- Screens tocadas só usam tokens
- Light e dark revisados lado a lado
- Touch targets e contraste ok
- Estados empty/loading/error cobertos
- Consistente com navegação e marca do app
