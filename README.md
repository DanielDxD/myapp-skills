# myapp-skills

Repositório de Agent Skills instaláveis via [`npx skills`](https://github.com/vercel-labs/skills).

## Instalação

```bash
# Todas as skills deste repositório
npx skills add <owner>/myapp-skills

# Uma skill específica
npx skills add <owner>/myapp-skills --skill cinematographic-websites

# Listar skills disponíveis (local)
npx skills add ./ --list
```

Substitua `<owner>/myapp-skills` pelo caminho GitHub do repositório após o publish.

## Catálogo

| Skill | Uso |
|-------|-----|
| `website-copywriting` | Copy humana, específica e persuasiva para sites e interfaces |
| `cinematographic-websites` | Sites cinematográficos com Lenis, GSAP e ScrollTrigger |
| `apple-inspired-websites` | Sites de produto estilo apple.com (scrub, carrosséis, comparativos) |
| `project-discovery` | Mapear stack, arquitetura e riscos antes de editar |
| `nextjs-architecture` | App Router, server/client, cache e features |
| `nextjs-server-actions` | Mutações App Router com Server Actions, forms e revalidate |
| `nextjs-server-actions-security` | Authz, CSRF/origem, IDOR e hardening de Server Actions |
| `react-component-engineering` | Componentes React reutilizáveis e acessíveis |
| `reactjs-application-development` | SPAs React (Vite, Router, server state) |
| `react-native-development` | Expo / React Native, navegação e performance mobile |
| `react-native-expo-navigation` | Expo Router no SDK 57 (Stack/Tabs, typed routes, deep links) |
| `better-mobile-interface-react-native` | UI RN/Expo moderna com tokens e paridade light/dark |
| `react-native-live-activities` | Live Activities iOS e ongoing updates Android |
| `react-native-background-location` | GPS em background (task manager, FGS, stores) |
| `react-native-realtime-location` | Localização em tempo real em foreground + broadcast |
| `react-native-routes` | Trajetos no mapa (Directions, polyline, ETA, re-route) |
| `taxi-machine` | App de transporte white-label (passageiro/motorista) |
| `google-maps-apis` | Directions, Places, Geocoding e demais APIs HTTP Google |
| `google-maps-sdk` | MapView nativo (react-native-maps / Expo Maps) |
| `geolocation` | Obter coordenadas com permissões e accuracy |
| `reverse-geocoding` | Coordenada → endereço (reverse geocode) |
| `swiftui-development` | Apps e features SwiftUI |
| `swift-testing` | Testes Swift Testing (unitário) + XCUITest (interface) |
| `android-development` | Apps Android nativos com Kotlin e Jetpack |
| `jetpack-compose` | UI Android com Jetpack Compose e Material 3 |
| `android-unit-testing` | Testes unitários Android/Kotlin (ViewModel, coroutines, fakes) |
| `better-mobile-interface` | UI mobile moderna e consistente (Compose + SwiftUI, light/dark) |
| `better-web-interface` | Design e consistência de interfaces web |
| `design-system` | Tokens, componentes, patterns e governança de design system |
| `storybook-docs` | Stories CSF, Docs/Autodocs e documentação viva no Storybook |
| `vapor-swift` | APIs e backends server-side Swift com Vapor 4 e Fluent |
| `rust` | Linguagem Rust: ownership, async, crates e tooling Cargo |
| `axum` | APIs HTTP em Rust com Axum, Tower e extractors |
| `rust-best-practices` | Idioms, Clippy, API design e hygiene de crates Rust |
| `rust-axum-production` | Axum endurecido para produção (shutdown, observabilidade, deploy) |
| `rust-axum-websocket` | WebSockets em Rust com Axum |
| `rust-unit-testing` | Testes unitários e de integração em Rust com cargo test |
| `golang-apis` | APIs HTTP em Go com Gin |
| `golang-microservices` | Microsserviços em Go (HTTP/gRPC, mensageria, outbox) |
| `golang-goroutines` | Concorrência em Go: goroutines, channels e context |
| `golang-websocket` | WebSockets em Go com Gin e gorilla/websocket |
| `golang-unit-testing` | Testes unitários e de handler em Go |
| `golang-scylladb` | Integração Go com ScyllaDB (gocql/gocqlx) |
| `golang-video-dash` | Transcode MP4 → MPEG-DASH com FFmpeg e serving VOD em Go |
| `tus-golang-nextjs` | Upload resumível TUS (tusd/Gin + tus-js-client no Next.js) |
| `nestjs-microservices` | Microsserviços NestJS (transporters, patterns, hybrid) |
| `rust-microservices` | Microsserviços em Rust (Axum/tonic, mensageria, outbox) |
| `microservices-architecture` | Arquitetura de microsserviços: limites, dados, eventos, ops |
| `terraform` | Infraestrutura como código com Terraform |
| `scylladb` | Modelagem CQL, partition keys e operações ScyllaDB |
| `elasticsearch` | Indexação, Query DSL e aggregations Elasticsearch |
| `floci` | Emulador local AWS (Floci) para dev e CI |
| `ollama-api` | API Ollama (chat, stream, OpenAI-compatible) |
| `claude-api` | API Anthropic Claude (Messages, tools, stream) |
| `chatgpt-api` | API OpenAI ChatGPT (Completions/Responses, tools) |
| `gemini-api` | API Google Gemini (generateContent, tools, stream) |
| `mvvm-architecture` | Separação View / ViewModel / Model |
| `typescript-strict` | Tipagem estrita e validação na borda |
| `api-design` | Contratos HTTP, erros, paginação e authz |
| `database-prisma` | Schema, migrations e queries Prisma |
| `database-drizzle` | Schema, migrations e queries Drizzle ORM |
| `directus-integration` | Integra aplicações com o Directus via API/SDK, consultas e erros |
| `directus-data-modeling` | Modela Collections, Fields, Relations e constraints no Directus |
| `directus-auth-permissions` | RBAC, permissões e policies no Directus para segurança |
| `directus-extensions` | Hooks e flows do Directus para regras de negócio e automações |
| `directus-files` | Upload e consumo de assets no Directus com permissões e cache |
| `directus-migrations-import-export` | Migrações/snapshots e import/export de schema/dados no Directus |
| `tauri` | Apps desktop com Tauri 2 |
| `authentication-authorization` | Sessões, RBAC, ownership e cookies seguros |
| `testing-fullstack` | Vitest, Testing Library e Playwright |
| `web-performance` | Core Web Vitals e otimização mensurável |
| `production-readiness` | Segurança, observabilidade e go-live |

## Convenções

- Cada skill vive em `skills/<nome>/SKILL.md`.
- Frontmatter exige `name` e `description` (o quê + quando).
- Skills são autoacionáveis por descrição; requisitos do projeto prevalecem sobre a skill em caso de conflito.
- `react-component-engineering` cobre APIs de componentes; `reactjs-application-development` cobre a aplicação completa.
