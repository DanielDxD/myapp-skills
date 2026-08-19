---
name: angular
description: Desenvolve apps Angular 22 com standalone, signals, Signal Forms, resource/httpResource, control flow e inject(). Use ao criar ou revisar projetos Angular, componentes, rotas, forms, HttpClient ou quando o usuário pedir Angular, NgRx legado ou signals.
---

# Angular 22

Código novo: **standalone + signals**. Signal Forms e `resource`/`httpResource` estão estáveis. Preserve NgModules, Reactive Forms e RxJS pesado onde o projeto já vive — não reescreva numa feature lateral.

## Stack típica

| Peça | Papel |
|------|--------|
| **Angular 22** | Components, router, DI |
| **Signals** | Estado síncrono da UI |
| **Signal Forms** | Forms novos (`@angular/forms/signals`) |
| **httpResource / resource** | Dados async reativos |
| **Router** | Rotas standalone, guards, resolvers |

HttpClient clássico + `firstValueFrom` só se o projeto ainda não adotou `httpResource`. Estilo: o design system do app (Material, CDK, Angular Aria).

## Organização

```
src/app/
  core/              # providers, auth, interceptors
  shared/            # ui reutilizável
  features/<domínio>/
    routes.ts
    *.ts             # component + template + styles colocalizados
  app.config.ts
  app.routes.ts
  app.ts
```

- Um feature = rotas + componentes + serviços daquele domínio.
- `providedIn: 'root'` para singletons; providers no `Route` para escopo de feature.
- Não recrie `NgModule` em código novo.

## Regras inegociáveis

- Componentes `standalone: true` (default atual); `imports` explícitos no `@Component`.
- Estado de UI: `signal` / `computed` / `linkedSignal`. Não sincronize signal↔subject sem necessidade.
- Control flow novo: `@if`, `@for` (track), `@switch`, `@defer`. Sem `*ngIf` / `*ngFor` / `*ngSwitch` em templates novos.
- DI: `inject()` no field initializer; constructor só quando o estilo legado do arquivo exigir.
- Authz no servidor; guards (`CanActivateFn`) só para UX/navegação.
- Change detection: OnPush / zoneless se o app já estiver assim — não reative Zone.js num app zoneless.

## Componentes e signals

```ts
@Component({
  selector: 'app-item-list',
  templateUrl: './item-list.html',
  imports: [RouterLink],
})
export class ItemList {
  private readonly api = inject(ItemsApi)

  readonly query = signal('')
  readonly items = httpResource(() => `/api/items?q=${this.query()}`)

  readonly filtered = computed(() => this.items.value() ?? [])
}
```

```html
@if (items.error(); as err) {
  <p role="alert">{{ err.message }}</p>
} @else if (items.isLoading()) {
  <p>Carregando…</p>
} @else {
  @for (item of filtered(); track item.id) {
    <a [routerLink]="['/items', item.id]">{{ item.name }}</a>
  } @empty {
    <p>Nenhum item.</p>
  }
}
```

- Inputs: `input()` / `input.required()`; outputs: `output()`.
- Two-way: `model()`.
- Queries: `viewChild` / `contentChild` (signal queries), não `@ViewChild` em código novo.
- `@defer` para blocos pesados abaixo da dobra; placeholders e `on viewport` conscientes.

## Forms

Código novo: Signal Forms.

```ts
import { form, FormField, required } from '@angular/forms/signals'

readonly data = signal({ email: '' })
readonly loginForm = form(this.data, (path) => {
  required(path.email)
})
```

- Template: `[formField]` + importe `FormField`.
- Reactive Forms (`FormGroup`, `FormControl`) só em telas já existentes ou widgets que ainda exigem `ControlValueAccessor`.
- Migração pontual: `compatForm` — não misture os dois modelos no mesmo formulário sem ponte.

## Dados async

- `resource` / `httpResource` para GET reativo a signals (query, id).
- Mutações: serviço + `httpClient`/`fetch` e recarregar o `resource` (`reload()`), ou invalidar a fonte.
- RxJS: operators em APIs que já são streams (WebSocket, eventos). Não envolva todo HTTP em `Observable` novo.
- `computed` para derivar; `effect` só para I/O (log, storage, DOM). Effects não são para encadear estado.

## Router

- `Routes` com `loadComponent` / `loadChildren` para lazy.
- Guards e resolvers funcionais: `CanActivateFn`, `ResolveFn`.
- `withComponentInputBinding()` para params → `input()`.
- `provideHttpClient(withInterceptors([...]))` em `app.config.ts`.

## A11y

- Controles nativos primeiro; **Angular Aria** para widgets complexos (listbox, menu, tabs) em vez de ARIA artesanal.
- Foco visível, labels, `track` estável no `@for`.
- CDK a11y (`LiveAnnouncer`, focus trap) quando o kit do projeto já usa CDK.

## Anti-padrões

- `NgModule` em feature nova
- `*ngIf` / `ngFor` clássico em template novo
- `constructor(private http: HttpClient)` em classe nova
- `subscribe()` em template via campos manuais em vez de `resource`/`async` já padronizado
- Signal Forms + `[(ngModel)]` no mesmo form
- Store NgRx para um GET simples
- Reativar Zone.js para “fazer o binding funcionar” — corrija o signal

## Integração com outras skills

- UI web: `better-web-interface`
- Tipagem: `typescript-strict`
- Testes: `testing-fullstack` (adapte para Karma/Jest/Vitest do repo)
- Authz: `authentication-authorization`

## Critérios de conclusão

- Componentes standalone com control flow novo
- Estado em signals; HTTP via `resource`/`httpResource` ou padrão já do repo
- Forms novos em Signal Forms
- Rotas lazy e guards funcionais
- Sem NgModules/`*ngIf` introduzidos sem necessidade
- Build e testes do escopo passando
