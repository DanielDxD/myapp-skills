---
name: directus-auth-permissions
description: Administra autenticação e permissões no Directus — roles, granular permissions, policies de acesso por usuário e restrições de escrita/leitura. Use ao configurar segurança do CMS, revisar RBAC, evitar vazamento de dados e alinhar integrações do app às regras do Directus.
---

# Directus Auth & Permissions

Segurança no Directus é combinar:

1. quem é o usuário (auth),
2. o que ele pode fazer (permissions),
3. quais linhas/itens ele enxerga (policies/filters).

Essa skill evita “deixar tudo liberado” por conveniência e melhora previsibilidade entre UI e CMS.

## Authn (quem é você)

- Use o fluxo de login que o seu projeto adotou (email/senha, SSO, tokens existentes).
- Garanta que o client trabalha com tokens de usuário e trata expiração/refresh corretamente.
- Nunca use credenciais admin no front.

## RBAC (o que pode fazer)

Ao configurar Roles:

- Separe por nível de acesso: `public`, `authenticated`, `editor`, `admin` (ajuste aos seus papéis).
- Para cada Collection, defina permissões de:
  - read (listar/detalhar)
  - create
  - update
  - delete
- Evite “create/update globais” para collections que representam dados críticos.

## Policies (o que é visível)

Quando o app exige acesso por ownership/tenant:

- Use policies para restringir leitura e escrita por `user`/`role`/`tenant_id`/escopo.
- Garanta consistência: se o UI não mostra, o backend também deve bloquear.
- Para relações (deep reads), verifique se as policies cobrem também as consultas relacionadas.

## Boas práticas para evitar vazamento

- Não dependa apenas de filtros no client.
- Aplique authz na Collection e, quando necessário, em actions/endpoint custom (extensions).
- Crie testes manuais de segurança:
  - usuário A não vê itens de B
  - usuário A não consegue atualizar campos proibidos
  - guest não obtém detalhes via `fields`/`deep`

## Integração com o app

- Mapeie status de erro do Directus para UX:
  - `401/403` → prompt/login/redirect
  - `404` → recurso inexistente ou não autorizado (não vaze existência)
  - validação → mensagens por campo
- Mantenha uma camada de “Data Access” no app que não assume permissões.

## Anti-padrões

- Role `admin` no app mobile
- Permitir `read` global para collections com dados sensíveis
- Policies frágeis baseadas em um campo sem índice ou sem validação de escopo
- `deep` sem conferir o que o usuário realmente pode ler

## Critérios de conclusão

- Roles e permissões justificadas por fluxos do produto
- Policies implementadas para ownership/tenant quando exigido
- A app demonstra comportamento consistente para 401/403/404
- Não há caminhos de vazamento via endpoints/fields/deep

