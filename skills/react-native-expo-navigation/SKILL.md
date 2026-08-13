---
name: react-native-expo-navigation
description: Navegação com Expo Router no Expo SDK 57 (React Native 0.86) — Stack, Tabs, Drawer, typed routes, deep links, grupos de auth e imports via expo-router. Use ao criar ou migrar navegação Expo, rotas tipadas, layouts ou deep linking em apps SDK 56/57+.
---

# React Native with Expo Navigation (SDK 57)

Expo Router é a navegação padrão em apps Expo SDK 57. File-based routes, layouts nativos e params tipados. Não importe `@react-navigation/*` no código da aplicação — use os entry points do `expo-router` (padrão desde SDK 56).

App geral: `react-native-development`. UI: `better-mobile-interface-react-native`.

## Baseline SDK 57

| Item | Valor |
|------|--------|
| Expo | SDK 57 |
| React Native | 0.86 |
| React | 19.2.x |
| Node | 22.13+ recomendado |

- Alinhe deps com `npx expo install` / `expo install --fix`.
- Rode `npx expo-doctor` após upgrade.
- Preserve a estrutura `app/` existente do projeto.

## Estrutura de rotas

```text
app/
  _layout.tsx          # root providers + Stack/Slot
  index.tsx
  (auth)/
    _layout.tsx
    login.tsx
  (app)/
    _layout.tsx        # tabs ou stack autenticado
    index.tsx
    trip/[id].tsx
  +not-found.tsx
```

- Pastas `()` agrupam layout sem segmento de URL.
- `[param]` e `[...slug]` para dinâmicos.
- `_layout.tsx` define Stack / Tabs / Drawer daquele nível.

## Imports (SDK 56+)

```tsx
// Correto
import { Stack, Tabs, Link, useRouter, useLocalSearchParams } from 'expo-router';
import { ThemeProvider, DarkTheme } from 'expo-router/react-navigation';

// Evitar no app code
import { createNativeStackNavigator } from '@react-navigation/native-stack';
```

| Antes | Depois |
|-------|--------|
| `@react-navigation/native` | `expo-router/react-navigation` |
| `@react-navigation/bottom-tabs` | `expo-router/js-tabs` ou `<Tabs>` |
| `@react-navigation/native-stack` | `<Stack>` do expo-router |

## Regras inegociáveis

- Tipar params (typed routes / helpers do projeto); não confie em strings mágicas.
- Não navegar durante render — só em handlers/efeitos após ação.
- Separar árvores auth vs app (redirect com `useSegments` / `Stack.Protected` / padrão do SDK).
- Deep links e scheme declarados em `app.json` / `app.config`.
- Headers/tabs acessíveis; títulos e labels reais (não “Screen1”).

## Stack, Tabs, Drawer

- Root: providers (SafeArea, theme, query) no `_layout` raiz.
- `Stack` para fluxos lineares (onboarding, trip detail).
- `Tabs` para destinos principais; badge via APIs do Expo Router quando necessário.
- `Drawer` só se o produto já usar esse padrão.
- Modais: `presentation: 'modal'` / formSheet conforme plataforma.

## Auth e guards

```tsx
// Padrão: redirect se sessão ausente
const { user } = useAuth();
const segments = useSegments();
useEffect(() => {
  const inAuth = segments[0] === '(auth)';
  if (!user && !inAuth) router.replace('/(auth)/login');
  if (user && inAuth) router.replace('/(app)');
}, [user, segments]);
```

- Loading de sessão: splash/slot até hidratar auth — evita flash de tela errada.
- Logout limpa stack sensível.

## Deep linking

- Scheme único do app (`myapp://`).
- Universal Links / App Links com associated domains quando em produção.
- Teste cold start e warm start com path + query.
- Params inválidos → not-found ou fallback seguro.

## Anti-padrões

- Importar `@react-navigation/*` direto no app SDK 56/57
- Múltiplos `NavigationContainer` manuais com Expo Router
- `router.push` em loop sem dependências corretas
- Rotas tipadas desligadas “para ir mais rápido”
- Misturar React Navigation clássico e Expo Router sem fronteira

## Critérios de conclusão

- Layouts e rotas alinhados ao SDK 57 / Expo Router atual do projeto
- Auth groups sem flash incorreto
- Params tipados nos fluxos tocados
- Deep link do fluxo principal verificado
- Imports só via `expo-router*`
