---
name: ruby
description: Desenvolve software em Ruby 3.4+ com Bundler, Enumerable, pattern matching, Data, keyword arguments, erros e testes (RSpec/Minitest). Use ao criar ou revisar projetos Ruby, gems, CLIs ou scripts — sem pressupor Rails.
---

# Ruby

Escreva Ruby idiomático, explícito e previsível. Metaprogramação só quando o ganho de API for claro. Web/MVC: skill `ruby-on-rails`.

## Tooling

- Ruby 3.4+ em código novo; respeite `.ruby-version` / `.tool-versions` do repo.
- Bundler: `Gemfile` + `Gemfile.lock`; `bundle exec` para binaros do projeto.
- `rake` para tarefas; RuboCop se o projeto já usa — não imponha cops novos numa feature.
- YJIT habilitado em produção quando a runtime permitir.

## Organização

```
lib/
  my_gem.rb
  my_gem/
bin/
test/   # ou spec/
my_gem.gemspec   # se gem
```

Apps (não Rails):

```
lib/my_app/
  cli.rb
  domain/
exe/my_app
```

- Arquivos/constantes no autoload path combinam (`my_app/user.rb` → `MyApp::User`).
- API pública fina na raiz do módulo; internals em submódulos.

## Linguagem

- Keyword arguments em APIs novas; evite options hash opaco (`opts = {}`) salvo compat.
- `Data.define` para value objects imutáveis; `Struct` só se o projeto já usa.
- Pattern matching (`case`/`in`) para payloads fechados (hashes/JSON conhecidos).
- `Enumerable` em vez de `for`; `each_with_object`, `filter_map`, `tally`.
- Frozen string literals: `# frozen_string_literal: true` no topo se o projeto adota.
- `||=` para memoização simples; cuidado em valores falsy (`false`/`nil`).
- Prefira `require_relative` interno; `require` para gems.

```ruby
User = Data.define(:id, :email)

def parse_status(payload)
  case payload
  in { status: "ok", id: String => id }
    [:ok, id]
  in { status: "error", message: message }
    [:error, message]
  end
end
```

## Erros

- Exceções para falhas excepcionais; retorno tagueado (`Success`/`Failure` ou `[:ok, x]`) quando o caller sempre trata.
- Hierarquia `class NotFound < Error`; não `raise "string"` em libs.
- `rescue` específico (`rescue JSON::ParserError`); nunca `rescue nil` / `rescue Exception`.
- Bordas I/O (HTTP, FS, process) resgatam e traduzem; núcleo de domínio permanece puro quando possível.

## Concorrência

- Ractors/threads só com modelo explícito; GIL ainda limita CPU-bound em threads.
- Fibers / `async` se o projeto já usa; não introduza runtime paralelo numa função isolada.
- Timeout em I/O externo; não assuma que a gem faz isso.

## Gems e APIs

- Dependências mínimas; evite gems abandonadas ou que sobrepõem stdlib (`net-http`, `json`, `pathname`).
- Version constraints conscientes (`~>`); não `>= 0`.
- Monkey patches / `refine` com escopo; sem reabrir classes core em apps.

## Testes

- RSpec ou Minitest — o do projeto.
- Teste comportamento da API pública, não internals privados.
- Fakes/stubs nas bordas (HTTP); evite mock de tudo.
- `bundle exec rspec` / `rake test` no escopo alterado.

## Anti-padrões

- `method_missing` / `define_method` para um único atalho
- Hash `{"id" => , "Id" =>}` misturando string/símbolo sem `ActiveSupport` (e mesmo assim evite)
- `eval`, `class_eval` em input de usuário
- Assumir Rails (`ActiveRecord`, `params`, `app/models`) nesta skill
- `rescue StandardError` engolindo bugs
- Mutar constantes e class variables como estado global

## Integração com outras skills

- Web MVC / API Rails: `ruby-on-rails`
- Container: `docker`

## Critérios de conclusão

- Ruby idiomático; keywords e tipos de domínio (`Data`/structs) claros
- Erros tipados ou retornos explícitos nas bordas
- Gemfile consistente; sem deps mortas novas
- Testes do caminho crítico passando
- Sem metaprogramação injustificada
