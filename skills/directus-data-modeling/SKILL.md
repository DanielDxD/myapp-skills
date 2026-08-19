---
name: directus-data-modeling
description: Modela dados no Directus — Collections, Fields, Relations, constraints, system fields, drafts e versionamento. Use ao estruturar o schema do projeto, revisar naming/relacionamentos ou preparar migrações e integrações.
---

# Directus Data Modeling

Modelagem no Directus define o “contrato de verdade” do seu produto: o que pode ser lido/escrito, como os relacionamentos funcionam e como o sistema evolui sem quebrar integrações.

## Princípios

- Modelar por domínio (bounded context), não por tela.
- Relações são comportamento; pense em consultas reais: listagens, detalhes e filtros.
- Preserve estabilidade: evite renomear/reestruturar sem estratégia de migração.

## Collections

Ao criar uma Collection:

- Defina a finalidade (read/write) e o tipo de uso: catálogo, conteúdo, transações, configurações.
- Use nomes consistentes: plural e sem abreviações obscuras.
- Tenha clareza de versionamento: se mudanças quebram consumidores, trate como versionado.

## Fields (tipagem e intenção)

- Tipos corretos importam: strings, numbers, booleans, date/timestamps, JSON, enums.
- Para campos com domínio fechado, prefira tipos/constraints que limitem valores.
- Evite “campos genéricos” (ex.: `metadata` como JSON solto) sem necessidade.

### Validação

- Valide formato e limites no Directus quando isso reduzir trabalho no client.
- Mensagens para o usuário devem ser claras; mensagens para logs devem ser técnicas.

## Relations

Diretrizes:

- One-to-many: quando um “pai” tem muitos “filhos” e você consulta por pai.
- Many-to-many: quando o relacionamento tem semântica própria (ou precisa de tabelas de junção com atributos).
- Polimórfico: só se fizer sentido (caso contrário, separe collections).

Regras:

- Nomeie relations com intenção (`trip.driver`, `trip.passengers`) para facilitar leitura no codegen/DTOs.
- Evite relação circular desnecessária.

## System fields e workflow

Decida e documente o uso de:

- timestamps (`created`, `updated` quando aplicável ao seu projeto)
- `status` / publish flow (draft → published) se o conteúdo precisar de revisão
- user attribution (fields de autoria, se o projeto usar)

Se o projeto tem rascunho:

- defina o que é visível para quais roles (rascunho só interno, publicado para público).

## Versionamento e evolução do schema

- Mudanças compatíveis: adicione campos/relations opcionais primeiro.
- Mudanças incompatíveis: crie nova Collection/coluna versionada ou faça migração com fallback.
- Mantenha “fonte de verdade” no schema exportado/na pasta de migrações (veja `directus-migrations-import-export`).

## Critérios de conclusão

- Naming e tipos do schema refletem domínio real
- Relations suportam queries reais (list/detail/search) sem N+1 improvisado no client
- Regras de draft/status são coerentes com roles e políticas
- Evolução do schema tem estratégia (compatível → versionado → migrado)

