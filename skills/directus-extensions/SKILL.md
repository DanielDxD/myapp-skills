---
name: directus-extensions
description: Cria e revisa extensões no Directus — hooks, flows e lógica customizada — para automatizar eventos, transformar dados e expor comportamentos específicos. Use ao integrar regras de negócio no CMS, adicionar validação e efeitos colaterais confiáveis ou auditar idempotência e falhas.
---

# Directus Extensions (Hooks & Flows)

Extensões no Directus são onde o “CMS vira produto”: validações, transformações e automações acontecem junto do lifecycle dos dados.

Use esta skill para definir limites: o que fica no schema/policies e o que vira extensão. Extensão ruim vira risco operacional.

## Decidir onde implementar

- **Schema/fields/constraints**: validações estruturais, tipos, limites.
- **Auth/permissions/policies**: quem vê/edita o quê.
- **Hooks**: efeitos colaterais imediatos e consistentes no write.
- **Flows**: automações mais longas, disparadas por eventos, com re-tentativas/controle.
- **Endpoints custom**: quando a API precisa de contrato próprio.

## Hooks

Regras:

- Idempotência: a mesma escrita não deve duplicar efeitos.
- Não faça trabalho pesado no request path; se precisar, use “enqueue”/side effect rápido.
- Valide entrada e falhas com mensagens úteis para o usuário quando fizer sentido.
- Sempre trate erros (não engula): falha deve retornar status coerente.

## Flows

Regras:

- Defina o trigger com clareza (evento, coleção, status).
- Gere uma “correlation id” para rastrear logs e retries.
- Faça retry apenas para erros transitórios e com backoff.
- Proteja contra eventos duplicados (eventual delivery).

## Transformações e normalização

- Transformações que devem ser determinísticas ficam nos hooks.
- Transformações que variam por contexto (ex.: tenant) devem derivar de dados do item + config.
- Não armazene segredos em extensão. Use env/secret management do seu setup.

## Operação e segurança

- Logue apenas o necessário; evite PII completa.
- Teste cenários:
  - update simples
  - criação com campos obrigatórios
  - falha em etapa externa
  - retries (duplicação)
- Respeite policies: extensão não “escapa” de authz.

## Integração com o restante

- Dados: `directus-data-modeling`
- Segurança: `directus-auth-permissions`
- Consumo no app: `directus-integration`
- Arquivos/Assets: `directus-files`

## Critérios de conclusão

- Hooks/flows são determinísticos e idempotentes
- Falhas retornam comportamento coerente e não deixam dados inconsistentes
- Logs rastreáveis e sem vazamento de segredos/PII

