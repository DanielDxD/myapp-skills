---
name: testing-fullstack
description: Pirâmide de testes full-stack com Vitest, Testing Library e Playwright, fixtures, mocks e cobertura baseada em risco. Use ao adicionar testes, configurar CI de qualidade, depurar flakiness ou definir estratégia de teste.
---

# Testing Full-stack

Teste comportamento de risco, não implementação privada. Prefira poucos testes estáveis a muitos frágeis.

## Pirâmide

1. **Unit** — domínio, utils, ViewModels, policies, schemas.
2. **Integration** — API + DB (ou testcontainers), hooks com providers.
3. **E2E** — fluxos críticos do usuário (Playwright).

Não empurre tudo para E2E.

## O que priorizar

- Authz e ownership
- Pagamentos / mutações irreversíveis
- Validação de schemas e erros
- Fluxos principais: signup, checkout, create/edit core entity
- Regressões já ocorridas (bug → teste)

## Ferramentas

- Vitest (ou Jest do projeto) para unit/integration
- Testing Library: interaja como usuário (`getByRole`), evite detalhes de implementação
- Playwright: selectors resilientes, fixtures de auth, isolamento por teste
- MSW para HTTP no front quando útil

## Práticas

- Arrange / Act / Assert claros.
- Fixtures e factories reutilizáveis; evite copiar payloads gigantes.
- Mocke só na borda; não mocke o sistema sob teste.
- Determinismo: sem dependência de relógio/rede real sem controle.
- Nomes: `deve [comportamento] quando [condição]`.

## Anti-flaky

- Evite `waitForTimeout` fixo; espere condições.
- Isolamento de DB: transaction rollback ou DB limpa por worker.
- Seeds mínimos e explícitos.
- Retries só como último recurso no CI, com investigação.

## CI

- Typecheck + unit em todo PR
- E2E no PR ou main conforme custo
- Falha bloqueante para testes do caminho crítico

## Critérios de conclusão

- Pelo menos os riscos da mudança cobertos
- Testes legíveis e determinísticos
- E2E só nos fluxos que importam
- CI executa a suíte relevante
- Sem asserts frágeis em markup irrelevante
