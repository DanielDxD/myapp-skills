---
name: phoenix
description: Desenvolve apps Phoenix 1.8 com contexts, Ecto, LiveView, scopes de authz, PubSub, generators e APIs JSON. Use ao criar ou revisar projetos Phoenix, routers, LiveViews, controllers, migrations, Channels ou phx.gen.*.
---

# Phoenix 1.8

Trate Phoenix como a borda HTTP/realtime sobre OTP. Linguagem, processos e Mix: skill `elixir`. Código novo: Phoenix 1.8 — **scopes** para acesso a dados, LiveView quando a UI for server-rendered, controllers JSON quando o cliente for separado.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **Phoenix 1.8** | Endpoint, router, controllers, LiveView |
| **Ecto** | Schemas, changesets, migrations, Repo |
| **Postgrex** | Postgres (preferido) |
| **PubSub** | Fan-out / LiveView updates |
| **Swoosh** | E-mail (auth magic link incluso) |
| **Bandit** | HTTP server (default recente) |

Preserve o que `mix phx.new` / o repo já geraram (Tailwind, esbuild, `core_components`).

## Organização

```
lib/
  my_app/                 # contexts (domínio)
    accounts/
    blog/
  my_app_web/
    router.ex
    endpoint.ex
    controllers/
    live/
    components/
    plugs/
priv/repo/migrations/
test/
```

- Contexts (`MyApp.Blog`) são a API de domínio: `list_posts(scope)`, `get_post!(scope, id)`.
- `MyAppWeb` só HTTP/LiveView: plug, params, render. Sem SQL no LiveView.
- `router.ex` declara pipelines e `live_session`; não acumule lógica.

## Regras inegociáveis

- Authn ≠ Authz. Scope (user, org, tenant) entra em **toda** função de context que lê/escreve dados do usuário.
- Generators (`phx.gen.live`, `html`, `json`) já enfiçam scope — não remova o parâmetro “para simplificar”.
- Changesets validam na borda; nunca `Repo.insert` com map cru de params.
- Migrations: nunca edite migration já aplicada em produção — crie outra.
- Não exponha schema Ecto com campos internos (hash de senha, tokens) no JSON/LiveView — use structs/JSON views.
- Secrets via runtime config / env (`config/runtime.exs`), não hardcode.
- `force_ssl` e cookies seguros em prod.

## Scopes (1.8)

O scope centraliza o contexto da request (`current_user`, org, …) e filtra queries/PubSub.

```elixir
def list_posts(%Scope{} = scope) do
  Post
  |> where(user_id: ^scope.user.id)
  |> Repo.all()
end

def get_post!(%Scope{} = scope, id) do
  get_post_query(scope)
  |> Repo.get!(id)
end
```

- LiveView: rotas autenticadas dentro de `live_session` com `on_mount` que define `current_scope`.
- PubSub: tópicos já scoped (`org:#{id}`) para não vazar eventos cross-tenant.
- `mix phx.gen.auth` define o scope default e magic link — não troque por senha só “porque sempre foi assim” sem pedido.

## Router e pipelines

```elixir
pipeline :browser do
  plug :accepts, ["html"]
  plug :fetch_session
  plug :fetch_live_flash
  plug :put_root_layout, html: {MyAppWeb.Layouts, :root}
  plug :protect_from_forgery
  plug :put_secure_browser_headers
end

pipeline :api do
  plug :accepts, ["json"]
end

scope "/", MyAppWeb do
  pipe_through [:browser, :require_authenticated_user]

  live_session :authenticated,
    on_mount: [{MyAppWeb.UserAuth, :ensure_authenticated}] do
    live "/posts", PostLive.Index, :index
  end
end
```

- Browser vs API em pipelines separados.
- Plugs de auth no pipeline; autorização fina no context (scope).

## LiveView

- `mount` / `handle_params` para carregar dados via context + scope.
- Eventos: `handle_event` magro → context → `assign` / stream.
- Streams para listas; evite mandar collections gigantes no socket.
- `live_redirect` / patches para navegação; CSRF já está no form Phoenix.
- JS hooks só para APIs do browser (clipboard, mapa); não para regra de negócio.

## Ecto

- Schema ≠ API pública. Changeset por ação (`create_changeset`, `update_changeset`).
- `Ecto.Multi` / `Repo.transaction` para writes compostos.
- Evite N+1: `preload` consciente ou `join`.
- Constraints no banco (unique, FK) + changeset (`unique_constraint`).
- Soft delete só com filtro consistente em todas as queries do context.

## JSON / controllers

- `action_fallback` e views/JSON estáveis `{data | errors}`.
- Params: `params["id"]` + changeset; strong-ish via embedded schema ou changeset.
- Status HTTP corretos (`404`, `401`, `403`, `422`). Contratos: skill `api-design`.

## Testes

- `ConnCase` / `LiveViewTest` / `DataCase`.
- Cubra: happy path, 401/403, changeset inválido, scope (user A não lê recurso de B).
- Sandbox Ecto; não compartilhe DB real entre testes async sem setup.

## Anti-padrões

- `Repo.get` sem scope em recurso de usuário
- SQL/Ecto no `handle_event`
- Migration reescrita após deploy
- LiveView para API mobile (prefira JSON)
- Assinar PubSub global e filtrar “no cliente”
- `String.to_atom` em params

## Integração com outras skills

- Linguagem/OTP: `elixir`
- Contratos HTTP: `api-design`
- Authz profunda: `authentication-authorization`
- Container: `docker`
- Go-live: `production-readiness`

## Critérios de conclusão

- Contexts recebem scope; queries filtradas
- LiveView ou JSON alinhado ao cliente real
- Changesets + migration nova (não editada) se o schema mudou
- Authn no pipeline; authz no domínio
- Testes de ownership/403 passando
- Config por env; SSL/cookies ok para o ambiente alvo
