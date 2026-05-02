---
title: Least Privilege — princípio raiz de segurança
type: concept
tags:
  - security
  - principle
  - least-privilege
  - authorization
  - defense-in-depth
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Principle of Least Privilege
  - POLP
  - Least Privilege
---

# Least Privilege — princípio raiz

**O que é**: cada entidade (usuário, processo, serviço, token, key) deve ter **apenas** as permissões **mínimas necessárias** para sua função, pelo **menor tempo possível**. Formulado por Saltzer & Schroeder em *The Protection of Information in Computer Systems* (1975). Princípio #1 de segurança — **todas** as políticas de autorização derivam dele.

> "Every program and every privileged user of the system should operate using the least amount of privilege necessary to complete the job." — Saltzer & Schroeder

## Três dimensões

### 1. **Scope** — menos permissões

Token/role só tem permissões que o caso de uso exige. "Admin full access" = antipattern; fatiar em `admin:users`, `admin:billing`, `admin:content`.

### 2. **Time** — menor duração

Access tokens curtos (5-60min); refresh rotativo; privilégios elevados temporários (Just-In-Time access, JIT).

### 3. **Blast radius** — menor impacto se comprometido

- Service A não deve ter acesso ao DB de service B.
- CI/CD token não deve deployar em prod sem aprovação explícita.
- API key ≠ root key.

## Aplicações cross-domain

### Autorização de usuário

- Começar com **deny by default** → adicionar permissões explicitamente.
- Nunca "grant all" por atalho. Role sprawl é sintoma (ver [[rbac]]); ABAC ou PBAC pode ser fix.
- Ver [[authorization-models]], [[rbac]], [[abac]].

### Cloud IAM

- **AWS**: policies com `Effect: Allow` + `Action: s3:GetObject` (não `s3:*`); `Resource: arn:aws:s3:::my-bucket/*` (não `*`); `Condition` com MFA/IP/tag.
- **Google Cloud**: predefined roles > basic roles (Editor/Viewer); custom roles quando fino.
- **Azure**: built-in roles específicos (Storage Blob Data Reader, não Storage Account Contributor).

### Database

- User da aplicação: `SELECT, INSERT, UPDATE, DELETE` em tabelas específicas. **Não** `DBA`, **não** `SUPERUSER`.
- Migration user: separado; só roda migrations, não tem acesso a dados.
- Read-replica user: `SELECT` only; usado em relatórios.
- **Row-Level Security** (Postgres RLS) para tenant isolation forte.

### Secrets / credentials

- API keys com scope mínimo (ex: GitHub fine-grained PAT com `contents:read` em 1 repo, não classic PAT com `repo` all).
- Secret rotation periódica (90 dias padrão).
- **Break-glass** accounts: credenciais emergenciais em cofre, auditadas, rara vez usadas.
- Ver [[secrets-management]].

### OAuth scopes

- Request **somente** scopes necessários. `openid email` para login; não pedir `offline_access` se não precisa refresh.
- Incremental authorization: pedir scope adicional apenas quando feature específica é usada.
- Ver [[oauth-2]].

### Pods / containers

- Kubernetes: não rodar como `root`; `securityContext: runAsUser: 1000`.
- `capabilities: drop: ["ALL"]` + add apenas as necessárias.
- Service Account com RBAC restrito (ver `20-stacks/kubernetes/` quando destilar).

### File system

- App lê só o que precisa. `chmod 644` ≠ `777`.
- `/etc/shadow` permissões restritas.
- POSIX ACL (ver [[acl]]) para casos específicos.

## Padrões que operacionalizam POLP

### Separation of Duties (SoD)

User que **aprova** ≠ user que **executa**. Transação financeira crítica: requester submete, approver aprova, executor dispara.

### Just-In-Time (JIT) access

Privilégios elevados expiram rapidamente. Ex: Azure PIM, AWS IAM Identity Center session duration, Google Workspace privileged roles com TTL.

### Break-glass

Conta emergency com **alerta obrigatório** ao ser usada. Audit completo da sessão. Senha em cofre físico/digital; rotacionada após uso.

### Separation of Environments

Dev → staging → prod. Credenciais, contas cloud e dados **distintos**. Engenheiro de dev não tem prod access por default.

### Workload identity over static credentials

Serviço usa identidade derivada do ambiente (AWS IAM Role, GCP service account, K8s ServiceAccount, SPIFFE SVID) em vez de API key estática. Ver [[spiffe-spire]].

## Anti-patterns

1. **`admin: true`** flag em banco — bypass sem audit; comitê inteiro fica administrador.
2. **Wildcard resource** em policy (`Resource: *`) — explode blast radius.
3. **Token eterno** — JWT de 30 dias + sem rotação = RCE latente.
4. **Role "superuser"** compartilhado por 10 pessoas — audit impossível.
5. **Secret em repo** (ou `.env` comitado) — público + permanente.
6. **Service account genérico** compartilhado por N serviços — se comprometido, N explodem.
7. **Production access default** para engenharia — deveria ser exceção com audit.
8. **CI/CD com escopo amplo** — token do workflow lê todos os secrets da org (GitHub default problemático). Fine-grained quando possível.
9. **"Temporary elevation" sem TTL** — elevação temporária que virou permanente.
10. **Production debug com root** — shell de emergência sem gravar comandos.

## Compliance & frameworks

POLP é requisito explícito em:
- **PCI DSS** 7.1 (access limitation)
- **ISO 27001** A.9.1 (access control policy)
- **SOC 2** CC6.1 (logical access)
- **HIPAA** §164.312(a)(1)
- **NIST SP 800-53** AC-6 "Least Privilege"
- **GDPR** art. 32 (security of processing)

## Como grandes empresas operacionalizam

- **Google BeyondCorp**: todo acesso contextual; "trust" calculado por request. Ver [[beyond-corp]].
- **Netflix**: "Bless" — SSH cert de curta duração gerado por policy engine.
- **AWS Well-Architected**: pilar "Security" com POLP como item #1.
- **Microsoft Azure**: PIM (Privileged Identity Management) — roles elevados por demanda, aprovação + MFA + gravação de sessão.
- **Lyft/Netflix/Shopify**: internal tool "BatPermit"-like — permission escalation flow com review.
- **Shopify**: "Brainframe" — workflow de approval para acesso prod com peer review.

## Relações

- **Modelos**: [[authorization-models]], [[rbac]], [[abac]], [[rebac]], [[acl]]
- **Zero Trust**: [[beyond-corp]] operacionaliza POLP em rede
- **IAM**: [[iam]], [[identity-provider]], [[mfa-step-up]]
- **Secrets**: [[secrets-management]], [[credential-storage]]
- **Workload identity**: [[spiffe-spire]]
- **Cloud-specific**: AWS IAM policies, GCP IAM, Azure RBAC (ver providers quando destilar)
- **Runtime antipatterns**: [[../runtime-antipatterns/op-021-admin-sem-audit|OP-021]] (bypass sem audit), [[../runtime-antipatterns/op-011-logica-negocio-infra|OP-011]]
- **OWASP**: [[owasp-top-10]] A01 (Broken Access Control), A07 (Auth Failures)

## Fontes

- [Saltzer & Schroeder — The Protection of Information in Computer Systems (1975)](https://www.cs.virginia.edu/~evans/cs551/saltzer/) (consultada 2026-04-24)
- [NIST SP 800-53 — AC-6 Least Privilege](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final) (consultada 2026-04-24)
- [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) (consultada 2026-04-24)
- [Google Cloud IAM best practices](https://cloud.google.com/iam/docs/using-iam-securely) (consultada 2026-04-24)
- [Microsoft Zero Trust](https://learn.microsoft.com/en-us/security/zero-trust/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[authorization-models]]
- [[beyond-corp]]
- [[iam]]
- [[rbac]]
- [[abac]]
- [[secrets-management]]
- [[spiffe-spire]]
- [[security-principles]]
