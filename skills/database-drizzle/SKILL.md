---
name: database-drizzle
description: Modelagem relacional com Drizzle ORM, schema TypeScript, drizzle-kit migrations, índices, transações, queries eficientes e integração tipada. Use ao criar schemas, migrar dados, otimizar queries ou integrar drizzle-orm em APIs e Server Components.
---

# Database Drizzle

O schema TypeScript é o contrato. Migrations do drizzle-kit são a história. Queries devem ser previsíveis sob carga.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **drizzle-orm** | Queries tipadas, relations, SQL builder |
| **drizzle-kit** | `generate` / `migrate` / `push` / studio |
| **Postgres** | Default (pg / postgres.js / neon / etc. do projeto) |
| **Schema TS** | Tabelas, enums, indexes em arquivos versionados |

Preserve drivers e layout já adotados pelo projeto. Em apps novos, preferir Postgres.

## Organização recomendada

```
src/db/
  schema/          # tables + relations por domínio
  index.ts         # client exportado
  migrations/      # SQL gerado pelo kit (se versionado no repo)
drizzle.config.ts
```

- Um client por processo (singleton em dev hot-reload).
- Não exponha o client à UI; encapsule em repositories/services.
- Schema é fonte da verdade; não edite SQL de migration já aplicada — gere outra.

## Modelagem

```ts
import { pgTable, uuid, text, timestamp, index } from "drizzle-orm/pg-core";
import { relations } from "drizzle-orm";

export const users = pgTable("users", {
  id: uuid("id").defaultRandom().primaryKey(),
  email: text("email").notNull().unique(),
  createdAt: timestamp("created_at", { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp("updated_at", { withTimezone: true }).defaultNow().notNull(),
});

export const posts = pgTable(
  "posts",
  {
    id: uuid("id").defaultRandom().primaryKey(),
    authorId: uuid("author_id")
      .notNull()
      .references(() => users.id, { onDelete: "cascade" }),
    title: text("title").notNull(),
  },
  (t) => [index("posts_author_id_idx").on(t.authorId)],
);

export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
}));
```

- Nomes de tabela/coluna claros; `onDelete` consciente (`cascade` só quando correto).
- Prefira enums do Postgres/`pgEnum` para conjuntos fechados.
- Timestamps `createdAt` / `updatedAt` em entidades mutáveis.
- Normalize o suficiente; desnormalize só com motivo de leitura medido.
- Declare `relations()` quando for usar a API relacional (`db.query`).

## Migrations

- Produção: `drizzle-kit generate` → revisar SQL → `migrate` (ou pipeline equivalente).
- Evite `push` em produção; reserve para protótipo/local explícito.
- Migrations forward-safe; rewrites destrutivos só com plano de dados.
- Em deploy: migrate → release app (ou estratégia documentada do projeto).

## Queries eficientes

- Selecione só colunas necessárias (`select({ ... })`).
- Evite N+1: `with` em `db.query` ou joins explícitos / batch.
- Paginação com cursor ou `limit`/`offset` limitado; rejeite page size abusivo.
- Índices para filtros/ordenação frequentes e FKs quentes.
- Transações: `db.transaction(async (tx) => { ... })` para writes multi-tabela.
- Prepared statements / pools conforme o driver do projeto.

```ts
const post = await db.query.posts.findFirst({
  where: eq(posts.id, id),
  with: { author: true },
  columns: { id: true, title: true },
});
```

## Integração

- Valide input (Zod etc.) antes de montar `where` / `values` / `set`.
- Trate erros do driver (unique violation, FK) como domínio (`409`, `404`, etc.).
- Inferência: use tipos do schema (`typeof users.$inferSelect` / `$inferInsert`).
- Server Components / route handlers: abra conexão via singleton; feche no shutdown se o runtime exigir.

## Soft delete e multi-tenant

- Soft delete: filtro consistente (`isNull(deletedAt)`) em todas as leituras.
- Multi-tenant: sempre escopar por `tenantId`/`orgId` nas queries; nunca confiar só no client.

## Anti-padrões

- `select()` sem limite em listas
- `with` profundo “por precaução”
- Lógica de negócio espalhada em helpers SQL opacos
- Alterar produção com `drizzle-kit push`
- Índices ausentes em foreign keys filtradas
- Expor row crua com campos internos (hash, tokens)

## Critérios de conclusão

- Schema e relations coerentes
- Migration gerada e SQL revisado
- Queries sem N+1 óbvio
- Índices para hot paths
- Erros de DB mapeados para a API/domínio
- Client encapsulado; tipos derivados do schema
