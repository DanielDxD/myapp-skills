---
name: docker-compose
description: Orquestra serviços locais e de CI com Docker Compose v2 (docker compose), healthchecks, networks, volumes, profiles e env_file. Use ao criar ou revisar compose.yaml, stacks app+DB, depends_on ou ambientes de desenvolvimento containerizados.
---

# Docker Compose

Compose Spec v2: comando **`docker compose`** (plugin), não o binário legado `docker-compose`. Imagem única / Dockerfile: skill `docker`.

Compose brilha em **dev e CI**. Produção no mesmo arquivo só se o projeto já operar assim (ex.: VPS única). Kubernetes/Nomad não entram nesta skill.

## Regras inegociáveis

- Sem chave `version:` (obsoleta no Compose Spec).
- `depends_on` com `condition: service_healthy` quando o consumidor precisa do DB/broker **pronto**, não só “container up”.
- Secrets e senhas via `env_file` / variáveis do host — não hardcode em YAML commitado (valores de dev locais ok se documentados e não-prod).
- Volumes nomeados para dados; bind mounts para código em dev.
- Rede explícita se houver mais de um serviço se falando; não assuma `default` em docs de time.
- Imagens pinadas (`postgres:16-alpine`), iguais à disciplina da skill `docker`.

## Layout

```
compose.yaml              # (ou compose.yml / docker-compose.yml já no repo)
compose.override.yaml     # dev local, gitignored ou commitado se o time acordar
.env.example
```

Preserve o nome de arquivo que o repo já usa. Novo arquivo: `compose.yaml`.

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    env_file: .env
    environment:
      DATABASE_URL: postgres://app:app@db:5432/app
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://127.0.0.1:3000/health"]
      interval: 10s
      timeout: 3s
      retries: 5
      start_period: 20s
    networks: [internal]

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
      POSTGRES_DB: app
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d app"]
      interval: 5s
      timeout: 3s
      retries: 10
    networks: [internal]

networks:
  internal:

volumes:
  pgdata:
```

## Serviços

- Nome DNS = nome do serviço (`db`, `redis`). App conecta em `db:5432`, não `localhost` de dentro do container.
- `build` vs `image`: build para o código do repo; image para software de prateleira.
- `ports` só o que o host precisa; resto fica na network interna.
- `restart: unless-stopped` em stacks de longa duração; `no` em jobs one-shot.
- `profiles: [tools]` para adminers, mailhog, seeders — não subam no `up` default se forem opcionais.

```yaml
services:
  mailhog:
    image: mailhog/mailhog:v1.0.1
    profiles: [dev]
    ports: ["8025:8025"]
```

`docker compose --profile dev up`.

## Env e secrets

- `.env` ao lado do compose é interpolado no YAML (`${VAR}`). `env_file` injeta no container. Não confunda os dois.
- `.env.example` commitado; `.env` no `.gitignore`.
- `environment:` sobrescreve `env_file` para URLs internas (`db`, não `localhost`).
- Compose `secrets:` quando o time já usa Docker secrets; senão env/file bind read-only.

## Dependências e ordem

- Healthcheck no **serviço dependido** (Postgres, Redis, Rabbit).
- `condition: service_started` só quando não há health (evite).
- `condition: service_completed_successfully` para migrators one-shot:

```yaml
services:
  migrate:
    build: .
    command: ["bundle", "exec", "rails", "db:prepare"]
    depends_on:
      db:
        condition: service_healthy
```

App `depends_on: migrate: { condition: service_completed_successfully }` quando quiser migrate-before-boot.

## Dev vs prod

| | Dev | Prod (se Compose) |
|--|-----|-------------------|
| Código | bind mount + bind sync | só artefato da imagem |
| Comando | `command:` override (reload) | CMD da imagem |
| Portas | publicadas | mínimas / interno |
| Recursos | soltos | `deploy.resources` / limites do engine |

`compose.override.yaml` (merge automático) para mounts e `tty`. Não deixe override de prod acidental.

## Comandos

```bash
docker compose up --build
docker compose up -d
docker compose logs -f app
docker compose exec app sh
docker compose down           # mantém volumes
docker compose down -v        # destrói dados — peça confirmação
```

Nunca `-v` em ambiente com dados reais sem o usuário pedir.

## Anti-padrões

- `docker-compose` v1 / `version: "3.8"`
- `depends_on: [db]` sem healthcheck (app crash-loop no boot)
- `network_mode: host` sem motivo (quebra DNS de serviço)
- Bind de `/var/run/docker.sock` em app de negócio
- Uma network `external` sem documentar quem cria
- Senha de prod no YAML
- `ports: "5432:5432"` em CI compartilhado sem necessidade (conflito)

## Integração com outras skills

- Dockerfile / hardening da imagem: `docker`
- App Rails/Phoenix/Nuxt: a skill da stack define o `command` e a URL do DB
- Go-live real em cluster: `production-readiness` (+ orquestrador do time)

## Critérios de conclusão

- `docker compose config` válido, sem `version`
- Healthchecks nos dados/brokers; app espera `service_healthy`
- Rede interna; portas publicadas só o necessário
- `.env.example` alinhado; secrets fora do git
- `docker compose up --build` sobe o fluxo local documentado
