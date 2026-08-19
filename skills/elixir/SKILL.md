---
name: elixir
description: Desenvolve software em Elixir 1.20+ com OTP, Mix, pattern matching, GenServer, Supervisors, typespecs e ExUnit. Use ao criar ou revisar projetos Elixir, CLIs, workers, libs Hex, concorrência OTP — sem pressupor Phoenix ou Ecto.
---

# Elixir

Escreva Elixir idiomático e concorrente. Processos são leves: modele falhas com supervisores, não com `try` em todo lugar. Phoenix/Ecto ficam na skill `phoenix`.

## Tooling

- Mix: `mix new`, umbrelas quando houver apps múltiplos.
- Hex para deps; pin consciente em `mix.exs` / `mix.lock`.
- `mix test`, `mix format`, `mix credo` (se o projeto usa), `mix dialyzer` quando houver Dialyzer.
- Elixir 1.20+ / OTP 27+ em código novo; respeite `.tool-versions` / `elixir` do repo.
- Compilador e sistema de tipos do 1.20 avisam — trate warnings novos no escopo alterado.

## Organização

```
lib/
  my_app.ex            # API pública fina
  my_app/
    application.ex     # supervision tree
    ...
mix.exs
test/
  test_helper.exs
```

Umbrella:

```
apps/
  my_app_domain/
  my_app_worker/
```

- Módulos por domínio; `MyApp` reexporta o que for API estável.
- Separe código puro (testável sem processos) de processos OTP.
- Visibilidade: funções privadas `defp`; não exponha internals no `@moduledoc` público.

## Linguagem

- Pattern matching e guards na borda da função, não `if` aninhado.
- Pipe `|>` para transformações; quebre o pipe se a leitura piorar.
- Structs e maps com chaves átomo conhecidas; `String.to_existing_atom/1` se vier de fora.
- `@spec` nas APIs públicas; `@type` / `@typedoc` para tipos de domínio.
- `with` para pipelines de `{:ok, _}` / `{:error, _}`; não silencie o error clause.
- Recursão + accumulators; `Enum`/`Stream` para coleções. Tail recursion quando o volume exigir.

```elixir
def fetch_user(id) when is_binary(id) do
  with {:ok, user} <- Repo.fetch(id),
       :active <- user.status do
    {:ok, user}
  else
    :inactive -> {:error, :inactive}
    {:error, _} = err -> err
  end
end
```

## OTP

- **Application** sobe o supervision tree, não trabalho de negócio.
- **Supervisor** (`:one_for_one` default): filhos `restart: :permanent` salvo transientes reais.
- **GenServer**: estado mínimo; `handle_call` para request/response, `handle_cast` fire-and-forget, `handle_info` mensagens externas.
- Timeouts explícitos em `call`; nunca `GenServer.call(pid, msg, :infinity)` sem justificativa.
- **Task** / `Task.Supervisor` para trabalho pontual; `async_nolink` + `yield` quando o caller não pode morrer junto.
- **Registry** / **DynamicSupervisor** para instâncias por id (ex.: sessão, socket).
- Mensagens: tuples tagueadas `{:event, payload}`; não envie maps opacos sem contrato.

Estado vive no processo dono. Evite ETS até haver evidência (hot path, fan-out). Evite Agent para domínio — prefira GenServer nomeado.

## Erros e processos

- Let it crash **dentro** da árvore de supervisão: bugs e invariantes.
- Erros de input/I/O esperado: `{:error, reason}` na API, não raise.
- `raise` / `throw` só para invariantes de programação.
- `try/rescue` nas bordas (JSON, HTTP, disco), não no núcleo do domínio.
- Telemetria (`:telemetry`) em operações importantes; logs estruturados, sem secrets.

## Concorrência

- Isolamento: um processo, um pedaço de estado.
- Backpressure: mailboxes não são infinitas — limite, drop ou shed load.
- Não compartilhe estado mutável em `:persistent_term` para dados que mudam.
- Timeouts e shutdown (`Supervisor.stop`, `System.stop`) conscientes.

## Testes

- ExUnit; `async: true` quando não houver estado global.
- `start_supervised!/1` para processos — o ExUnit encerra sozinho.
- Mocks: prefira fakes/ports (behaviours) a mock pesado de módulos.
- `mix test` no escopo alterado deve passar.

## Anti-padrões

- GenServer que faz de tudo (HTTP + DB + timer + cache)
- `String.to_atom/1` em input externo
- `Enum` em hot path gigante onde `Stream` ou recursão caberia
- `Process.sleep` em produção para “esperar o outro processo”
- Supervisors vazios ou `restart: :temporary` em serviços críticos
- Assumir Phoenix (`MyAppWeb`, LiveView, Ecto.Repo) nesta skill

## Integração com outras skills

- Web / LiveView / Ecto: `phoenix`
- Contratos HTTP: `api-design`
- Deploy em container: `docker`

## Critérios de conclusão

- Código formatado; warnings do compilador tratados no escopo
- APIs com `{:ok, _}` / `{:error, _}` ou raise justificado
- Processos sob supervisor; timeouts explícitos
- Typespecs nas funções públicas tocadas
- Testes ExUnit dos caminhos críticos passando
