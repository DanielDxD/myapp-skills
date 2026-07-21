---
name: react-native-development
description: Desenvolve apps Expo e React Native com navegação, estado, listas, animações, módulos nativos, diferenças iOS/Android, acessibilidade e testes. Use ao criar ou refatorar apps mobile React Native, telas Expo Router ou bridges nativas.
---

# React Native Development

Mobile-first: performance de lista, safe areas, gestos e diferenças de plataforma importam tanto quanto a UI.

## Fundação

- Prefira o workflow do projeto (Expo managed / prebuild / bare).
- TypeScript estrito; tipar params de navegação.
- Organizar por feature: `screens`, `components`, `hooks`, `api`.
- Não copie padrões web cegamente (`div`, CSS livre, scroll da window).

## Navegação

- React Navigation ou Expo Router conforme o repo.
- Tipar rotas e params.
- Headers, deep links e estados de auth (stack separada logged-in/out).
- Evite navegar durante render; faça em efeitos/respostas a ações.

## UI e layout

- `SafeAreaProvider` / insets.
- Flexbox com cuidado em telas pequenas; teste iPhone SE e Android comum.
- Toque: área mínima ~44pt; feedback pressionado.
- Teclado: `KeyboardAvoidingView` / avoiding libs do projeto.
- Dark mode e temas via tokens do app.

## Listas e performance

- `FlatList`/`SectionList`/`FlashList` com `keyExtractor` estável.
- `getItemLayout` quando altura fixa.
- Evite inline functions/objects pesados em `renderItem` sem necessidade.
- Imagens: cache e tamanhos adequados; preferir libs já no projeto.
- Anime com Reanimated/Gesture Handler quando o projeto já usa; mantenha worklets corretos.

## Estado e dados

- Server state com React Query ou padrão do app.
- Storage seguro para tokens (SecureStore/Keychain), não AsyncStorage para secrets.
- Offline: fila de mutações só com desenho explícito.

## Nativo e plataformas

- `Platform.select` / arquivos `.ios.tsx` / `.android.tsx` quando a UX diverge.
- Permissões com mensagens claras e fallback.
- Módulos nativos: documente config de app.json/plugins e rebuild necessário.
- Respeite guidelines de store (privacidade, background modes).

## A11y e QA

- `accessibilityLabel` / role / state.
- Dynamic Type / font scaling: layouts que não quebrem.
- Testes: Jest + RNTL; E2E (Maestro/Detox) para fluxos críticos se existir no repo.

## Anti-padrões

- ScrollView com mapa de 500 itens
- Secrets em AsyncStorage
- Assumir parity total iOS/Android sem testar
- Bloquear JS thread com trabalho pesado
- Imports web-only (`localStorage`, `window`)

## Critérios de conclusão

- Telas tipadas e navegáveis
- Lista principal performática
- Auth/storage seguros
- Comportamento ok em iOS e Android (ou gap documentado)
- A11y básica na feature tocada
