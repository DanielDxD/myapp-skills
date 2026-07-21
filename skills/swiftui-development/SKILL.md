---
name: swiftui-development
description: Desenvolve interfaces SwiftUI com composição de views, Observation e state wrappers, navegação, concorrência Swift, arquitetura, acessibilidade, previews, performance e testes. Use ao criar apps ou features iOS/macOS em SwiftUI.
---

# SwiftUI Development

Composição declarativa, estado previsível e concorrência segura. Prefira views pequenas e ViewModels testáveis quando o projeto usa MVVM.

## Composição

- Views pequenas, nomeadas pelo propósito.
- Extraia subviews e `ViewModifier`s em vez de bodies gigantes.
- Prefira composição a herança.
- Containers de layout (`VStack`, `Grid`, `Layout`) com spacing do design system do app.

## Estado

- `@State` — estado local de view.
- `@Binding` — estado emprestado.
- `@Observable` / Observation — modelos e ViewModels modernos.
- `@Environment` — dependências transversais (não para todo estado de feature).
- Evite `@ObservedObject` legado em código novo se Observation estiver disponível no deployment target.

Regras:

- Fonte única da verdade.
- Side effects em `.task` / `.onChange` com cancelamento cooperativo.
- Não faça rede no `body`.

## Navegação

- `NavigationStack` com valor tipado (`navigationDestination`).
- Sheets/fullScreenCover com identidade estável.
- Deep links: parse → estado de rota → stack.
- Evite `NavigationView` legado em código novo.

## Concorrência

- `async/await` e `Task` com cancelamento em `.task`.
- `@MainActor` para estado de UI.
- Não bloqueie a main thread com I/O.
- Modele erros com `throws` e estados de falha na UI.

## Arquitetura

- Se o app usa MVVM, aplique a skill `mvvm-architecture`.
- Services/repositórios injetáveis para teste.
- Separe networking de View.

## Acessibilidade e UI polish

- Labels, traits, Dynamic Type.
- Touch targets e contraste.
- Reduzir motion quando `accessibilityReduceMotion`.
- SF Symbols com hierarquia tipográfica coerente.

## Previews e performance

- Previews com dados de exemplo e estados (loading/error/empty).
- Evite trabalho pesado em `body`; use `EquatableView` / diffing consciente quando necessário.
- Listas grandes: `List`/`LazyVStack` adequadas; identifique com `id` estável.
- Profile com Instruments se houver jank.

## Testes

- ViewModels/services com XCTest.
- UI tests para fluxos críticos.
- Snapshots só se o projeto já depender deles.

## Anti-padrões

- Massive `ContentView`
- Rede e parsing no `body`
- `@State` para dados de servidor sem camada de modelo
- Force unwrap em caminhos de produção
- Ignorar cancelamento de tasks

## Critérios de conclusão

- Estado e concorrência corretos na MainActor
- Navegação tipada estável
- Previews dos estados principais
- A11y básica coberta
- Sem force unwraps desnecessários no caminho feliz/erro tratado
