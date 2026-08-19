---
name: directus-files
description: Gerencia arquivos no Directus — upload, permissões, assets/transformações e consumo na UI. Use ao integrar mídia (imagens, vídeos, PDFs) ao CMS, controlar acesso e preparar cache/CDN para o app.
---

# Directus Files (Uploads & Assets)

Arquivos são o tipo de dado que mais costuma gerar custos e riscos (privacidade, caching e performance). Trate como domínio: permissões e pipelines claros.

## Objetivo

- Upload confiável e com limites.
- Acesso controlado (público vs privado) alinhado a políticas/roles.
- Consumo previsível na UI (URLs estáveis, cache, fallback).

## Permissões e privacidade

Regra de ouro:

- Se o arquivo for privado, não entregue URL “aberta” sem controle.

Checklist:

- Verifique permissões de leitura para `directus_files` (ou collection de arquivos) e para paths/asset delivery do seu projeto.
- Se a app exige proteção por usuário: use mecanismos compatíveis com tokens/sessões (o padrão do seu deployment).

## Uploads

- Defina limites de tamanho/tipo no Directus e trate erro no app.
- Faça uploads em fluxos claros:
  - pré-validação (tamanho, mime, extensão) no client
  - upload
  - confirmação do item (criar/update do item que referencia o arquivo)
- Para consistência: se a gravação do item falhar após upload, defina cleanup ou política de retenção.

## Asset serving e caching

- Prefira URLs/caminhos que suportem caching em CDN.
- Configure headers conforme o tipo (cache long para imagens estáticas; cache curto para assets dinâmicos).
- Em UI, use lazy loading quando fizer sentido, mas garanta que o conteúdo acima da dobra (LCP) não fique “sem mídia”.

## Transformações e variantes

Se o projeto usar transform/derivations:

- Padronize tamanhos e formatos (ex.: WebP/AVIF para imagens).
- Garanta que o app consome a variante correta por contexto (list vs hero).

## Integração com o app

- No app: trate estados `uploading`, `failed`, `retry`.
- Exponha ao UI apenas:
  - `id`/reference do arquivo
  - URL final (ou builder de URL) pronta para consumo
  - metadados mínimos (nome, tamanho, mime) se necessários

## Anti-padrões

- URLs de ativos expostas para arquivos privados
- Upload sem tratamento de tamanho e mime (pode virar vetor de ataque)
- Carregar mídia sem considerar LCP/CLS
- Referenciar arquivos sem confirmar persistência/associação ao item

## Critérios de conclusão

- Upload e associação ao item são consistentes
- Permissões privadas funcionam (usuário não autorizado não acessa)
- UI consome assets com cache e fallback

