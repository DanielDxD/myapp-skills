---
name: ruby-on-rails
description: Desenvolve apps Ruby on Rails 8.1 com MVC, Active Record, Hotwire ou API mode, jobs Solid Queue, credentials, migrations e Kamal. Use ao criar ou revisar projetos Rails, controllers, models, jobs, mailers, Turbo/Stimulus ou rotas.
---

# Ruby on Rails 8.1

Convenção sobre configuração. Código novo: **Rails 8.1** + Ruby 3.4 quando o projeto permitir. Linguagem pura: skill `ruby`. Preserve Rails 7.x existente; não “upgrade de passagem”.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **Rails 8.1** | Router, MVC, generators |
| **Active Record** | Persistência, migrations |
| **Propshaft** | Assets (default 8) |
| **Hotwire** | Turbo + Stimulus em apps HTML |
| **Solid Queue / Cache / Cable** | Jobs, cache, pubsub DB-backed |
| **Kamal** | Deploy (se o escopo incluir) |
| **Credentials / ENV** | Segredos |

Front separado (SPA/mobile): `--api` ou `config.api_only`. Não force Turbo numa API JSON.

## Organização

```
app/
  controllers/
  models/
  views/           # omitido em api_only
  jobs/
  mailers/
  policies/        # se o projeto usa Pundit/ActionPolicy
  services/        # POROs de caso de uso (quando o model engordar)
config/
db/migrate/
test/  # ou spec/
```

- Fat models com cuidado: validações e invariantes no model; orquestrações multi-model em service/PORO.
- `app/services` não é lixeira — um objeto por caso de uso.
- Concerns só para duplicação real, não para esconder god objects.

## Regras inegociáveis

- Strong parameters em todo input de controller.
- Authz no servidor (Pundit/ActionPolicy/scopes); nunca só esconder botão no view.
- SQL: query methods / binds; nunca interpolar input em string SQL.
- Migrations reversíveis quando possível; **não edite** migration já aplicada em produção.
- Secrets: `Rails.application.credentials` ou ENV — nunca commit de chaves.
- CSRF ligado em HTML; APIs com token/session explícitos.
- Jobs idempotentes; argumentos serializáveis (IDs, não AR objects gigantes).

## Controllers e rotas

```ruby
class PostsController < ApplicationController
  before_action :set_post, only: %i[show update destroy]

  def create
    @post = Current.user.posts.build(post_params)
    if @post.save
      redirect_to @post
    else
      render :new, status: :unprocessable_content
    end
  end

  private

  def set_post
    @post = Current.user.posts.find(params[:id])
  end

  def post_params
    params.require(:post).permit(:title, :body)
  end
end
```

- Scoping por usuário/tenant no `find` — `Post.find(params[:id])` em recurso privado é IDOR.
- Status HTTP corretos (`404`, `401`, `403`, `422`).
- JSON: `jbuilder` / `alba` / serializers do projeto; não `as_json` solto com colunas internas.
- `Current` attributes para request store (user, request_id) — reset automático.

## Active Record

- Validações + constraints no banco (unique, FK, check).
- `has_many` / `belongs_to` com `inverse_of` quando ciclos atrapalharem.
- N+1: `includes` / `preload` / `strict_loading` em dev.
- Enums, `normalizes`, `composed_of` quando o domínio pedir.
- Transactions para writes compostos.
- Queries: scopes encadeáveis; `find_by!` quando a ausência é erro.

## Hotwire (HTML)

- Turbo Drive/Frames/Streams para UX sem SPA.
- Stimulus para JS mínimo (clipboard, toggle) — regra de negócio fica no server.
- `turbo_stream` responses pontuais; não substitua o app por SPA React sem pedido.
- Formulários: `form_with`, erros no model, `status: :unprocessable_content`.

## Jobs, mailers, realtime

- Solid Queue (default 8) salvo o repo já usar Sidekiq/GoodJob — não troque o backend numa feature.
- `deliver_later` em mailers; previews em `ActionMailer::Preview`.
- Solid Cable / Solid Cache quando o 8 já configurou; Redis só se já for padrão.

## Front e API

- HTML+Hotwire: views + partials + helpers magros.
- API mode: versionamento se público (`/api/v1`); auth token/session documentada.
- Contratos: skill `api-design`.

## Testes

- Fixtures ou FactoryBot — o do projeto.
- Controller/request specs para authz e strong params.
- Model specs para invariantes; system tests (Capybara) para o fluxo Hotwire crítico.
- `bin/rails test` / `bundle exec rspec` no escopo.

## Deploy

- Kamal se o repositório já tiver `config/deploy.yml` ou o usuário pedir deploy.
- `RAILS_MASTER_KEY` / credentials por ambiente.
- Migrations no release (`kamal` / pipeline), não `db:migrate` manual esquecido.
- Imagem: skill `docker`.

## Anti-padrões

- `Post.find(params[:id])` sem ownership
- Callbacks AR em cascata (emails, HTTP, jobs) invisíveis
- `update_all` / `delete_all` sem scope
- Editar migration velha
- Introduzir Sidekiq + Solid Queue juntos
- Assumir Webpacker / Sprockets clássico em app 8 (Propshaft)
- Lógica de billing/authz só no Stimulus

## Integração com outras skills

- Linguagem: `ruby`
- HTTP: `api-design`
- Authz: `authentication-authorization`
- Imagem: `docker`
- Compose local: `docker-compose`
- Go-live: `production-readiness`

## Critérios de conclusão

- Strong params + authz/scoping no servidor
- Migration nova se o schema mudou
- Jobs/mailers async quando I/O externo
- HTML via Hotwire **ou** JSON API — alinhado ao cliente
- Testes do fluxo e do IDOR negativo
- Secrets fora do git
