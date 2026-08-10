---
name: golang-apis
description: Desenvolve APIs HTTP em Go com Gin — rotas, middleware, binding, validação, erros JSON, authz, paginação e organização por domínio. Use ao criar, revisar ou depurar handlers Gin, routers, DTOs ou serviços HTTP em Go.
---

# Golang APIs (Gin)

Contratos claros e estáveis vencem handlers ad hoc. Toda mutação passa por binding, validação e autorização no servidor.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **Gin** | Router, middleware, binding, context |
| **encoding/json** | JSON request/response (ou lib do projeto) |
| **context** | Cancelamento e valores de request |
| **validator** | Tags `binding:"required,..."` do Gin |

Para concorrência de background, WebSocket ou testes: use `golang-goroutines`, `golang-websocket`, `golang-unit-testing`. Contratos HTTP gerais: alinhe com `api-design`.

## Organização recomendada

```
cmd/api/main.go
internal/
  config/
  http/
    router.go
    middleware/
    handlers/
  domain/          # ou services/ por bounded context
  repository/
```

- `main` só wiring: config, deps, `r.Run`.
- Handlers finos; regra de negócio em services.
- Preserve layout e libs já adotados pelo projeto.

## Regras inegociáveis

- Handlers: `func(c *gin.Context)`; nunca ignore `c.Request.Context()`.
- Bind com `ShouldBindJSON` / `ShouldBindQuery` / URI; falhe com `400` tipado.
- Authn ≠ Authz: autenticado não autoriza ownership/role.
- Não exponha structs internas com secrets; use DTOs de response.
- Erros estáveis: `{ "code", "message", "details?" }` + status HTTP corretos.
- Config e secrets via env; sem hardcode.
- Graceful shutdown com `http.Server` + signal (não só `r.Run` cego em produção).

## Router e grupos

```go
r := gin.New()
r.Use(gin.Recovery(), RequestID(), Logger())

v1 := r.Group("/api/v1")
{
    v1.GET("/health", health)

    auth := v1.Group("/")
    auth.Use(Authenticate())
    auth.GET("/todos", listTodos)
    auth.POST("/todos", createTodo)

    admin := v1.Group("/admin")
    admin.Use(Authenticate(), RequireRole("admin"))
    admin.GET("/users", listUsers)
}
```

- Versionamento (`/api/v1`) quando a API for pública ou multi-cliente.
- Middleware de auth no grupo, não copiado em cada rota.
- Path params tipados; rejeite IDs inválidos cedo.

## Binding, validação e erros

```go
type CreateTodoRequest struct {
    Title string `json:"title" binding:"required,min=1,max=200"`
}

func createTodo(c *gin.Context) {
    var req CreateTodoRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.AbortWithStatusJSON(http.StatusBadRequest, gin.H{
            "code": "validation_error", "message": "invalid request", "details": err.Error(),
        })
        return
    }
    // service call with c.Request.Context()
}
```

- Status: `400`, `401`, `403`, `404`, `409`, `422`, `429`, `500` conforme o caso.
- `code` machine-readable; `message` segura; sem stack em produção.
- Distinga validação (cliente) de falha interna (servidor).

## Paginação e filtros

- Cursor-based para listas grandes; offset só quando simples e limitado.
- Limite page size; rejeite valores abusivos.
- Ordenação determinística.
- Documente filtros suportados.

## Autorização

- Extraia identity no middleware; guarde em `c.Set` com chave tipada/constante.
- Verifique ownership/roles no handler ou service em toda rota sensível.
- Rate limit em auth e endpoints caros quando o risco existir.
- CORS explícito se houver front separado.

## Observabilidade

- Logue request id, user id (se houver), rota, status, latência.
- Não logue secrets nem payloads sensíveis completos.
- Propague `context.Context` até DB e clients HTTP.

## Anti-padrões

- Lógica de negócio inteira no handler
- `c.Get` sem checar existência / tipo
- `200` com erro no body sem convenção
- Ignorar cancelamento do request
- Validação só no frontend
- Gin em mode debug em produção
- Panic em caminho feliz em vez de erro HTTP

## Fluxo ao implementar uma feature

1. Inspecione router, middleware e handlers existentes.
2. Defina DTO request/response + regras de validação.
3. Service com a regra de negócio (context-aware).
4. Handler Gin com authz e status corretos.
5. Testes dos casos de risco (`golang-unit-testing`).
6. Atualize config/env/docs só do que mudou.

## Critérios de conclusão

- Rotas agrupadas e versionadas quando aplicável
- Input validado; erros e status consistentes
- Authz coberta nas rotas sensíveis
- Paginação/limites seguros
- Context propagado; sem vazamento de campos internos
- Contrato utilizável por pelo menos um cliente real do repo
