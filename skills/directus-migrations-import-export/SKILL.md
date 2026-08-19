---
name: directus-migrations-import-export
description: Gerencia migrações e snapshots no Directus — export/import de schema e dados, versionamento do estado do CMS entre ambientes e pipelines seguros. Use ao preparar dev/staging/prod, reduzir risco de drift e auditar mudanças de schema e conteúdo.
---

# Directus Migrations, Import & Export

O Directus muda seu schema e seu conteúdo com o tempo. Para manter previsibilidade entre ambientes, versionar migrações e snapshots é essencial.

## Objetivo

- Evitar drift de schema entre dev/staging/prod.
- Permitir reprodutibilidade de conteúdo (quando necessário).
- Garantir que mudanças destrutivas sejam percebidas antes de aplicar.

## Migrações (schema)

Regras:

- Trate schema como código: migrações/snapshots versionados no repositório.
- Aplique migrações na ordem correta e com revisão (PR) quando mudar coleções/fields/relations/constraints.
- Não “edite no prod” para resolver; crie migração e rode no pipeline.
- Mantenha mudanças compatíveis primeiro: adicione → migre conteúdo → remova/transforme em etapas futuras.

## Import/Export de dados

Use quando:

- Seed inicial do projeto (ex.: categorias padrão)
- Reset controlado de conteúdo em staging
- Migração de dados entre versões do schema

Regras:

- Exporte o mínimo necessário para seeds: dados essenciais e referências.
- Ao importar, valide dependências: relations/foreign keys precisam existir no destino.
- Evite sobrescrever produção com snapshots genéricos; mantenha exports por ambiente.

## Estratégia de ambientes

- Dev: maior velocidade, mas sempre com snapshots/migrações em PR.
- Staging: validação completa (auth, queries, UI).
- Prod: mudanças com janela e rollback/forward strategy documentada.

## Segurança

- Dados exportados podem conter PII: trate com cuidado (separar storage, permissões, não logar).
- Não comite segredos ou tokens em exports.

## Integração com o toolkit

- Modelagem: `directus-data-modeling`
- Permissões: `directus-auth-permissions`
- Consumo: `directus-integration`
- Extensões: `directus-extensions`
- Arquivos: `directus-files`

## Critérios de conclusão

- Mudanças de schema seguem um caminho reprodutível (export/snapshot + migração)
- Import/seed é controlado e documentado por ambiente
- Pipeline reduz drift e facilita rollback quando houver incidente

