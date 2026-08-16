---
name: nextjs-server-actions
description: Implementa mutações Next.js App Router com Server Actions — "use server", forms, useActionState, validação, revalidate e redirects. Use ao criar ou refatorar actions, formulários de mutação, progressive enhancement ou quando o usuário mencionar Server Actions.
---

# Next.js Server Actions

Actions são mutações no servidor invocáveis por form/`startTransition` — não um substituto de GET. Arquitetura App Router: `nextjs-architecture`. Segurança (authz, CSRF, IDOR): `nextjs-server-actions-security`.

## Quando usar

| Use Server Action | Use Route Handler |
|-------------------|-------------------|
| Mutação do próprio app (form, botão) | API pública, mobile, terceiros |
| Progressive enhancement com `<form>` | Webhooks, cron, upload direto |
| Co-localizada à UI que muta | Contrato REST/JSON estável para clientes externos |

Não faça fetch de leitura em action. Prefira Server Components / loaders.

## Organização

```
features/<domínio>/
  actions.ts          # "use server" — mutações do domínio
  schemas.ts          # Zod (ou equivalente)
  components/         # forms client ou server
```

- `"use server"` no **topo do arquivo** de mutações importáveis, ou inline em Server Components.
- Uma action = uma intenção de negócio (`createOrder`, `archivePost`) — não um “controller faz tudo”.
- Extraia I/O (DB, mail) para services; a action orquestra validate → authz → mutate → revalidate.

## Regras inegociáveis

- Input validado com schema no servidor (FormData ou objeto). Tipos do client não são garantia.
- Authn/authz **dentro** da action (ver skill de segurança).
- Retorno serializável: `{ ok: true, data }` / `{ ok: false, code, message }` — ou `redirect`/`notFound`.
- Após write: `revalidatePath` / `revalidateTag` / `updateTag` do que ficou stale.
- Sem secrets, Prisma client ou módulos Node no bundle de Client Components — só importe a action.

## Forms e progressive enhancement

```tsx
// Server Component — funciona sem JS
import { createTodo } from "./actions";

export function CreateTodoForm() {
  return (
    <form action={createTodo}>
      <input name="title" required maxLength={200} />
      <button type="submit">Add</button>
    </form>
  );
}
```

```tsx
"use client";
import { useActionState } from "react";
import { createTodo } from "./actions";

export function CreateTodoForm() {
  const [state, formAction, pending] = useActionState(createTodo, null);
  return (
    <form action={formAction}>
      <input name="title" required />
      <button disabled={pending}>Add</button>
      {state && !state.ok ? <p>{state.message}</p> : null}
    </form>
  );
}
```

- `useFormStatus` no botão filho para pending sem elevar estado.
- `useActionState` para erros de validação/domínio de volta ao form.
- Hidden fields só para dados não sensíveis já conhecidos do server (IDs que a action **revalida**).

## Action idiomática

```ts
"use server";

import { z } from "zod";
import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";

const Schema = z.object({
  title: z.string().trim().min(1).max(200),
});

export async function createTodo(prev: unknown, formData: FormData) {
  const parsed = Schema.safeParse({ title: formData.get("title") });
  if (!parsed.success) {
    return { ok: false as const, code: "validation", message: "Invalid title" };
  }

  // authz + persist...
  revalidatePath("/todos");
  redirect("/todos");
}
```

- Assinatura `(prevState, formData)` quando usada com `useActionState`; senão `(formData)` ou args explícitos.
- `redirect()` lança — não misture com `return` depois.
- Extra args: `.bind(null, id)` no server; o id ainda é input hostil — revalide ownership.

## Cache e navegação

- `revalidateTag("todos")` quando o fetch usou a mesma tag.
- `revalidatePath` para a rota visível; evite revalidate global sem motivo.
- `redirect` após create/delete que muda a URL; senão retorne data e deixe o form/router atualizar.
- Não chame `cookies()`/`headers()` sem necessidade — isso opta a rota para dinâmica.

## Erros

- Esperados (validação, 409, sem permissão): objeto `{ ok: false, code, message }` seguro para UI.
- Inesperados: logue + error boundary / `digest`; não vaze stack ao client.
- Não use `try/catch` para engolir falha de `redirect` sem reraisar.

## Anti-padrões

- Action que só faz `fetch` para o próprio Route Handler
- Validação só no client (`zod` no form sem parse na action)
- Retornar entidades completas do DB (hash, tokens, PII)
- `"use server"` em arquivo misturado com helpers de client
- Mutação sem revalidate (UI stale)
- Leitura/`searchParams` via action

## Fluxo ao implementar

1. Schema de input + tipo de resultado.
2. Action: parse → sessão → policy → persist → revalidate/redirect.
3. Form server ou `useActionState` no island.
4. Teste happy path + validação + unauthenticated (skill de segurança).

## Critérios de conclusão

- `"use server"` isolado e importável sem vazar server-only
- Schema na borda da action
- Revalidate/redirect corretos após o write
- Resultado serializável e útil para o form
- Sem leitura disfarçada de action
- Authz coberta (`nextjs-server-actions-security`)
