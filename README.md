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
| `cinematographic-websites` | Sites cinematográficos com Lenis, GSAP e ScrollTrigger |
| `project-discovery` | Mapear stack, arquitetura e riscos antes de editar |
| `nextjs-architecture` | App Router, server/client, cache e features |
| `react-component-engineering` | Componentes React reutilizáveis e acessíveis |
| `reactjs-application-development` | SPAs React (Vite, Router, server state) |
| `react-native-development` | Expo / React Native, navegação e performance mobile |
| `swiftui-development` | Apps e features SwiftUI |
| `mvvm-architecture` | Separação View / ViewModel / Model |
| `typescript-strict` | Tipagem estrita e validação na borda |
| `api-design` | Contratos HTTP, erros, paginação e authz |
| `database-prisma` | Schema, migrations e queries Prisma |
| `authentication-authorization` | Sessões, RBAC, ownership e cookies seguros |
| `testing-fullstack` | Vitest, Testing Library e Playwright |
| `web-performance` | Core Web Vitals e otimização mensurável |
| `production-readiness` | Segurança, observabilidade e go-live |

## Convenções

- Cada skill vive em `skills/<nome>/SKILL.md`.
- Frontmatter exige `name` e `description` (o quê + quando).
- Skills são autoacionáveis por descrição; requisitos do projeto prevalecem sobre a skill em caso de conflito.
- `react-component-engineering` cobre APIs de componentes; `reactjs-application-development` cobre a aplicação completa.
