---
name: golang-goroutines
description: Concorrência idiomática em Go com goroutines, channels, context, errgroup, worker pools e sync. Use ao paralelizar trabalho, evitar vazamentos, orquestrar cancelamento ou revisar código concorrente em Go.
---

# Golang Goroutines

Toda goroutine precisa de um dono, um fim e um caminho de erro. Preferir `context` para cancelamento a signals ad hoc.

## Quando usar

- Paralelizar I/O independente (fan-out) com limite de concorrência.
- Pipelines com stages claros e backpressure.
- Trabalho em background ligado ao lifecycle do processo ou do request.
- Não use goroutine para “deixar o handler retornar mais rápido” sem ownership do lifecycle.

## Regras inegociáveis

- Derive contexts: `context.WithCancel` / `WithTimeout` / `WithDeadline` a partir do parent.
- Cancele e `Wait` (ou equivalente) antes de retornar em funções que spawnam trabalho.
- Nunca ignore o valor de retorno de erros de goroutines filhas.
- Channels: defina quem fecha; receivers não fecham o que não possuem.
- Dados compartilhados: `mutex`, channels ou estruturas concorrentes — escolha uma estratégia, não as duas às cegas.
- Sem `go func()` em loop capturando variável de iteração sem binding explícito (`v := v`).

## Context e errgroup

```go
g, ctx := errgroup.WithContext(ctx)
g.SetLimit(8)

for _, id := range ids {
    id := id
    g.Go(func() error {
        return fetchOne(ctx, id)
    })
}

if err := g.Wait(); err != nil {
    return err
}
```

- `golang.org/x/sync/errgroup` para fan-out com cancelamento no primeiro erro.
- `SetLimit` para evitar explosão de goroutines.
- Propague o mesmo `ctx` até I/O (DB, HTTP, disco).

## Worker pool

```go
jobs := make(chan Job)
var wg sync.WaitGroup

for i := 0; i < workers; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        for job := range jobs {
            process(ctx, job)
        }
    }()
}

for _, job := range batch {
    select {
    case <-ctx.Done():
        close(jobs)
        wg.Wait()
        return ctx.Err()
    case jobs <- job:
    }
}
close(jobs)
wg.Wait()
```

- Buffer só com motivo (desacoplar ritmos); unbounded growth é bug.
- Shutdown: pare de enviar → feche channel de jobs → `Wait` workers.

## Channels e select

- Prefira communication via channels para handoff de ownership.
- `select` com `ctx.Done()` em sends/receives que podem bloquear.
- Evite channel de notificação + boolean confuso; use `close` ou context.
- Fan-in: uma goroutine por source ou `errgroup`, merge explícito.

## sync

| Primativa | Uso |
|-----------|-----|
| `Mutex` / `RWMutex` | Estado compartilhado curto |
| `WaitGroup` | Esperar N workers |
| `Once` | Init idempotente |
| `Map` / atomics | Casos específicos; meça antes de otimizar |

- Segure o lock o mínimo possível; não faça I/O sob mutex.
- Documente invariantes do estado protegido.

## Vazamentos e races

- Sintomas: goroutines crescendo, testes flaky, shutdown que não termina.
- Ferramentas: `go test -race`, pprof goroutine, timeouts em testes.
- Cada `go` deve ter condição de saída ligada a context, channel close ou `WaitGroup`.

## Anti-padrões

- Goroutine sem cancelamento nem join
- `time.Sleep` como coordenação
- Fire-and-forget em request HTTP sem tracking
- Fechar channel do lado do receiver
- Mutex + channel para a mesma coordenação sem necessidade
- Ignorar `ctx.Err()` após `select`
- Pool sem limite sob carga

## Critérios de conclusão

- Toda goroutine tem lifecycle claro (start/stop)
- Context propagado e respeitado
- Erros das filhas agregados ou tratados
- Sem data races (`-race` limpo nas áreas tocadas)
- Backpressure ou limite de concorrência onde há fan-out
- Shutdown não deixa trabalho órfão
