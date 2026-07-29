---
name: vapor-swift
description: Desenvolve APIs e backends server-side Swift com Vapor 4, Fluent, middleware, autenticação JWT/sessões, validação, testes XCTVapor e deploy Linux/Docker. Use ao criar, revisar ou depurar projetos Vapor, rotas, controllers, migrations, WebSockets ou serviços Swift no servidor.
---

# Vapor (Swift)

Trate Vapor como um backend Swift tipado e concorrente. Preferência absoluta: Vapor 4 + `async/await` + Fluent async. Não escreva código novo com `EventLoopFuture` salvo interoperabilidade com libs legadas.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **Vapor** | HTTP, routing, middleware, content, erros |
| **Fluent** | ORM, models, migrations, queries |
| **Fluent drivers** | Postgres (preferido), MySQL, SQLite |
| **JWT / sessions** | Auth API ou web |
| **Leaf** | Templates HTML quando o app servir páginas |
| **XCTVapor** | Testes de integração HTTP |

Preserve a estrutura e as libs já adotadas pelo projeto. Em apps novos, organize por domínio.

## Organização recomendada

```
Sources/App/
  configure.swift      # DB, middleware global, migrations, services
  routes.swift         # apenas app.register(collection:)
  Controllers/         # RouteCollection por recurso/domínio
  Models/              # Fluent models + migrations
  DTOs/                # request/response Codable (não exponha Model cru)
  Middleware/
  Services/            # regras de negócio / integrações
  Utilities/
Tests/AppTests/
```

- Controllers implementam `RouteCollection` e possuem prefixo, middleware e handlers.
- `routes.swift` fica declarativo: só registra collections.
- Middleware global (CORS, logging, correlation id, error mapping) em `configure.swift`.
- Middleware de auth/autorização fica no grupo da rota.

## Regras inegociáveis

- Handlers: `func ...(req: Request) async throws -> T` onde `T: AsyncResponseEncodable`.
- Decode com `req.content.decode` / `req.query.decode`; nunca confie em body bruto sem tipo.
- Valide input na borda; falhe com `Abort` e status HTTP corretos.
- Authn ≠ Authz: autenticar não autoriza ownership/role.
- Não exponha `Model` Fluent direto se houver campos internos; use DTOs.
- Migrations versionadas; nunca altere migration já aplicada em produção — crie outra.
- Um `Application` tipado: configure secrets via env, não hardcode.
- Linux é o alvo de produção; teste mentalmente portabilidade (paths, Foundation, libs C).

## Routing e controllers

```swift
struct TodosController: RouteCollection {
    func boot(routes: any RoutesBuilder) throws {
        let todos = routes.grouped("api", "v1", "todos")
        todos.get(use: index)
        todos.post(use: create)

        let protected = todos.grouped(UserToken.authenticator(), User.guardMiddleware())
        protected.group(":id") { todo in
            todo.get(use: show)
            todo.patch(use: update)
            todo.delete(use: delete)
        }
    }
}
```

- Path params: `req.parameters.get("id", as: UUID.self)`.
- Query: decode em struct `Content` dedicada (filtros, paginação).
- Agrupe por versão (`api/v1`) quando a API for pública ou multi-cliente.
- Camadas de acesso: público → autenticado → admin, cada uma adicionando middleware.

## Content, validação e erros

- Request/response: structs `Content` / `Codable` explícitas.
- Validação: `Validatable` + `validations(_:)` ou validação manual clara antes de persistir.
- Erros:
  - `Abort(.badRequest)` / `.unauthorized` / `.forbidden` / `.notFound` / `.conflict`
  - Mensagens seguras para o cliente; detalhes internos só em log
  - Middleware de erro customizado se o contrato JSON do produto exigir `{ code, message }`
- Não vaze stack traces em produção (`app.environment`).

## Fluent

- Models: `final class` / `@unchecked Sendable` conforme o padrão do projeto; fields `@Field`, `@ID`, relations `@Parent`/`@Children`/`@Siblings`.
- Sempre `try await` nas queries async.
- Evite N+1: `with(\.$relation)` / eager loading consciente.
- Índices e uniques nas migrations para hot paths e constraints reais.
- Transações: `req.db.transaction { db in ... }` para writes compostos.
- Soft delete somente com filtro consistente em todas as leituras.
- Multi-tenant: escopar **sempre** por `tenantId`/`userId` na query; nunca confiar só no client.

```swift
guard let todo = try await Todo.find(id, on: req.db) else {
    throw Abort(.notFound)
}
```

## Autenticação e autorização

- **API:** Bearer JWT ou token opaco em `Authorization`.
- **Web:** cookies/sessions com flags seguras quando servir browser.
- Encadeie authenticators idempotentes (cookie + bearer) e termine com `guardMiddleware()`.
- Policies de ownership no handler/service: `guard todo.$user.id == user.id else { throw Abort(.forbidden) }`.
- Hash de senha com Bcrypt (ou API atual do Vapor); nunca armazene plaintext.
- Rate limit em login e endpoints caros quando o risco existir.

## Middleware

Implemente `AsyncMiddleware`:

```swift
struct CorrelationIDMiddleware: AsyncMiddleware {
    func respond(to request: Request, chainingTo next: any AsyncResponder) async throws -> Response {
        let id = request.headers.first(name: "X-Request-Id") ?? UUID().uuidString
        request.logger[metadataKey: "request-id"] = .string(id)
        let response = try await next.respond(to: request)
        response.headers.replaceOrAdd(name: "X-Request-Id", value: id)
        return response
    }
}
```

- CORS no início da cadeia quando houver front separado.
- FileMiddleware só se servir estáticos de propósito.
- Não faça I/O pesado de negócio em middleware global.

## Configuração e ambiente

Em `configure.swift`:

1. Decodificar env tipado (DB URL, JWT secret, port).
2. Registrar databases e migrations.
3. Middleware global.
4. Lifecycle / services no `app.storage` ou via `Application` extensions.
5. `autoMigrate()` apenas em development/test — produção migra no pipeline de deploy.

## Testes

- Use `XCTVapor` / `testing` APIs do projeto para HTTP end-to-end.
- DB de teste isolado (SQLite em memória ou Postgres de CI).
- Cubra: happy path, 401/403, validação 400, 404, ownership.
- Evite testes que dependam de ordem global sem reset de estado.

```swift
try await app.test(.GET, "api/v1/todos") { res async in
    XCTAssertEqual(res.status, .ok)
}
```

## Performance e concorrência

- Não bloqueie o event loop com trabalho síncrono pesado; use `async` e APIs não bloqueantes.
- Tipos cruzando fronteiras de concorrência devem ser `Sendable` quando exigido.
- Connection pool do driver: ajuste só com evidência.
- Paginação com limite máximo; rejeite `limit` abusivo.
- Logs estruturados com request id; sem PII/secrets.

## Deploy

- Imagem Docker multi-stage (build macOS/CI → runtime Linux slim).
- Health route (`GET /health`) para orquestradores.
- Graceful shutdown; migrations antes de liberar tráfego.
- Secrets via env/secret manager.
- Observe build time e tamanho de imagem; cache de SPM no CI.

## WebSockets e tempo real

- Use quando o domínio exigir push real; não para CRUD simples.
- Autentique na conexão; revalide autorização por mensagem quando necessário.
- Backpressure e timeouts conscientes; teste reconexão no cliente.

## Integração com o restante do toolkit

- Contratos HTTP: alinhe com `api-design`.
- Authz profunda: `authentication-authorization`.
- MVVM no client SwiftUI/iOS: `mvvm-architecture` + `swiftui-development`.
- Go-live: `production-readiness`.

## Anti-padrões

- Lógica de negócio inteira em `routes.swift`
- `EventLoopFuture` em código novo sem necessidade
- Retornar Model Fluent com password hash / tokens
- Migration editada após deploy
- Auth só no cliente iOS/web
- `try!` / force unwrap em caminhos de request
- SQLite em produção multi-instância sem plano
- Ignorar falhas de decode com defaults silenciosos perigosos

## Fluxo ao implementar uma feature

1. Inspecione `Package.swift`, `configure.swift` e controllers existentes.
2. Modele DTO + (se preciso) Fluent model/migration.
3. Service com a regra de negócio.
4. Controller `RouteCollection` com authz e status corretos.
5. Testes XCTVapor dos casos de risco.
6. Atualize migrations/env/docs só do que mudou.

## Critérios de conclusão

- Rotas registradas via `RouteCollection`, tipadas e versionadas quando aplicável
- Handlers 100% async/await no código novo
- Input validado; erros HTTP corretos
- Authn/authz no servidor com ownership coberto
- DTOs sem vazamento de campos internos
- Migration adicionada (não reescrita) se o schema mudou
- Testes dos fluxos críticos passando
- Config por env; pronto para Linux/Docker se o escopo incluir deploy
