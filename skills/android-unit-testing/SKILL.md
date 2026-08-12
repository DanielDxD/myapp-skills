---
name: android-unit-testing
description: Testes unitários Android/Kotlin com JUnit, coroutines TestDispatcher, Turbine, fakes, MockK opcional e testes de ViewModel. Use ao adicionar testes, cobrir ViewModels/use cases, eliminar flakiness ou definir estratégia de teste em projetos Android.
---

# Android Unit Testing

Teste regras de risco em JVM (`test/`), não em device, sempre que possível. UI Compose instrumentada é complementar — não substitua unitários de domínio/ViewModel.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **JUnit 4/5** | Runner e asserts do projeto |
| **kotlinx-coroutines-test** | `runTest`, `TestDispatcher`, `advanceUntilIdle` |
| **Turbine** (se presente) | Asserts em Flow |
| **Fakes** | Repos, clock, data sources |
| **MockK / Mockito** | Só se o projeto já usar |
| **Robolectric** | Quando precisar de framework Android leve em JVM |

Prefira fakes a mocks. Não adicione MockK/Robolectric só para um teste trivial.

## O que priorizar

- ViewModels: estados loading/success/error e authz
- Use cases e policies de domínio
- Mapeamento DTO ↔ domain
- Parsers/validação
- Regressões (bug → teste)
- Reducers / state holders Compose (sem UI) quando existirem

## Layout

```text
src/
  main/...
  test/.../          # unit JVM
  androidTest/.../   # instrumentado (Espresso/Compose) — só fluxos que exigem
```

- Espelhe o package de `main`.
- Nome: `FooViewModelTest`, `ParseCursorTest`.

## ViewModel + coroutines

```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
class TodoViewModelTest {
    private val dispatcher = StandardTestDispatcher()

    @Test
    fun load_emitsReady() = runTest(dispatcher) {
        Dispatchers.setMain(dispatcher)
        try {
            val repo = FakeTodoRepo(listOf(Todo("1", "Buy milk")))
            val vm = TodoViewModel(repo)

            vm.load()
            dispatcher.scheduler.advanceUntilIdle()

            assertEquals(UiState.Ready(/* ... */), vm.state.value)
        } finally {
            Dispatchers.resetMain()
        }
    }
}
```

- Use `runTest` + `StandardTestDispatcher` / `UnconfinedTestDispatcher` conforme o caso.
- Sempre `setMain`/`resetMain` em testes que tocam `viewModelScope` + Main.
- Evite `Thread.sleep`; avance o scheduler.
- Estado: observe `StateFlow`/`Flow` após idle; com Turbine:

```kotlin
vm.state.test {
    assertEquals(UiState.Loading, awaitItem())
    assertTrue(awaitItem() is UiState.Ready)
    cancelAndIgnoreRemainingEvents()
}
```

## Table-driven / parametrizado

```kotlin
@Test
fun cursorCases() {
    data class Case(val input: String, val ok: Boolean)
    listOf(
        Case("", false),
        Case("10", true),
    ).forEach { case ->
        val result = parseCursor(case.input)
        assertEquals(case.ok, result.isSuccess, case.input)
    }
}
```

- Nomeie o comportamento na mensagem de falha.
- Cases independentes.

## Fakes

```kotlin
class FakeTodoRepo(
    private var items: List<Todo> = emptyList(),
    var fail: Boolean = false,
) : TodoRepo {
    override suspend fun all(): List<Todo> {
        if (fail) error("network")
        return items
    }
}
```

- Implemente interfaces de domínio/data.
- Controle falhas e dados por teste.
- Clock fake para expiração/tokens/cache.

## O que não fazer em unitário JVM

- Iniciar Activity/Fragment real sem Robolectric justificado
- Room/Retrofit reais — use fake ou in-memory só quando o teste for do DAO e o projeto já tiver padrão
- Dependência de tempo/rede real
- Ordem de execução entre classes

## Compose (nível unitário)

- Teste estado/holders e funções puras sem UI.
- UI: `createComposeRule` em `androidTest` (ou Robolectric se o projeto já configurar).
- Semântica: `onNodeWithText` / `onNodeWithTag` com tags de teste estáveis (`testTag`).
- Não transforme todo unitário em UI test.

## Flakiness e CI

- Determinismo: dispatcher de teste, fakes, seeds fixos.
- Isolamento: sem singletons mutáveis compartilhados sem reset.
- CI: `./gradlew test` (ou task do módulo) no PR; instrumentados em pipeline separado se caros.
- `-Pandroid.testInstrumentationRunnerArguments...` só quando necessário.

## Anti-padrões

- Mockar a classe sob teste
- Assertar detalhes de implementação (quantas vezes chamou log)
- Testes que passam só com internet
- `runBlocking` sem `runTest` em código de coroutine
- Cobertura de linhas no lugar de riscos

## Integração

- App e arquitetura: `android-development`
- UI: `jetpack-compose`
- MVVM: `mvvm-architecture`

## Critérios de conclusão

- Riscos da mudança cobertos em `src/test`
- Coroutines determinísticas (`runTest` + Main fake)
- Fakes nas bordas; asserts legíveis
- Sem flakiness óbvia
- `./gradlew test` (módulo tocado) passa
