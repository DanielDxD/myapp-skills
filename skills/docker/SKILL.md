---
name: docker
description: Constrói imagens Docker de produção com Dockerfile multi-stage, .dockerignore, USER non-root, HEALTHCHECK, tags pinadas e secrets só em runtime. Use ao dockerizar apps, revisar Dockerfile, reduzir imagem ou endurecer containers — sem orquestração multi-serviço.
---

# Docker

Uma imagem, um processo. Multi-serviço, redes e volumes compartilhados: skill `docker-compose`. Detecte o runtime (`package.json`, `Gemfile`, `mix.exs`, `go.mod`, `Cargo.toml`, `pyproject.toml`) e adapte — não copie um Dockerfile Node em app Elixir.

## Regras inegociáveis

- Tags **pinadas** (`node:22-alpine`, digest quando o risco exigir). Evite `latest` em produção.
- Multi-stage: build tools ficam no stage de build; runtime mínimo no final.
- `.dockerignore` antes do primeiro `COPY . .` — senão `node_modules`, `.git` e secrets entram no context.
- Secrets só em runtime (`ENV` injetado, files, orchestrator). Nunca `COPY .env` nem `ENV API_KEY=...` no Dockerfile.
- Processo como USER não-root.
- Um concern por container (app **ou** db **ou** worker).
- `HEALTHCHECK` em imagens que sobem serviço de longa duração.

## .dockerignore

```
.git
.gitignore
**/.env
**/.env.*
**/node_modules
**/dist
**/target
**/_build
**/deps
**/.venv
**/coverage
**/*.md
Dockerfile*
compose*.yml
compose*.yaml
```

Ajuste ao stack. Não ignore o que o build precisa (`package-lock.json`, `mix.lock`, `Gemfile.lock`).

## Multi-stage (padrão)

```dockerfile
# syntax=docker/dockerfile:1

FROM node:22-bookworm-slim AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

FROM deps AS build
COPY . .
RUN npm run build && npm prune --omit=dev

FROM node:22-bookworm-slim AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN useradd --system --uid 1001 --create-home app
COPY --from=build --chown=app:app /app/package.json ./
COPY --from=build --chown=app:app /app/node_modules ./node_modules
COPY --from=build --chown=app:app /app/dist ./dist
USER app
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD node -e "fetch('http://127.0.0.1:3000/health').then(r=>process.exit(r.ok?0:1)).catch(()=>process.exit(1))"
CMD ["node", "dist/server.js"]
```

Adapte:

| Stack | Build | Runtime |
|-------|--------|---------|
| Node | `npm ci` / `pnpm` | `node` no artefato |
| Rails | `bundle deploy`, assets | `rails server` / Puma |
| Phoenix | `mix deps.get` + `assets` + `mix release` | release OTP |
| Go | `CGO_ENABLED=0 go build` | scratch/distroless + binário |
| Rust | `cargo build --release` | slim + binário |

- `WORKDIR` explícito; `COPY` do mais estável (lockfile) para o mais volátil (fonte) — cache de layers.
- `RUN` combinados com sentido (menos layers), mas legíveis.
- Alpine: ótimo se as native deps compilam; glibc (Debian slim) quando `musl` quebra gems/N-API.

## Segurança e runtime

- `USER` numérico ou nomeado depois de instalar pacotes (apt precisa root).
- Não instale compilador no stage final.
- `--chown` no `COPY --from` para o user final.
- Filesystem: assuma read-only root em orquestradores; escreva só em `/tmp` ou volume.
- Drop capabilities e read-only ficam no compose/k8s — documente se a imagem exigir.
- `EXPOSE` é documentação; o bind real é no run/compose.

## Tags, build e CI

```bash
docker build -t myapp:$(git rev-parse --short HEAD) .
docker build --target build .   # validar stage
```

- Tag imutável (git sha) + `myapp:prod` móvel se o fluxo exigir.
- `--platform` explícito em CI (`linux/amd64`) quando o host é ARM.
- BuildKit (`DOCKER_BUILDKIT=1` / syntax directive) para cache mounts conscientes:

```dockerfile
RUN --mount=type=cache,target=/root/.npm npm ci
```

Não cacheie secrets.

## Anti-padrões

- `FROM ubuntu:latest` + `apt-get` de toolchain no mesmo stage que roda prod
- `COPY . .` sem `.dockerignore`
- Root como PID 1 sem tini/dumb-init quando o app não reage a SIGTERM
- `latest` em produção
- Dois processos (nginx + app) no mesmo container sem supervisor pedido
- Healthcheck que pinga a internet
- Comitar `.dockerignore` que ignora o lockfile

## Integração com outras skills

- Vários serviços, DB, redes: `docker-compose`
- Go-live: `production-readiness`
- Rails/Phoenix/Nuxt: use a skill da linguagem **e** esta para a imagem

## Critérios de conclusão

- Dockerfile multi-stage alinhado ao runtime detectado
- `.dockerignore` cobrindo git, deps e env
- User non-root; secrets fora da imagem
- Tag pinada; HEALTHCHECK se for serviço
- Imagem sobe e responde localmente (`docker build` + `docker run`)
