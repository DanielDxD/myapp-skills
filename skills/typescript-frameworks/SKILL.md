---
name: typescript-frameworks
description: Projeta frameworks TypeScript com inversão de controle, plugins, ciclo de vida, convenções, DX e API estável. Use ao criar ou evoluir frameworks, SDKs opinativos, runtimes de plugin, meta-frameworks, CLIs com hooks ou quando o usuário pedir plugin API, IoC ou extension points em TS.
---

# Frameworks TypeScript

Um framework **chama o código do usuário**. Convenção, ciclo de vida e pontos de extensão vencem um “kit de helpers” grande. Pacotes publicáveis e `exports`: skill `typescript-libraries`. Tipos: `typescript-strict`.

## Lib vs framework

| | Biblioteca | Framework |
|--|------------|-----------|
| Controle | Consumidor chama | Framework chama (IoC) |
| Forma | Funções/tipos pequenos | App shape, lifecycle, plugins |
| Config | Argumentos | Convenção + config tipada + escape hatches |
| Sucesso | API mínima e estável | DX: defaults certos, erros acionáveis, extensão segura |

Se o usuário só precisa de funções, não invente um framework.

## Stack típica

| Peça | Papel |
|------|--------|
| **TypeScript** | Contratos públicos, module augmentation consciente |
| **Plugin API** | `setup` / hooks / `resolveId` — o do domínio |
| **Config schema** | Zod/Valibot/TypeBox no boot |
| **CLI** (opcional) | `create-*`, `dev`, `build` |
| **Harness de teste** | App mínimo + plugin fake |

Entrega npm segue `typescript-libraries` (exports, peers, semver).

## Organização

```
src/
  index.ts              # API pública: createApp, definePlugin, tipos
  core/                 # lifecycle, DI, graph
  plugins/              # plugins oficiais (opcionais via subpath)
  config.ts             # load + validate
  errors.ts             # erros com hint
packages/               # monorepo: core, cli, plugin-*
create-foo/             # scaffold
```

- Core pequeno. Features oficiais são plugins (mesmo que first-party).
- O usuário não importa `core/internal`.
- Convenções de arquivos (`app/`, `plugins/`, `routes/`) documentadas e estáveis.

## Regras inegociáveis

- **Ordem do lifecycle** documentada e testada (register → boot → ready → close).
- Plugins são **composição**, não monkey-patch de internals.
- Config validada no boot; falha com caminho do campo e valor esperado.
- API pública vs `experimental` vs `@internal` — sem cinza.
- Breaking de hook/config = major. Prefira adicionar hook a mudar payload.
- Erros dizem **o que fazer** (arquivo, opção, link). Não `undefined is not a function` na borda.
- Escape hatch explícito (`hooks`, `raw` options) melhor que “leia o source do core”.
- Defaults que funcionam localmente; nada de 40 flags obrigatórias no `create`.

## Ciclo de vida e IoC

O framework é o dono do fluxo. O usuário preenche slots.

```ts
export interface Plugin<TOptions = unknown> {
  name: string
  setup?: (ctx: FrameworkContext, options: TOptions) => void | Promise<void>
}

export function definePlugin<T>(plugin: Plugin<T>): Plugin<T> {
  return plugin
}

// createApp() registra plugins, valida config, dispara hooks na ordem.
```

- `definePlugin` / `defineConfig` existem para inferência, não para magia.
- Contexto (`app`, `config`, `hooks`, `logger`) é a unidade de extensão — não globais.
- Hooks: nomes estáveis (`build:done`), payload versionado. Evite `on('*')` como API pública.
- Async: `await` hooks em série quando a ordem importa; paralelo só se for independente e documentado.
- Shutdown: `close`/`dispose` em ordem inversa ao boot; plugins não vazam handles.

## Convenção e config

- Convenção sobre configuração: pastas e nomes default; config só para desvio.
- Um objeto de config **tipado** (`FooConfig`); nested por domínio, não 200 keys no root.
- `satisfies FooConfig` / `defineConfig()` para autocomplete sem `any`.
- Feature flags de framework são estáveis ou experimentais — não “beta eterno” na API default.
- Resolução de path relativa ao root do projeto, não ao `cwd` acidental.

## Tipos como contrato

- O usuário estende via generics do `createApp` ou via module augmentation **documentada** (`declare module 'foo'`).
- Não abuse de declaration merging para estado global implícito.
- Plugins que adicionam campos no `app` devem tipar esse aumento (declaration merging do plugin **opt-in**).
- Evite `any` no `ctx`; o plugin mal tipado contamina o app inteiro.
- Generics fluem do config → app → plugins; defaults sensatos.

## DX

- Mensagens: problema + local + correção. Ex.: “Plugin `auth` duplicado. Use `name` único ou `enforce: 'pre'`.”
- Stack traces apontam para o arquivo do usuário, não só para o core minificado (source maps no CLI).
- Scaffold (`create-foo`) gera o happy path: tsconfig, script `dev`, um plugin exemplo.
- Docs: ciclo de vida, ordem de plugins, “como escrever um plugin”, o que é estável.
- Dev mode rápido; cache de graph com invalidação explícita.

## Plugins oficiais vs ecossistema

- First-party: qualidade de referência (tipos, testes, erros).
- Third-party: contrato estável; sem importar internals (`foo/dist/core.js`).
- Sem versionar plugin contra `*` do core — peer `^` no major atual.
- Isolamento: plugin A não quebra B; ordem `enforce: 'pre' | 'post'` só quando o grafo exigir.

## Testes

- Harness: `createTestApp({ plugins })` in-process.
- Cubra: ordem de hooks, falha de config, plugin duplicado, close/dispose, extensão de tipos (teste de compilação).
- Snapshot de mensagens de erro **estáveis** (sem paths absolutos de máquina).

## Anti-padrões

- God `config: Record<string, any>`
- Plugin que importa arquivos internos do core
- Lifecycle não documentado / ordem diferente em dev vs prod
- Transformar o framework numa linguagem (DSL pesada) sem necessidade
- Breaking silencioso de hooks em minor
- Singleton global (`getApp()`) como única forma de extensão
- Scaffold que gera código depreciado
- “Lib de 80 helpers” vendida como framework

## Integração com outras skills

- Empacotar e publicar: `typescript-libraries`
- Tipos estritos e schemas: `typescript-strict`
- CLI de app vs lib: preserve o bundler do repo
- HTTP se o framework for server: `api-design`
- UI kit: `react-component-engineering` / `design-system` — não misturar com o core do runtime

## Critérios de conclusão

- `create*` + `definePlugin`/`defineConfig` tipados
- Lifecycle documentado, testado, com shutdown
- Config validada com erros acionáveis
- Superfície estável vs experimental marcada
- Plugins não dependem de internals
- Scaffold ou exemplo mínimo sobe o happy path
- Empacotamento alinhado a `typescript-libraries`
