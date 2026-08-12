---
name: jetpack-compose
description: Desenvolve UI Android com Jetpack Compose — composição, estado, Material 3, navegação, listas, efeitos colaterais, acessibilidade e performance. Use ao criar ou refatorar screens Compose, componentes reutilizáveis, theming ou migração pontual de Views para Compose.
---

# Jetpack Compose

UI declarativa: estado desce, eventos sobem. Composables pequenos, estáveis e previsíveis. Arquitetura de app e DI ficam em `android-development`; MVVM em `mvvm-architecture`. Estilo visual, tokens e paridade light/dark: `better-mobile-interface`.

## Princípios

- Composable = função de estado → UI. Sem side effects no corpo sem `LaunchedEffect`/`SideEffect`/etc.
- Estado o mais baixo possível; eleve só para compartilhar.
- Unidirectional data flow: `state` + `onEvent` / lambdas.
- Hoisting: componentes burros recebem valores e callbacks.
- Stability importa para recomposição; evite tipos instáveis desnecessários em params quentes.

## Estado

| API | Uso |
|-----|-----|
| `remember` | Cache de valor na composição |
| `rememberSaveable` | Sobrevive a recreation (bundle) |
| `mutableStateOf` / `MutableState` | Estado observável local |
| `collectAsStateWithLifecycle` | Flow/StateFlow com lifecycle |
| `ViewModel` + `StateFlow` | Estado de tela |

- Não use `ViewModel` para estado de widget efêmero (expand/collapse local).
- Evite `LiveData.observeAsState` em código novo se Flow estiver disponível.
- Derived state: `derivedStateOf` para cálculos derivados caros/filtrados.

## Side effects

- `LaunchedEffect(key)` — coroutines ligadas ao ciclo da composição.
- `DisposableEffect` — register/unregister com cleanup.
- `SideEffect` — publicar para APIs não-Compose.
- `rememberCoroutineScope` — ações disparadas por evento de UI.
- Nunca faça rede/IO direto no corpo do `@Composable`.

## Layout e Material

- Prefira Material 3 (`MaterialTheme`) alinhado ao design system do app.
- `Modifier` na ordem correta (size/padding/clickable/semantics).
- Slot APIs (`topBar`, `content`) em vez de parâmetros booleanos explosivos.
- Theming: colorScheme, typography, shapes — sem cores mágicas espalhadas.
- Dark theme e dynamic color só se o produto já adotar.

## Listas e performance

- `LazyColumn`/`LazyRow`/`LazyGrid` com `key` estáveis.
- Evite lambdas e objetos novos instáveis em items quentes sem necessidade.
- `contentType` quando ajudar o recycling.
- Imagens: lib do projeto (Coil/Glide Compose) com tamanho adequado.
- Meça recomposições (Layout Inspector / Compose metrics) antes de micro-otimizar.

## Navegação Compose

- Navigation Compose com rotas tipadas (type-safe routes se o projeto usa).
- Args serializáveis e defaults seguros.
- Escopos de `ViewModel` corretos (`navigation` back stack entry).
- Deep links integrados ao grafo.
- Evite múltiplos `NavController` sem hierarquia clara.

## Interop Views

- `AndroidView` / `AndroidViewBinding` para Views legadas pontuais.
- `ComposeView` em Fragments/Activities XML.
- Não misture dois sistemas de estado na mesma tela sem fronteira clara.

## Acessibilidade

- `contentDescription`, `semantics`, roles e merge quando necessário.
- Touch targets mínimos (~48dp).
- Fonte escalável; layouts que não cortam com fontScale alto.
- Ordem de foco e announcements coerentes.

## Previews e tooling

- `@Preview` com estados: loading, empty, error, content.
- Previews com tema claro/escuro e tamanhos de tela relevantes.
- Extraia preview data fakes; não dependa de Hilt real no preview.

## Anti-padrões

- Composable deus com rede, DB e navegação
- `GlobalScope` / coroutines sem cancelamento
- Estado duplicado (ViewModel + remember do mesmo campo)
- `Modifier` omitido em componentes públicos (dificulta layout)
- Recomposição causada por `List`/`Map` instáveis recriados a cada emit
- Ignorar configuration changes em estado crítico

## Critérios de conclusão

- UDF claro; side effects só em effect APIs
- Listas com keys; temas via `MaterialTheme`
- A11y básica na feature
- Previews dos estados principais
- Sem IO no corpo do Composable
- Integração com ViewModel/DI do projeto preservada
