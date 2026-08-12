---
name: android-development
description: Desenvolve apps Android nativos com Kotlin, Architecture Components, navegação, ciclo de vida, DI, networking e armazenamento. Use ao criar ou revisar features Android, Activities/Fragments, ViewModels, Repositories, Gradle modules ou integrações de plataforma.
---

# Android Development

Kotlin-first, arquitetura clara e APIs modernas do Jetpack. Preserve a stack do projeto (Hilt/Koin, Retrofit/Ktor, Room, etc.). UI declarativa detalhada fica em `jetpack-compose`; testes em `android-unit-testing`.

## Stack típica

| Peça | Papel |
|------|--------|
| **Kotlin** | Linguagem padrão; coroutines + Flow |
| **Jetpack** | ViewModel, Navigation, Lifecycle, DataStore |
| **DI** | Hilt (comum) ou Koin se já adotado |
| **Rede** | Retrofit/OkHttp ou Ktor Client |
| **Persistência** | Room, DataStore; EncryptedStorage quando necessário |
| **Gradle** | Version catalogs / convention plugins se existirem |

Não migre XML → Compose ou DI framework sem pedido explícito.

## Organização

```text
app/
  src/main/java/.../
    ui/                 # screens / fragments / activities
    feature/<domínio>/  # UI + ViewModel + navigation
    domain/             # use cases, models, interfaces
    data/               # repository impl, dto, sources
    di/
```

- Preferência: feature modules ou pacotes por domínio.
- UI magra; regras em domain/use case; I/O em data.
- Se o app usa MVVM, aplique também `mvvm-architecture`.

## Regras inegociáveis

- Sem trabalho de rede/DB na main thread; use coroutines com `Dispatchers` corretos.
- UI observa estado (StateFlow/LiveData); não consulta repositório direto da View quando houver ViewModel.
- Config e secrets: BuildConfig/local properties/CI secrets — nunca tokens em código.
- Permissões em runtime com rationale e fallback.
- Respeite o lifecycle: colete com `repeatOnLifecycle` / APIs equivalentes; cancele jobs ao destruir.

## Ciclo de vida e processos

- Sobreviva a rotation/process death: estado em ViewModel + `SavedStateHandle` quando necessário.
- Não segure `Context` de Activity em singletons/ViewModels.
- Application Context só para deps sem lifecycle de UI.
- Background: WorkManager para trabalho deferível; Foreground Service só quando a plataforma exige.

## Navegação

- Navigation Component (ou API do projeto) com rotas tipadas/args seguros.
- Deep links: declare intent filters e valide args.
- Single Activity é o padrão moderno; Fragments/Compose destinations conforme o projeto.
- Back stack previsível; não empilhe telas duplicadas sem motivo.

## Dados e sincronização

- Repository como fachada única para a UI/domain.
- DTOs ≠ domain models; mapeie na borda data.
- Offline: cache local + política de refresh explícita.
- Paginação: Paging 3 quando listas forem grandes.
- Erros de rede tipados; retry só em falhas transitivas.

## Concorrência

- `viewModelScope` / `lifecycleScope` — nunca GlobalScope em features.
- `supervisorScope` / exception handlers conscientes.
- Flow: `stateIn`/`shareIn` com `SharingStarted` adequado.
- Evite `runBlocking` em produção.

## UI e recursos

- Compose: use `jetpack-compose`.
- Views XML: ViewBinding; evite `findViewById` e Kotlin synthetics legados.
- Resources: strings, plurals, themes; sem hardcode de copy visível.
- Dark theme e configurações de fonte (acessibilidade).
- Densidades e tamanhos: layouts que não quebrem em small/large.

## Segurança e qualidade

- HTTPS; certificate pinning só se o projeto exigir.
- WebView: JavaScript bridge mínimo e allowlist.
- Storage de tokens: EncryptedSharedPreferences / Keystore patterns do projeto.
- ProGuard/R8: keep rules para reflection/serialization.
- Target/compile SDK alinhados ao projeto; trate deprecations no código tocado.

## Anti-padrões

- God Activity / God ViewModel
- Lógica de negócio em XML listeners ou Composables gigantes
- Vazamento de Context / observers sem remoção
- `!!` e casts cegos em Kotlin
- Network na UI thread
- Dependências novas sem alinhamento ao catálogo Gradle

## Critérios de conclusão

- Feature alinhada à arquitetura e DI existentes
- Estado sobrevive a lifecycle relevante
- I/O off main thread; erros tratados
- Navegação e args seguros
- Permissões e recursos OK
- Pronto para testes (`android-unit-testing`) e UI Compose se aplicável
