---
name: typescript-libraries
description: Cria e publica bibliotecas TypeScript — package.json exports, ESM, tipos .d.ts, superfície pública, peerDependencies, SemVer e tree-shaking. Use ao criar ou revisar pacotes npm, SDKs, utilitários compartilhados, monorepo packages ou quando o usuário pedir lib TS, tsup, publint ou dual package.
---

# Bibliotecas TypeScript

Uma lib é código que **o consumidor chama**. Superfície pequena, tipos honestos, publish previsível. Frameworks (IoC, plugins, convenções): skill `typescript-frameworks`. Tipagem de domínio: `typescript-strict`.

## Stack típica

| Peça | Papel |
|------|--------|
| **TypeScript** | `declaration`, `declarationMap`, strict |
| **tsup / unbuild / tsdown** | Bundle ESM (+ CJS só se o público exigir) |
| **Vitest** | Testes da API pública |
| **Changesets** (ou o do repo) | Changelog e bump |
| **publint / attw** | Exports e tipos no artefato |

Preserve o bundler já adotado. Não troque tsup→unbuild numa feature. `tsc` puro serve se o grafo for simples e o `exports` estiver correto.

## Organização

```
src/
  index.ts           # único barrel público (ou subpaths documentados)
  internal/          # não exportar
package.json
tsconfig.json        # DX
tsconfig.build.json  # emit
```

Monorepo:

```
packages/
  foo/               # @scope/foo
    src/index.ts
    package.json
```

- Arquivo público = contrato. Internals nunca saem pelo `exports`.
- Subpath (`@scope/foo/node`, `@scope/foo/browser`) só com motivo (I/O específico).
- Nomes de pacote estáveis; `private: true` até o primeiro publish intencional.

## Regras inegociáveis

- `exports` explícito (`import`, `types`; `require` só se houver CJS). Sem `"main"` órfão divergente.
- `"files"` (ou equivalente) publica só `dist` + README + LICENSE — não `src` a menos que seja a entrega.
- Tipos emitidos (`d.ts`) alinhados ao JS. Não minta com `export {}` vazio ou `any` na borda pública.
- Peer deps para host (React, Vue, Zod major): o app escolhe a versão; a lib não empacota o host.
- Semver: tipo público é API. Mudar assinatura / remover export = **breaking**.
- Tree-shake: named exports, `"sideEffects": false` se for verdade.
- Sem `postinstall` que baixa binários ou altera o projeto do consumidor.

## package.json

```json
{
  "name": "@scope/foo",
  "version": "0.1.0",
  "type": "module",
  "sideEffects": false,
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  },
  "files": ["dist"],
  "peerDependencies": {
    "typescript": ">=5.4"
  },
  "peerDependenciesMeta": {
    "typescript": { "optional": true }
  }
}
```

- Dual CJS/ESM: entrypoints **separados** e tipos correspondentes. Evite o dual package hazard (mesmo módulo resolvido duas vezes).
- `engines` se depender de Node APIs (`node:`). Lib isomórfica não importa `fs` no entry default.
- `imports` internos (`#internal/...`) para não vazar paths relativos no dist.

## API pública

- Minimize `export`. Reexporte só o que o consumidor precisa.
- Prefira funções e tipos nomeados a default export de “namespace objeto”.
- Generics com defaults úteis; `extends` real — não generics decorativos.
- Discriminated unions > flags booleanas combináveis (`isFoo & isBar`).
- Erros: classe ou union `{ ok: false; error: ... }` estável. Não `throw string`.
- Não reexporte tipos de dependência interna (acopla semver do vizinho).
- JSDoc em exports públicos: o que faz, throws, exemplo de uma linha.

```ts
export type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E }

export function parseId(raw: string): Result<Id, ParseError> { /* ... */ }
```

Deprecação: `@deprecated` + export antigo por **um** major; depois remova.

## tsconfig de build

- `strict` ligado. `declaration` + `declarationMap`.
- `rootDir` / `outDir` claros; não emita testes no `dist`.
- `stripInternal`: `/** @internal */` some dos `.d.ts` públicos.
- `moduleResolution: bundler` ou `nodenext` — o que o `exports` exige; não misture.

## Testes e qualidade

- Teste a superfície pública (`from '@scope/foo'`), não internals.
- Matrix mínima: versão de TypeScript suportada (`tstyche` / `tsd` se a lib for types-first).
- `publint` e arethetypeswrong (`attw`) no CI antes do npm publish.
- Bundle: olhe tamanho do entry; evite puxar `lodash` inteiro.

## Publish

- Changelog por changeset; tag git = versão.
- Provenance / `npm publish --access public` conforme o scope.
- Não publique `latest` quebrado: `prepack` roda build; `prepublishOnly` impede publish sujo.
- Breaking: major, guia de migração curto no CHANGELOG.

## Anti-padrões

- `export * from './internal'`
- `"main": "./src/index.ts"` como entrega de produção
- Empacotar React/Vue dentro da lib
- `types` apontando para outro arquivo que o `import`
- Default export + named do mesmo valor de formas incompatíveis CJS/ESM
- `skipLibCheck` para esconder `.d.ts` errados da própria lib
- Mudar tipos públicos em patch

## Integração com outras skills

- Tipos e narrowing: `typescript-strict`
- IoC / plugins / convenções: `typescript-frameworks`
- Componentes React publicados: `react-component-engineering`
- Design system: `design-system`
- HTTP SDK: `api-design`

## Critérios de conclusão

- `exports` + `files` + tipos coerentes (`publint`/`attw` ok)
- Superfície pública mínima, JSDoc nos exports
- Peers corretos; sem host bundlado
- Testes via entry público
- SemVer honesto; changelog da mudança
- Build reproduzível; `dist` não commitado (salvo o repo já versionar artefato)
