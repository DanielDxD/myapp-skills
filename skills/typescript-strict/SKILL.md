---
name: typescript-strict
description: Modelagem TypeScript estrita com narrowing, generics, validação em runtime e eliminação de any inseguro. Use ao tipar APIs, refatorar tipos, corrigir erros de compilação ou endurecer a segurança de tipos do projeto.
---

# TypeScript Strict

Tipos devem modelar o domínio e falhar cedo. Compile-time não substitui validação na borda.

## Regras

- Trate `strict` como baseline. Não desligue flags para “passar o build”.
- Proíba `any` novo. Prefira `unknown` + narrowing.
- Evite `as` cego; use type guards, predicates e schemas.
- Modele nullability explicitamente (`T | null`), não com sentinelas mágicas.
- Mantenha tipos de domínio em módulos dedicados; não espalhe interfaces duplicadas.

## Modelagem

- Use union types e discriminated unions para estados.
- Prefira `Readonly` / `as const` para dados imutáveis.
- Generics: constranja com `extends`; evite generics decorativos.
- Extraia tipos de valores (`typeof`, `ReturnType`, `Awaited`) em vez de duplicar.
- Para APIs externas, defina tipos de request/response e schemas Zod/Valibot equivalentes.

## Runtime

- Valide na borda: HTTP, forms, env, webhooks, localStorage.
- Inferência: `z.infer<typeof Schema>` como fonte única.
- Erros tipados: result types ou classes de erro discrimináveis.
- Env: parse com schema no boot; falhe se inválido.

## Refatoração segura

1. Introduza tipos no boundary.
2. Empurre `unknown` para dentro até narrowing.
3. Remova casts gradualmente.
4. Rode `tsc --noEmit` / typecheck do projeto após mudanças.

## Anti-padrões

- `any`, `as any`, `@ts-ignore` sem justificativa curta e local
- Interfaces gigantes opcionais demais
- Enums numéricos frágeis quando union de strings basta
- Tipagem só no front enquanto o backend devolve `any`
- Mentir para o compilador em vez de corrigir o modelo

## Critérios de conclusão

- Sem `any` injustificado nas áreas tocadas
- Boundaries validados em runtime
- Unions discriminadas para estados complexos
- Typecheck do projeto passando
- Tipos legíveis e alinhados ao domínio
