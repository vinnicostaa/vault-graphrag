---
title: ACL — Access Control List
type: concept
tags:
  - security
  - acl
  - authorization
  - dac
  - posix
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Access Control List
  - ACL
  - DAC
---

# ACL — Access Control List

**O que é**: modelo mais antigo de autorização — lista de **(sujeito, permissões)** anexada diretamente ao **recurso**. "Quem pode fazer o quê com ISTO?" Forma de Discretionary Access Control (DAC): owner do recurso decide quem acessa.

**Quando usar**: ver [[authorization-models#Como-escolher]]. Em curto: quando o número de recursos × sujeitos é gerenciável e cada grant é direto.

## Formato clássico

### POSIX ACL (Linux filesystem)

```
$ getfacl /data/confidential.txt
# file: /data/confidential.txt
# owner: alice
# group: engineering
user::rw-                    # owner: read+write
user:bob:rw-                 # user bob: read+write (ACL extension)
user:charlie:r--             # user charlie: read-only
group::r--                   # group engineering: read
group:finance:rw-            # group finance: read+write
mask::rw-                    # máscara de permissões efetivas
other::---                   # demais: nenhuma
```

Cada linha é entrada ACL: `(tipo, sujeito, permissões)`.

### S3 ACL (legado AWS)

```xml
<AccessControlPolicy>
  <Owner><ID>user-123</ID></Owner>
  <AccessControlList>
    <Grant>
      <Grantee xsi:type="CanonicalUser"><ID>user-456</ID></Grantee>
      <Permission>READ</Permission>
    </Grant>
    <Grant>
      <Grantee xsi:type="Group"><URI>http://acs.amazonaws.com/groups/global/AllUsers</URI></Grantee>
      <Permission>READ</Permission>
    </Grant>
  </AccessControlList>
</AccessControlPolicy>
```

AWS hoje desencoraja S3 ACL em favor de **Bucket Policy** (JSON, RBAC+ABAC). ACL ainda existe por compatibilidade mas **deprecated** para novos buckets (`BucketOwnerEnforced` default desde 2023).

### Windows NTFS ACL

DACL (Discretionary ACL) + SACL (System ACL para audit). Entries são ACEs (Access Control Entries) com `(SID, access_mask, flags, type=allow/deny)`.

## DAC vs MAC vs RBAC

- **DAC (Discretionary)**: owner decide. ACL/POSIX/NTFS são DAC. Flexível, difícil centralizar política.
- **MAC (Mandatory)**: sistema impõe (ex: SELinux, AppArmor, multi-level secrets classifications). Rígido; governo/militar.
- **RBAC**: papéis intermediam; admin controla. Ver [[rbac]].

ACL tradicional é DAC.

## Padrão canônico (ACL em aplicação)

```go
type ACLEntry struct {
    Subject    string   // user_id, group_id
    Permission string   // "read", "write", "delete", "share"
}

type ResourceACL struct {
    ResourceID string
    Owner      string
    Entries    []ACLEntry
}

func (acl *ResourceACL) Can(userID, permission string) bool {
    if userID == acl.Owner { return true }          // owner tem tudo
    for _, e := range acl.Entries {
        if e.Subject == userID && e.Permission == permission { return true }
    }
    return false                                     // deny by default
}
```

Em banco: tabela `resource_acl(resource_id, subject_id, permission)` com índice em `(resource_id, subject_id)`.

## Quando ACL é suficiente

- **Compartilhamento ad-hoc** simples ("este doc para você"). Quando grafo não compensa.
- **Filesystem / object storage** (legado; novos preferem bucket policy RBAC+ABAC).
- **Recursos com poucos sharings** (<10 grants por recurso).
- **Sem regras contextuais** (horário, IP, MFA).

## Quando ACL **não** basta

- **N usuários × N recursos** com sharing intenso → escala quadrática. Use [[rebac]] (grafo).
- **Hierarquia de recursos** (folder → sub-folders herdam) — ACL precisa propagação; [[rebac]] modela nativamente.
- **Permissões dependem de atributos** (horário, role, tenant) — [[abac]].
- **Auditoria de "por que negou"** difícil com ACL puro.

## ACL + RBAC híbrido

Comum: RBAC global (admin/user) + ACL local (compartilhamento de recurso individual).

Ex: Dropbox — user tem role base (regular/paid); sharing de pasta específica via ACL (grant a user X com read).

## Anti-patterns

1. **ACL + grafo implícito** — ACL em cada arquivo + pasta com ACL separada + herança ad-hoc. Migre para [[rebac]].
2. **ACL sem owner-bypass controlado** — owner sempre pode conceder a si mesmo? Sim, mas auditar. `share → {who, when, reason}`.
3. **ACL em tabela sem índice** — query `WHERE subject_id = ?` full scan. Índice composto obrigatório.
4. **"Public" como ACL entry** — use flag `is_public: bool` separado em vez de entry mágica (clareza).
5. **Comparação de string de permission** — use enum/constant (`PermissionRead = 1`).
6. **Sem TTL em grant temporário** — grant "até semana que vem" fica para sempre.

## S3 ACL legacy — como migrar

AWS orienta migrar de S3 ACL para **Bucket Policy + Object Ownership: `BucketOwnerEnforced`**:
- Bucket policy é JSON com RBAC+ABAC explícito.
- Object Ownership `BucketOwnerEnforced` desabilita ACL entirely.
- Simplifica auditoria (uma política por bucket vs N ACLs por objeto).

## Relações

- **Decision tree**: [[authorization-models]]
- **Alternativas**: [[rbac]] (centraliza por papel), [[abac]] (regras atributos), [[rebac]] (grafo nativo)
- **Extensão conceitual**: [[rebac]] é "ACL em grafo" (Zanzibar começou resolvendo problemas de ACL em escala Google Docs)
- **Runtime antipattern**: [[../runtime-antipatterns/op-025-coluna-sem-indice|OP-025]] (índice em tabela ACL é obrigatório)

## Fontes

- [POSIX.1e ACLs — draft](https://pubs.opengroup.org/onlinepubs/009695399/basedefs/sys/acl.h.html) (consultada 2026-04-24)
- [Linux `setfacl` / `getfacl` man pages](https://man7.org/linux/man-pages/man1/setfacl.1.html) (consultada 2026-04-24)
- [AWS S3 Object Ownership + ACL disable](https://docs.aws.amazon.com/AmazonS3/latest/userguide/about-object-ownership.html) (consultada 2026-04-24)
- [Microsoft NTFS Security Descriptors](https://learn.microsoft.com/en-us/windows/win32/secauthz/access-control-lists) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[authorization-models]]
- [[rbac]]
- [[abac]]
- [[rebac]]
- [[least-privilege]]
