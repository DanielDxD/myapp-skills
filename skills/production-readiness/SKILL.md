---
name: production-readiness
description: Prepara apps para produção com segurança, erros, logs, env, observabilidade, health checks e deploy seguro. Use antes de release, em checklists de go-live, hardening ou revisão de operação.
---

# Production Readiness

Pronto para produção significa previsível sob falha, observável e seguro por padrão.

## Checklist

### Segurança

- [ ] Secrets só em env/secret manager; nunca commitados
- [ ] Dependências auditadas; updates críticos aplicados
- [ ] Headers de segurança (CSP quando viável, HSTS, etc.)
- [ ] Authz em todas as rotas sensíveis
- [ ] Uploads e inputs validados; limites de tamanho
- [ ] CORS e cookies com least privilege

### Confiabilidade

- [ ] Health/readiness endpoints
- [ ] Timeouts e retries com backoff em I/O externo
- [ ] Circuit breaker ou degradação graciosa onde crítico
- [ ] Migrations seguras no pipeline de deploy
- [ ] Rollback plan documentado

### Observabilidade

- [ ] Logs estruturados com request id
- [ ] Níveis corretos; sem PII/secrets
- [ ] Error tracking (Sentry ou equivalente do projeto)
- [ ] Métricas: latência, erro, saturação
- [ ] Alertas acionáveis, não ruído

### Configuração

- [ ] Env validado no boot (schema)
- [ ] Separação clear: local / staging / prod
- [ ] Feature flags para mudanças arriscadas quando existirem
- [ ] Build reproduzível; versão/commit no runtime

### UX sob falha

- [ ] Páginas/estados de erro úteis
- [ ] Sem stack traces ao usuário
- [ ] Rate limit e mensagens claras em abuse paths

## Deploy

- Ordem típica: migrate → deploy app → verify health → smoke test.
- Blue/green ou canary quando o projeto já tiver.
- Não faça hotfixes manuais não rastreados em prod.

## Anti-padrões

- “Funciona na minha máquina” sem smoke em staging
- Logs que viram dump de body com tokens
- Alertas sem dono
- Deploy irreversível sem backup/migrate down strategy

## Critérios de conclusão

- Checklist crítico marcado
- Smoke do fluxo principal ok em ambiente alvo
- Erros visíveis no error tracker de staging/prod
- Rollback conhecido
- Secrets e env validados
