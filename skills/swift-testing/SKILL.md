---
name: swift-testing
description: Testes unitários e de integração com Swift Testing (@Test, #expect, #require, @Suite) e testes de interface com XCUITest. Use ao adicionar testes Swift/iOS/macOS, cobrir ViewModels, eliminar flakiness, migrar de XCTest unitário ou automatizar fluxos de UI.
---

# Swift Testing (unitário + interface)

**Swift Testing** é o padrão para unitários e integração em código Swift novo (Xcode 16+). **XCUITest** (XCTest) continua sendo a camada de automação de interface — Swift Testing não substitui UI automation.

## Stack e escopo

| Camada | Framework | Quando |
|--------|-----------|--------|
| Unit / integração / async | **Swift Testing** | Domínio, ViewModels, services, parsers |
| UI automation | **XCUITest** (`XCTestCase`) | Fluxos críticos no Simulator/device |
| Performance (`XCTMetric`) | **XCTest** | Até haver equivalente em Swift Testing |

- `import Testing` nos unitários novos.
- Não misture `#expect` e `XCTAssert` no mesmo teste.
- Ambos podem viver no mesmo projeto (targets separados: Unit Tests vs UI Tests).

## O que priorizar

- ViewModels / `@Observable`: loading, success, error, authz
- Use cases e policies de domínio
- Decoding, validação, formatters
- Regressões (bug → teste)
- Poucos UI tests nos caminhos que geram receita ou perda de dados (login, checkout, create core entity)

## Anatomia Swift Testing

```swift
import Testing

@Test("Parse cursor rejects invalid input")
func parseCursorRejectsInvalid() {
    #expect(throws: CursorError.invalid) {
        try parseCursor("%%%")
    }
}

@Test
func parseCursorAcceptsValue() throws {
    let value = try #require(try? parseCursor("10"))
    #expect(value == 10)
}
```

- `@Test` em funções livres ou métodos de `@Suite`.
- `#expect` — soft assert (continua após falha quando faz sentido no fluxo do teste).
- `#require` — hard unwrap/pré-condição; falha e para o teste.
- Nomes descritivos; display name opcional em `@Test("...")`.

## Suites, traits e paralelismo

```swift
@Suite("TodoViewModel", .tags(.viewModel))
struct TodoViewModelTests {
    @Test func loadEmitsReady() async { /* ... */ }
}
```

- Traits úteis: `.tags(...)`, `.enabled(if:)`, `.disabled(...)`, `.timeLimit(...)`.
- Swift Testing **roda em paralelo por padrão** — sem estado global mutável compartilhado; use fakes por teste ou isolamento explícito.
- Serialização: só quando inevitável (`.serialized` no suite, se disponível na versão do toolchain do projeto).

## Parametrizados

```swift
@Test(arguments: [
    ("", false),
    ("10", true),
])
func cursorCases(input: String, ok: Bool) {
    let result = Result { try parseCursor(input) }
    #expect(result.isSuccess == ok)
}
```

- Cases independentes; a falha deve identificar o argumento.

## Async e concorrência

```swift
@Test
func loadTodos() async throws {
    let repo = FakeTodoRepository(items: [.init(id: "1", title: "Milk")])
    let vm = TodoViewModel(repository: repo)

    await vm.load()

    #expect(vm.state == .ready(count: 1))
}
```

- `async`/`await` nativos — sem `expectation` XCTest para unitários novos.
- Injete clock/repo; evite `Task.sleep` para sincronizar.
- Confirmações / condições: use as APIs de confirmation do Swift Testing disponíveis no toolchain do projeto, não sleeps frágeis.
- `@MainActor` nos tipos de UI/ViewModel quando o código de produção exige.

## Fakes e isolation

```swift
struct FakeTodoRepository: TodoRepository {
    var items: [Todo] = []
    var shouldFail = false

    func all() async throws -> [Todo] {
        if shouldFail { throw RepoError.network }
        return items
    }
}
```

- Prefira protocols + fakes a mocks dinâmicos.
- Sem rede/disco real em unitários.
- Não compartilhe singletons mutáveis entre testes paralelos.

## ViewModel / Observation

- Teste o ViewModel **sem** hospedar `View` SwiftUI.
- Arrange fakes → Act (chamadas async) → Assert estado/`#expect`.
- Cobertura mínima: happy path, vazio, erro de rede, forbidden/unauth se existir.

## Testes de interface (XCUITest)

Target UI Test separado; classes `XCTestCase`:

```swift
final class LoginUITests: XCTestCase {
    var app: XCUIApplication!

    override func setUpWithError() throws {
        continueAfterFailure = false
        app = XCUIApplication()
        app.launchArguments += ["-UITests", "-reset"]
        app.launch()
    }

    func testLoginHappyPath() {
        let email = app.textFields["login.email"]
        email.tap()
        email.typeText("user@example.com")
        app.secureTextFields["login.password"].typeText("secret")
        app.buttons["login.submit"].tap()
        XCTAssertTrue(app.staticTexts["home.title"].waitForExistence(timeout: 5))
    }
}
```

Regras de UI:

- Selecione por **accessibility identifier** estáveis (`login.email`), não por texto localizado frágil.
- Em SwiftUI: `.accessibilityIdentifier("...")` nos controles sob teste.
- `continueAfterFailure = false` em fluxos lineares.
- Launch args / environment para conta de teste, feature flags e skip onboarding.
- Prefira `waitForExistence` a `sleep`.
- Um fluxo por teste; evite suítes monolíticas.
- Rode no Simulator no CI; device só quando necessário.

Pirâmide: muitos Swift Testing → poucos XCUITest.

## Migração desde XCTest unitário

1. Novos unitários → Swift Testing.
2. Ao tocar arquivo XCTest de lógica → converta se o custo for baixo.
3. UI e performance → permanecem XCTest.
4. Não faça big-bang; coexistência é esperada.

## Anti-padrões

- UI test para validar regra de domínio (faça unitário)
- Queries por `staticTexts["Entrar"]` sem identifier (quebra com i18n)
- Estado global + testes paralelos
- `XCTAssert` dentro de `@Test`
- Sleeps fixos; rede real sem contrato
- Snapshot UI em massa sem revisão (só se o time já tiver fluxo)

## Integração

- UI SwiftUI: `swiftui-development`
- Estilo / a11y identifiers coerentes: `better-mobile-interface`
- MVVM testável: `mvvm-architecture`
- Backend Vapor: `vapor-swift` (mesma preferência por fakes e `#expect` quando aplicável)

## Critérios de conclusão

- Unitários novos em Swift Testing (`@Test` / `#expect` / `#require`)
- Casos de risco cobertos; async sem flakiness
- Paralelo-safe (sem shared mutable state)
- UI tests só nos fluxos críticos, com identifiers estáveis
- Suite relevante passa no Xcode / `xcodebuild test` do esquema do projeto
