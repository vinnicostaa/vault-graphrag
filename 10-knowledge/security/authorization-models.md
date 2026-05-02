---
title: Authorization models — decision tree (RBAC · ABAC · ACL · ReBAC · PBAC)
type: concept
tags:
  - security
  - authorization
  - rbac
  - abac
  - acl
  - rebac
  - pbac
  - access-control
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Authorization Models
  - Access Control Models
---

# Authorization models — decision tree

**O que é**: panorama comparativo dos **modelos de autorização** mais usados. Esta é a **nota-mãe** — filhas (`rbac`, `abac`, `acl`, `rebac`, `least-privilege`) linkam de volta aqui para o decision tree.

> **Regra**: tudo que é decision tree RBAC vs ABAC vs ACL vs ReBAC vs PBAC vive **aqui**. Notas filhas descrevem o modelo específico mas **não reproduzem** a comparação.

## O espectro

Do mais simples ao mais expressivo:

```
ACL        →  RBAC      →   ABAC        →  ReBAC         →  PBAC
(lista)    (papéis)       (atributos)    (relações)       (policy-as-code)
```

Não é estritamente linear — ABAC pode coexistir com RBAC; PBAC é meta (engine que implementa os outros).

## Tabela de decisão

| Dimensão | [[acl\|ACL]] | [[rbac\|RBAC]] | [[abac\|ABAC]] | [[rebac\|ReBAC]] | PBAC (ex.: [[opa-rego\|OPA]]) |
|---|---|---|---|---|---|
| **Unidade de autorização** | (sujeito, recurso, permissão) | (role → permissões; user → roles) | (atributos × atributos) | (user —relação→ resource via grafo) | policy expressa como código |
| **Expressividade** | Baixa | Média | Alta | Muito alta (herança via grafo) | Máxima (Turing-complete em alguns) |
| **Complexidade operacional** | Baixa | Baixa-média | Alta | Alta (infra graph) | Alta (engine + DX) |
| **Explosão combinatória** | Com muitos users/recursos | Role sprawl (500+ roles) | Attribute sprawl | Controlado via grafo | Controlado via módulos |
| **Auditabilidade** | Fácil | Fácil | Difícil ("por que negou?") | Médio | Fácil (policy é código) |
| **Exemplos reais** | POSIX, S3 ACL, NTFS | AWS IAM roles clássico, Linux groups, Jira | AWS IAM condition keys, Azure RBAC+conditions, modelo Master Síndico | Google Docs sharing, Zanzibar, GitHub | OPA/Rego, Casbin, Cedar (AWS Verified Permissions) |

## Como escolher

```
Você tem:
│
├─ Poucos recursos, poucos users, poucas permissões?
│   → [[acl]] (POSIX-like). Simples, direto, auditável.
│
├─ Roles estáveis, reutilizáveis por N users (admin/user/guest)?
│   → [[rbac]]. Clássico enterprise. Risk: role sprawl.
│
├─ Permissão depende de atributos do user + recurso + contexto?
│   (plano, device, hora, tenant, sensibilidade do dado)
│   → [[abac]]. Flexível mas complexo.
│
├─ "User tem permissão em X se está relacionado a X"?
│   (compartilhamento como Google Docs; equipes em GitHub)
│   → [[rebac]]. Grafos. Zanzibar.
│
├─ Policies complexas, mudam frequentemente, precisa versionar/testar?
│   → PBAC com policy engine ([[opa-rego|OPA]], Cedar, Casbin). Meta-modelo;
│     pode implementar RBAC/ABAC/ReBAC como casos específicos.
│
└─ Combinações:
   • RBAC + ABAC (híbrido) — muito comum. Role dá base; atributos refinam.
   • ABAC + ReBAC — quando há hierarquia + atributos contextuais.
   • Qualquer um + PBAC engine — OPA/Cedar para centralizar.
```

## Padrões combinados em produção

### RBAC + ABAC (mais comum em SaaS)

```
Regras: role = "admin" → permissões base
        AND tenant_id do recurso = tenant_id do user  (ABAC)
        AND plan >= "premium"                          (ABAC)
        AND horário business hours                     (ABAC)
```

Ver [[security-principles#ABAC-Engine]] para implementação concreta em Go.

### RBAC + ReBAC (GitHub-style)

- `role=owner` vale para repositório inteiro.
- Equipe com `write` permission sobre repo X herda via grafo.
- User individual pode ter grant direto.

### ABAC expresso em PBAC

OPA/Rego policy que consome atributos:
```rego
allow if {
  input.user.role == "manager"
  input.resource.tenant_id == input.user.tenant_id
  input.action == "read"
  input.context.mfa_verified
}
```

## Anti-patterns gerais

1. **Reinventar engine** — use AWS IAM, OPA, Casbin, Oso. Implementar do zero acumula CVEs.
2. **Misturar authn e authz** — "user logou, logo pode X" é bug clássico. Ver [[oauth-2]] (authz) vs [[oidc]] (authn).
3. **ABAC inline em handler** (`if user.role == "admin" { ... }`) — espalha regra. Centraliza em engine. Ver [[../runtime-antipatterns/op-011-logica-negocio-infra|OP-011]].
4. **Role sprawl** — 500+ roles viram `admin_finance_read_2024_eu`. Sinal de que ABAC é mais adequado.
5. **Regra "admin bypass" sem audit** — ver [[../runtime-antipatterns/op-021-admin-sem-audit|OP-021]].
6. **Autorização baseada em cache stale** — ver [[../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit|OP-008]].
7. **Confiar só no frontend** — toda verificação no servidor. Ver [[security-principles#Princípios-inegociáveis]].

## Princípios transversais

- **[[least-privilege|Princípio do menor privilégio]]** — concede menos, não mais.
- **Deny by default** — negar se regra não casa; nunca allow by default.
- **Defense in depth** — camadas: edge (WAF) + auth middleware + ABAC + row-level security no DB.
- **Separation of duties** — user que aprova ≠ user que executa (operações críticas).
- **Audit tudo** — `actor, action, resource, reason, ts` append-only.

## OWASP A01 aplicado

Broken Access Control é #1 em OWASP Top 10 2021. Testes obrigatórios:
- **IDOR** — trocar `id` em URL revela recurso de outro user?
- **Missing function-level check** — chamada direta do endpoint admin sem ir pelo menu?
- **Vertical escalation** — usuário comum acessa função de admin?
- **Horizontal escalation** — usuário A acessa dados de usuário B?
- **Tenant isolation** — cross-tenant leak via SQL sem `WHERE tenant_id = $ctx`?

Ver [[owasp-top-10#A01-Broken-Access-Control]].

## Relações

- **Modelos específicos**: [[rbac]], [[abac]], [[acl]], [[rebac]]
- **Princípio raiz**: [[least-privilege]]
- **Engine PBAC**: [[opa-rego]]
- **Implementações reais**: [[zanzibar-openfga]] (ReBAC), [[security-principles#ABAC-Engine]] (ABAC aplicado)
- **OWASP**: [[owasp-top-10]] A01
- **Runtime antipatterns**: [[../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit|OP-008]], [[../runtime-antipatterns/op-011-logica-negocio-infra|OP-011]], [[../runtime-antipatterns/op-021-admin-sem-audit|OP-021]]
- **Regra operacional**: [[security-principles]]
- **Zero Trust**: [[beyond-corp]]

## Fontes

- [NIST SP 800-162 — Guide to Attribute Based Access Control (ABAC)](https://csrc.nist.gov/publications/detail/sp/800-162/final) (consultada 2026-04-24)
- [NIST SP 800-207 — Zero Trust Architecture](https://csrc.nist.gov/publications/detail/sp/800-207/final) (consultada 2026-04-24)
- [Google Zanzibar paper (2019)](https://research.google/pubs/pub48190/) (consultada 2026-04-24)
- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) (consultada 2026-04-24)
- [OWASP Top 10 2021 — A01 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[rbac]]
- [[abac]]
- [[acl]]
- [[rebac]]
- [[opa-rego]]
- [[least-privilege]]
- [[security-principles]]
- [[beyond-corp]]
- [[owasp-top-10]]
