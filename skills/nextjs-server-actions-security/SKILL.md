---
name: nextjs-server-actions-security
description: Endurece Server Actions Next.js contra IDOR, CSRF, mass assignment, vazamento de dados e abuso. Use ao auditar actions, revisar authz em mutações, configurar allowedOrigins ou quando o usuário pedir segurança de Server Actions.
---

# Next.js Server Actions Security

Toda Server Action é um endpoint POST público. O bundler esconde o nome; **não esconde** o contrato. Quem pode chamar a action é a rede — a action deve falhar fechada.

Implementação de actions: `nextjs-server-actions`. Authn/authz geral: `authentication-authorization`.

## Modelo de ameaça

| Ameaça | Por quê |
|--------|---------|
| **Endpoint enumerável** | IDs de action vazam no client bundle; POST pode ser forjado |
| **IDOR** | `id` no FormData/bind vem do atacante |
| **CSRF / origem cruzada** | Browser autenticado + POST cross-site |
| **Mass assignment** | Spread de FormData no `update` |
| **Data leak** | Return/throw com campos internos |
| **Abuse** | Sem rate limit em login, pagamento, invite |

## Regras inegociáveis

1. **Authn + authz no corpo da action** — middleware/layout não basta (podem ser contornados).
2. **Fail closed** — sem sessão ou sem policy → `{ ok: false, code: "unauthorized"|"forbidden" }` ou `forbidden()`.
3. **Nunca confie em IDs/campos do client** — recarregue o recurso e cheque ownership/tenant.
4. **Parse allowlist** — schema Zod (ou equivalente); ignore campos extras.
5. **Resposta mínima** — sem hashes, tokens, PII, stack, SQL.

## Authz dentro da action

```ts
"use server";

export async function deletePost(formData: FormData) {
  const user = await requireUser(); // cookies/session no server
  const id = z.string().uuid().parse(formData.get("id"));

  const post = await db.post.findUnique({ where: { id } });
  if (!post || post.tenantId !== user.tenantId) {
    return { ok: false as const, code: "not_found" }; // não vaze existência cross-tenant
  }
  if (post.authorId !== user.id && user.role !== "admin") {
    return { ok: false as const, code: "forbidden" };
  }

  await db.post.delete({ where: { id } });
  revalidatePath("/posts");
  return { ok: true as const };
}
```

- `requireUser()` em **toda** mutação autenticada; actions públicas (ex. newsletter) são exceção explícita + rate limit.
- Distinga 401 (anônimo) de 403 (autenticado sem permissão) internamente; para IDOR multi-tenant prefira 404 uniforme.
- Roles no JWT: revalide permissões críticas no servidor (sessão/DB), não só claim antiga.

## CSRF, origem e proxies

- Next.js verifica `Origin`/`Host` (e headers equivalentes) em Server Actions nas versões recentes — **não desligue**.
- Atrás de reverse proxy: configure `serverActions.allowedOrigins` (e `experimental.serverActions` se o projeto ainda usar a chave antiga) com os hosts reais.
- Cookies de sessão: `HttpOnly`, `Secure`, `SameSite=Lax` (ou `Strict` se o fluxo permitir). `SameSite=None` só com motivo.
- Não marque actions como “CSRF-safe” só porque usam POST; Origin + SameSite + authz juntos.

```ts
// next.config.ts — só os hosts do produto
experimental: {
  serverActions: {
    allowedOrigins: ["app.example.com", "www.example.com"],
  },
},
```

Ajuste à chave de config da major do Next do repo.

## Input hostil

- FormData, JSON e **argumentos de `.bind`** são controlados pelo client (o runtime cifra closures; isso não é authz).
- Proibido: `db.update({ data: Object.fromEntries(formData) })`.
- Uploads: mime/size no servidor; não confie em `type` do browser; store fora do webroot com nomes opacos.
- Strings de path/URL: allowlist; bloqueie `javascript:`, `//evil`, redirects abertos.

## O que não fechar / não retornar

- Não capture secrets, connection strings ou tokens em closures enviadas ao client.
- Não passe do server para a action via bind: objetos enormes, user completo, flags de admin “já checadas”.
- Return/select explícito. Erros: `code` estável + `message` segura; logue `digest` internamente.

## Rate limit e abuse

- Limite por action + identidade (user id ou IP) em login, signup, invite, pagamento, mail.
- Idempotência em mutações financeiras / jobs (`Idempotency-Key` ou chave de negócio).
- Rejeite payloads grandes (`serverActions.bodySizeLimit` alinhado ao upload real).

## Checklist por action nova

- [ ] Sessão resolvida no servidor
- [ ] Policy/ownership no recurso (não só “está logado”)
- [ ] Schema allowlist; IDs revalidados
- [ ] Sem mass assignment
- [ ] Return sem campos internos
- [ ] Revalidate só do que o user pode ver
- [ ] Rate limit se for caro ou autenticável por anônimo
- [ ] `allowedOrigins` correto no deploy (www, preview, proxy)

## Anti-padrões

- Confiar em botão escondido / `disabled` na UI
- Auth só no `middleware.ts`
- Tratar action ID como segredo
- `export async function` genérica que aceita `op` + `payload` (RPC aberto)
- `dangerouslyAllowSVG` / HTML do usuário sem sanitizar em campos rich
- Desativar verificação de origem para “o proxy quebrou”

## Critérios de conclusão

- Cada action tocada autentica e autoriza no servidor
- IDs/tenant sem IDOR demonstrável
- Schema allowlist; sem spread de FormData no DB
- Origem/cookies/limites alinhados ao deploy
- Erros e returns sem vazamento
- Abuse paths (auth, mail, pagamento) com limite ou idempotência
