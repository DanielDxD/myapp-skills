---
name: golang-unit-testing
description: Testes unitários e de handler em Go com testing, table-driven tests, httptest e Gin. Use ao adicionar testes, cobrir handlers/services, eliminar flakiness ou definir estratégia de teste em projetos Go.
---

# Golang Unit Testing

Teste comportamento de risco, não detalhes privados. Prefira table-driven claros a suítes frágeis.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **testing** | `Test*`, `t.Run`, `t.Helper` |
| **httptest** | Requests HTTP in-memory |
| **Gin** | `gin.New()` / mode test / handlers reais |
| **fakes** | Ports na borda (repo, clock, bus) |

Use `testify` só se o projeto já depender dele. Concorrência: rode com `-race` nos pacotes tocados.

## O que priorizar

- Authz e ownership
- Validação e status HTTP
- Mutações e erros de domínio (`404`, `409`)
- Regressões já ocorridas (bug → teste)
- Helpers puros e policies

## Table-driven

```go
func TestParseCursor(t *testing.T) {
    tests := []struct {
        name    string
        in      string
        want    Cursor
        wantErr bool
    }{
        {name: "vazio", in: "", want: Cursor{}},
        {name: "inválido", in: "%%%", wantErr: true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := ParseCursor(tt.in)
            if tt.wantErr {
                if err == nil {
                    t.Fatalf("expected error")
                }
                return
            }
            if err != nil {
                t.Fatalf("unexpected err: %v", err)
            }
            if got != tt.want {
                t.Fatalf("got %#v want %#v", got, tt.want)
            }
        })
    }
}
```

- Nome: comportamento + condição.
- Cases independentes; sem ordem global implícita.
- `t.Helper()` em asserts compartilhados.

## Handlers Gin

```go
func TestCreateTodo_Unauthorized(t *testing.T) {
    gin.SetMode(gin.TestMode)
    r := gin.New()
    r.POST("/todos", createTodo) // injete deps fake

    req := httptest.NewRequest(http.MethodPost, "/todos", bytes.NewBufferString(`{"title":"x"}`))
    req.Header.Set("Content-Type", "application/json")
    w := httptest.NewRecorder()
    r.ServeHTTP(w, req)

    if w.Code != http.StatusUnauthorized {
        t.Fatalf("status=%d body=%s", w.Code, w.Body.String())
    }
}
```

- Monte o router mínimo necessário (middleware real de auth + fake deps).
- Prefira `ServeHTTP` no engine a testar só com `CreateTestContext` quando o middleware importa.
- Compare status + campos relevantes do JSON, não o body inteiro frágil.

## Fakes e bordas

- Interface na borda (repository, mailer); fake in-memory no teste.
- Não mocke o sistema sob teste.
- Relógio/UUID/rand: injete para determinismo.
- DB real: reserve a testes de integração; isole com schema/transaction por teste.

## Concorrência e flaky

- `go test -race` em pacotes com goroutines.
- Evite `time.Sleep`; use channels, conditions ou clocks injetados.
- Timeouts via `context` nos testes que esperam eventos.
- Limpe estado global (`gin` mode ok; evite vars mutáveis de pacote).

## Organização

```
internal/service/service_test.go     # unit
internal/http/handlers/handler_test.go
```

- `_test` package externo (`package foo_test`) quando testar API pública.
- Package interno quando precisar de brancos internos com cuidado.
- Fixtures pequenas; builders em vez de JSON gigante copiado.

## Anti-padrões

- Um único teste monolítico sem `t.Run`
- Assert em mensagem de erro instável como contrato
- Dependência de rede/relógio real sem controle
- Testar só o happy path de authz
- Mocks gerados em excesso para tipos internos
- Ignorar `-race` em código concorrente novo

## Critérios de conclusão

- Riscos da mudança cobertos (authz, validação, erros)
- Table-driven legíveis e determinísticos
- Handlers exercitados via HTTP quando o contrato importa
- Fakes só na borda
- `go test` (e `-race` se concorrente) passando nos pacotes tocados
