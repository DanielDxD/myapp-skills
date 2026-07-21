---
name: authentication-authorization
description: Autenticação e autorização seguras com sessões, RBAC/ownership, proteção server-side, cookies e prevenção de falhas comuns. Use ao implementar login, guards, middleware de auth, Clerk/Auth.js/NextAuth, ou auditar controle de acesso.
---

# Authentication & Authorization

Nunca confie no client para segurança. Authn identifica; Authz autoriza.

## Princípios

- Toda rota sensível verifica sessão/token no servidor.
- Ownership e roles são checados por recurso, não só “usuário logado”.
- Secrets só no servidor; cookies com flags corretas.
- Fail closed: na dúvida, negue.

## Sessões e tokens

- Prefira sessões server-side ou cookies httpOnly seguros quando o stack permitir.
- Se JWT: vida curta, refresh rotacionado, audience/issuer validados.
- Cookies: `HttpOnly`, `Secure`, `SameSite` adequado; path mínimo.
- Invalide sessão no logout e em reset de senha/compromisso.

## Autorização

- Modele papéis (RBAC) e/ou policies por recurso.
- Padrão: `can(user, action, resource)` centralizado.
- Impedir IDOR: nunca buscar por id sem escopo do usuário/tenant.
- UI pode esconder ações; servidor deve bloquear de qualquer forma.

## Fluxos comuns

- Login/signup com rate limit e mensagens que não vazam existência de conta quando aplicável.
- OAuth: state/nonce, redirect allowlist.
- Password reset: tokens de uso único e expiração curta.
- Email verification quando o domínio exigir.

## Integrações

- Respeite o provider do projeto (Clerk, Auth.js, Supabase, custom).
- Não misture dois sistemas de sessão.
- Webhooks de auth: verifique assinatura.

## Falhas comuns a prevenir

- Auth só no middleware sem checagem na action/handler
- Role no JWT sem revalidar permissões críticas
- CORS permissivo + cookies
- Tokens em `localStorage` sem mitigação XSS
- Endpoints de admin sem audit log

## Critérios de conclusão

- Rotas sensíveis protegidas no servidor
- Ownership/RBAC testado (pelo menos casos positivos/negativos)
- Cookies/tokens configurados com segurança
- Logout e expiração funcionam
- Sem vazamento de dados entre tenants/usuários
