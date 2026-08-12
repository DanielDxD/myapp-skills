---
name: terraform
description: Provisiona infraestrutura com Terraform — state remoto, modules, workspaces/envs, providers, planos seguros, secrets e boas práticas de IaC. Use ao criar ou revisar .tf, modules, backends, CI de plan/apply ou mudanças de cloud (AWS, GCP, Azure, etc.).
---

# Terraform

Infra como código: mudanças revisáveis via `plan`, state remoto e módulos reutilizáveis. O que está no state é a verdade operacional — trate com cuidado.

## Conceitos

| Peça | Papel |
|------|--------|
| **Resource** | Objeto gerenciado (instância, fila, DNS…) |
| **Data source** | Leitura de algo existente |
| **Module** | Encapsula um pedaço reutilizável |
| **State** | Mapeamento endereço → ID real |
| **Backend** | Onde o state vive (S3+Dynamo, GCS, Terraform Cloud…) |
| **Provider** | Cloud/API (AWS, GCP, Azure, K8s…) |

## Regras inegociáveis

- State remoto + lock; nunca state local compartilhado em time.
- Secrets **não** vão para git nem para outputs sensíveis em claro no CI log.
- `terraform plan` revisado antes de `apply` em qualquer ambiente não-efêmero.
- Pin de versões: `required_version`, `required_providers` com constraints conscientes.
- Mudanças destrutivas (`force new`, delete) exigem confirmação explícita e plano de rollback.
- Um root por ambiente/conta (ou workspaces com disciplina) — evite um state gigante global.

## Layout sugerido

```text
infra/
  modules/
    networking/
    service-ecs/      # ou gke, lambda, etc.
  envs/
    dev/
    staging/
    prod/
      main.tf
      variables.tf
      outputs.tf
      backend.tf
      providers.tf
      versions.tf
```

- Modules sem backend próprio; roots de env configuram backend e passam variáveis.
- Nomes e tags padrão: `project`, `env`, `owner`, `managed-by=terraform`.

## Estado e backends

- Backend com encryption at rest e acesso least-privilege.
- Lock table/API para evitar applies concorrentes.
- `terraform state` mv/rm só com backup e entendimento de impacto.
- Importe recursos existentes com `import` block / `terraform import` quando for assumir gestão — não recrie produção sem necessidade.

## Modules e interfaces

- Inputs tipados (`variable` com `type`, `validation`, `sensitive = true` quando couber).
- Outputs mínimos; marque `sensitive` quando necessário.
- Versionamento de modules (git tag / registry); evite `main` flutuante em prod.
- Não exponha credenciais via output.

## Ambientes

- `dev` / `staging` / `prod` isolados (conta ou projeto cloud separados quando possível).
- Valores via `tfvars`, CI vars ou secrets store — não hardcode IDs de prod em module genérico.
- Workspaces: use só se o time já padronizou; caso contrário, roots separados são mais claros.

## Segurança

- IAM/roles least privilege para o runner de Terraform e para recursos criados.
- Nada de access keys de longa duração em código; prefira OIDC/role assumption no CI.
- Security groups / firewall: deny by default, abra só o necessário.
- Criptografia default em discos, buckets, filas quando a cloud oferecer.
- Escaneie plan (checkov/tfsec/trivy) se o pipeline já tiver — ou proponha no escopo de hardening.

## CI/CD

```text
fmt → validate → plan (PR) → apply (main/prod com aprovação)
```

- Plan artifact no PR; apply só na branch protegida / environment com review.
- `TF_LOG` verboso só para debug local — não no CI rotineiro com secrets.
- `-target` é escape hatch, não fluxo normal.

## Ciclo de mudança

1. Leia state/backend e modules existentes (`project-discovery` mental).
2. Altere module/root no menor escopo.
3. `terraform fmt -recursive` + `validate`.
4. `plan` e explique destroys/replaces.
5. `apply` após aprovação; verifique outputs e smoke na app.
6. Documente outputs necessários para o app (URLs, ARNs) sem secrets.

## Anti-padrões

- State no git
- Credenciais em `*.tf` / `tfvars` commitados
- Um único state para toda a empresa
- `apply` local em prod sem plan revisado
- Copiar-colar resource blocks em vez de module quando há 3+ envs
- Ignorar `lifecycle` / prevent destroy em recursos irrecuperáveis sem backup

## Integração

- Microsserviços que sobem em cloud: `microservices-architecture` + skill da linguagem
- App ready: `production-readiness`
- Não misture provisionamento de app (K8s manifests) com Terraform sem fronteira clara — TF para cloud foundation; CD para deploy de app, salvo o padrão do repo

## Critérios de conclusão

- Backend remoto + lock ok
- `fmt` / `validate` / `plan` limpos e revisados
- Sem secrets no repositório
- Modules/interfaces claros; envs isolados
- Destroid/replaces compreendidos antes do apply
- Tags e naming consistentes com o projeto
