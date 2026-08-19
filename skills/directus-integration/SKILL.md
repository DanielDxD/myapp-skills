---
name: directus-integration
description: Integra aplicações com o Directus — consumo da API/SDK, autenticação por token, consultas (filters, sort, pagination, fields/deep), tratamento de erros e segurança para ler/escrever dados de collections. Use ao conectar um frontend ou backend ao Directus.
---

# Directus Integration

Conectar ao Directus é sobre tratar contratos: autenticação, consultas previsíveis e erros consistentes. Evite “chamar a API sem modelo” (strings soltas) — tipagem e estados claros tornam o consumo confiável.

## Objetivo

- Ler e escrever dados de Collections com consultas controladas.
- Manter segurança: tokens do admin nunca no cliente.
- Padronizar tratamento de erros e estados (loading/error/success).

## Regras inegociáveis

- Nunca exponha tokens admin/privileged no mobile/browser.
- Use um client único por app/processo (singleton) com base URL e interceptors.
- Sempre trate erros HTTP + mensagens do Directus; não engula falhas.
- Separe “contrato de dados” (DTOs/types) do código de UI.

## Integração via API (HTTP)

Padrão de consumo:

1. Authn → obter Access Token (ou usar token do usuário).
2. Query → `GET /items/<collection>` com:
   - `fields` (o mínimo necessário)
   - `filter` e `sort`
   - `limit` + paginação (`page`, ou cursor conforme seu padrão)
   - `deep` apenas quando precisar de relações
3. Mutação → `POST/PATCH/DELETE` com payload validado.

## Integração via SDK (recomendado)

Se o projeto usar SDK do Directus (ex.: `@directus/sdk`):

- Crie uma camada `directusClient` única com:
  - base URL
  - strategy de refresh (se necessário)
  - timeout/retry para falhas transitórias
- Centralize “mapeamento de resposta” para DTOs do seu domínio.

## Consultas e projeções

- Prefira “fields enxutos” para reduzir payload e custo.
- Evite `deep` profundo sem necessidade (latência e complexidade).
- Em listas: pagine e filtre no servidor.
- Em detalhe: use `fields` com o essencial e relações específicas.

## Tratamento de erros (contrato)

Mapeie erros para categorias:

- `401/403` → autenticação/autorizações (tratamento de sessão)
- `404` → recurso não existe
- `409` → conflito de versão/concorrência (se aplicável ao seu fluxo)
- `422` → validação (campos com mensagens)
- `5xx` → erro interno (fallback + log)

Regra: mensagens para UI devem ser humanas; detalhes internos vão para logs.

## Segurança

- Configure CORS e regras do lado Directus para o mínimo necessário.
- Para endpoints sensíveis, valide ownership/role no backend (não só no client).
- Se houver uploads, use a skill `directus-files` (não misture lógica aqui).

## Integração com outras skills

- Modelagem: `directus-data-modeling`
- Permissões: `directus-auth-permissions`
- Extensões: `directus-extensions`
- Arquivos: `directus-files`
- Deploy/migrações: `directus-migrations-import-export`

## Critérios de conclusão

- Cliente com autenticação segura e erro padronizado
- Consultas enxutas (fields) e paginação correta
- DTOs/types presentes onde fizer sentido
- Sem tokens privilegiados no client

