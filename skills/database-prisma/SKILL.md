---
name: database-prisma
description: Modelagem relacional com Prisma, migrations, índices, transações, queries eficientes e integração TypeScript. Use ao criar schemas, migrar dados, otimizar queries ou integrar Prisma em APIs e Server Components.
---

# Database Prisma

O schema é o contrato. Migrations são a história. Queries devem ser previsíveis sob carga.

## Modelagem

- Modele entidades e relacionamentos no `schema.prisma` com nomes claros.
- Use `@id`, `@unique`, `@relation` e onDelete conscientes (`Cascade` só quando correto).
- Prefira enums do Prisma para conjuntos fechados.
- Timestamps: `createdAt` / `updatedAt` por padrão em entidades mutáveis.
- Normalize o suficiente; desnormalize só com motivo de leitura medido.

## Migrations

- Sempre gere migration (`prisma migrate`) em vez de só `db push` em produção.
- Revisar SQL gerado antes de aplicar.
- Migrations devem ser forward-safe; evite rewrites destrutivos sem plano de dados.
- Em deploy: migrate → release app (ou estratégia documentada do projeto).

## Queries eficientes

- Selecione só campos necessários (`select` / `include` enxuto).
- Evite N+1: use `include`/`select` adequados ou queries em lote.
- Paginação com cursor ou skip/take limitado.
- Índices para filtros/ordenação frequentes (`@@index`, unique compostos).
- Transações (`$transaction`) para writes atômicos multi-tabela.

## Integração

- Um PrismaClient por processo (singleton em dev hot-reload).
- Não exponha o client direto à UI; encapsule em repositories/services.
- Valide input antes de montar where/data.
- Trate erros Prisma (`P2002` unique, `P2025` not found) como domínio.

## Soft delete e multi-tenant

- Soft delete: filtro consistente em todas as leituras.
- Multi-tenant: sempre escopar por `tenantId`/`orgId` nas queries; nunca confiar só no client.

## Anti-padrões

- `findMany` sem limite
- `include` profundo “por precaução”
- Lógica de negócio espalhada em callbacks de middleware sem clareza
- Alterar produção com `db push`
- Índices ausentes em foreign keys filtradas

## Critérios de conclusão

- Schema e relations coerentes
- Migration revisada
- Queries sem N+1 óbvio
- Índices para hot paths
- Erros de DB mapeados para a API/domínio
