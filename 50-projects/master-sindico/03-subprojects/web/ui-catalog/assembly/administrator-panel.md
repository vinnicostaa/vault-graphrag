---
title: Administrator Panel — UI Spec
type: ui-spec
tags: [ui-catalog, assembly, master-sindico, frontend, administradora, abac]
bc: assembly
source: _chaos/ARQUITETURA-ASSEMBLEIA.pdf §3.3-3.4 + DIAGRAMA-VISUAL.pdf §2 + MÓDULO-TELAS.pdf TELA 3,7,8
absorbed_in: 2026-04-25 — Fase E
status: absorbed
---

# Administrator Panel — Painel da Administradora (D-127)

> **Origem**: 3 PDFs Master Síndico — destilados em Fase E (2026-04-25).
> **D-127 (2026-04-25)**: Administradora = entity `AssemblyAdministratorLink` por-assembleia (cargo cedido pelo síndico, auto-invalida em `Assembly.End()`).
> **PermissionGroup canônico (ADR-0039 D-116 IAM)**: `pg.assembly.administrator_delegation`.

> Sprint 2-4 · App `assembly` · Rotas: `/admin/assembleias/:id/*`
> Perfil: Administradora (com Membership ABAC `role=administrator` E AssemblyAdministratorLink ativo).

---

## Visão Geral — 4 Ações Canônicas

A administradora tem **4 capabilities** (PermissionGroup `pg.assembly.administrator_delegation`):

| Capability | Tela | Quando |
|---|---|---|
| `assembly.fracao_ideal.import` | Importar Fração Ideal | Pré-assembleia (1x ou em correções) |
| `assembly.inadimplencia.register` | Cadastro Inadimplência | Até T-1h da 1ª convocação |
| `assembly.proxy.validate` | Validar Procurações | Pré-assembleia até "Iniciar" |
| `assembly.assembly.homologate` | Homologar Votação | Durante/pós assembleia |

**Capabilities adicionais** (R3.4 doc do cliente):

- `assembly.observation.add` — adicionar observações (com validação opcional do síndico).
- `assembly.edital.generate` — gerar edital automático ou anexar.
- `assembly.assembly.end` — encerrar assembleia (alternativa ao síndico).
- `assembly.vote.assisted_register` — registrar voto presencial assistido.

> **Configuração de validação cruzada**: o síndico pode configurar se observações da administradora exigem validação dele antes de virar oficial. Default: validação obrigatória.

---

## Tela 1 — Aceite de Convite (Onboarding)

**Rota**: `/admin-invite/:token` (acesso público via link de email)

```
┌──────────────────────────────────────────────────┐
│  CONVITE PARA ADMINISTRAR ASSEMBLEIA             │
│                                                  │
│  Você foi convidado por Carlos Pereira (síndico) │
│  do condomínio Aurora para administrar a         │
│  Assembleia Ordinária 2026/1 (15/03/2026).       │
│                                                  │
│  Empresa: Admin XYZ Ltda                         │
│  CNPJ: 12.345.678/0001-90                        │
│  Responsável: Maria Souza (CPF *.***.***-44)     │
│                                                  │
│  Capabilities concedidas:                        │
│  • Importar fração ideal                         │
│  • Registrar inadimplência                       │
│  • Validar procurações                           │
│  • Homologar votação                             │
│  • Encerrar assembleia                           │
│                                                  │
│  Validade: até encerramento da assembleia        │
│  + 24h (auto-expira)                             │
│                                                  │
│  Confirme seu CPF para ativar:                   │
│  [___.___.___-__]                                │
│                                                  │
│  [✓] Aceito o termo de responsabilidade          │
│      (Lei 14.063/2020 + LGPD)                    │
│                                                  │
│  [Recusar]                  [Ativar acesso →]   │
└──────────────────────────────────────────────────┘
```

### Fluxo backend (D-127)

1. Token HMAC validado (`{assembly_id, expires_at}`).
2. Confirma CPF do responsável.
3. Aceita termo de responsabilidade.
4. `AssemblyAdministratorLink.status: pending_invite → accepted`.
5. Quando `Assembly.Start()` → `accepted → active`.
6. Cascade `Assembly.End()` → `active → expired`.

---

## Tela 2 — Dashboard da Administradora

**Rota**: `/admin/assembleias` (Administradora autenticada)

```
┌──────────────────────────────────────────────────────┐
│  📊 PAINEL DA ADMINISTRADORA — Maria Souza          │
│  Empresa: Admin XYZ Ltda                             │
│                                                      │
│  ASSEMBLEIAS ATIVAS                                  │
│  ┌────────────────────────────────────────────────┐ │
│  │ Aurora — Ordinária 2026/1                      │ │
│  │ 🟡 EM PREPARAÇÃO • 15/03/2026 (T-3 dias)      │ │
│  │ Capabilities: ✓ todas as 4                     │ │
│  │ ⚠️ 3 pendências:                              │ │
│  │ • Importar fração ideal                        │ │
│  │ • 2 procurações aguardando validação           │ │
│  │ • Inadimplência não cadastrada                 │ │
│  │ [Abrir →]                                       │ │
│  ├────────────────────────────────────────────────┤ │
│  │ Edelweiss — Extraord. Reforma                  │ │
│  │ 🟢 EM ANDAMENTO • 14/03/2026                   │ │
│  │ [Painel ao vivo →]                              │ │
│  └────────────────────────────────────────────────┘ │
│                                                      │
│  HISTÓRICO (últimas 30 dias)                        │
│  • Aurora — Implantação (12/01) ✅ ENCERRADA       │
└──────────────────────────────────────────────────────┘
```

---

## Tela 3 — Painel de Construção da Assembleia (visão admin)

**Rota**: `/admin/assembleias/:id`

```
┌──────────────────────────────────────────────────────┐
│  ASSEMBLEIA ORDINÁRIA 2026/1 (T-3 dias)              │
│  Status: PUBLICADA • Síndico: Carlos Pereira         │
│                                                      │
│  ┌─CHECKLIST ADMIN──────────────────────────────────┐│
│  │ ✅ Convite aceito                                ││
│  │ ✅ Edital validado                               ││
│  │ ⚠️ Importar fração ideal [Abrir →]              ││
│  │ ⚠️ Inadimplência [Abrir →]                      ││
│  │ ⚠️ Validar procurações (3 pendentes) [Abrir →]  ││
│  │ ⏳ Homologação (após votação)                    ││
│  │ ⏳ Voto presencial assistido (durante)           ││
│  └──────────────────────────────────────────────────┘│
│                                                      │
│  OBSERVAÇÕES DA ADMINISTRADORA                       │
│  ┌──────────────────────────────────────────────┐   │
│  │ [+ Adicionar observação]                     │   │
│  │ ⚠️ Observações exigem validação do síndico    │   │
│  │   (configurável)                              │   │
│  └──────────────────────────────────────────────┘   │
│                                                      │
│  [📊 Score projetado: 78/100] [📈 Simulador]        │
└──────────────────────────────────────────────────────┘
```

---

## Tela 4 — Importar Fração Ideal

> **Tela detalhada em [[./pre-assembly#tela-9-importar-fração-ideal-administradora|pre-assembly]]**.

Capability: `assembly.fracao_ideal.import`. Capacidade audita evento `FracaoIdealImported` (audit trail).

---

## Tela 5 — Cadastro de Inadimplência

> **Tela detalhada em [[./pre-assembly#tela-10-registrar-inadimplência-administradora|pre-assembly]]**.

Capability: `assembly.inadimplencia.register`. Cutoff T-1h da 1ª convocação.

---

## Tela 6 — Validar Procurações

> **Tela detalhada em [[./pre-assembly#tela-8-validar-procurações-administradora|pre-assembly]]**.

Capability: `assembly.proxy.validate`. ASM-010: 100% digital.

---

## Tela 7 — Voto Presencial Assistido

> **Tela detalhada em [[./live-session#voto-presencial-assistido-asm-016--r57-§6|live-session]]**.

Capability: `assembly.vote.assisted_register`. Termo de voto + operator_id audit trail.

---

## Tela 8 — Homologação da Votação

> **Tela detalhada em [[./post-assembly#tela-3-homologação-votação-obrigatória-pós-ata-—-asm-027|post-assembly]]**.

Capability: `assembly.assembly.homologate`. Síndico OU administradora (D-127).

---

## Tela 9 — Encerramento da Assembleia (alternativo ao síndico)

> **Tela detalhada em [[./post-assembly#tela-1-encerrar-assembleia-síndicoadministradora|post-assembly]]**.

Capability: `assembly.assembly.end`.

---

## Tela 10 — Auditoria de Ações da Administradora (Síndico)

**Rota**: `/assembleias/:id/admin/audit-trail` (Síndico read-only)

```
┌──────────────────────────────────────────────────────┐
│  AUDIT TRAIL — Administradora (Maria Souza)          │
│                                                      │
│  Filtros: [Todas ações ▼] [Período ▼]               │
│                                                      │
│  ┌────────────────────────────────────────────────┐ │
│  │ 12/03 14:32 — fracao_ideal.import              │ │
│  │ Hash planilha: 7e8f...                         │ │
│  │ Soma frações: 100.0000 ✓                       │ │
│  ├────────────────────────────────────────────────┤ │
│  │ 12/03 14:45 — inadimplencia.register           │ │
│  │ Unidades bloqueadas: 12                        │ │
│  │ Origem: upload planilha                        │ │
│  ├────────────────────────────────────────────────┤ │
│  │ 13/03 10:21 — proxy.validate (rejeitada)       │ │
│  │ ProxyID: prx_7a3...                            │ │
│  │ Justificativa: "CPF não confere com unidade"  │ │
│  ├────────────────────────────────────────────────┤ │
│  │ 13/03 10:25 — proxy.validate (aprovada)        │ │
│  │ ProxyID: prx_8b2...                            │ │
│  │ Voto será replicado para Apt B-105             │ │
│  │ ...                                            │ │
│  └────────────────────────────────────────────────┘ │
│                                                      │
│  Total ações: 47 • Última: 14/03 21:34              │
└──────────────────────────────────────────────────────┘
```

### Regras (ASM-### NOVO Fase E — auditoria 20+ eventos)

Todas as ações da administradora geram evento `AssemblyAdministratorLinkUsed` (audit append-only). Síndico pode auditar a qualquer momento. Eventos cobertos:

- `fracao_ideal.import`
- `inadimplencia.register`
- `inadimplencia.liberar_voto`
- `proxy.validate.approve`
- `proxy.validate.reject`
- `vote.assisted_register`
- `assembly.homologate`
- `assembly.end`
- `observation.add`
- `edital.generate`

---

## Termo de Responsabilidade

```
Ao aceitar este convite, você declara que:

1. É responsável legal designado pela empresa Admin XYZ Ltda
   para administrar esta assembleia.

2. As ações executadas em seu nome (importação de fração
   ideal, registro de inadimplência, validação de procurações,
   homologação de votação) estarão registradas em audit trail
   imutável (Lei 14.063/2020).

3. Compromete-se ao tratamento confidencial dos dados
   pessoais dos moradores conforme LGPD.

4. Está ciente que o acesso é cedido temporariamente pelo
   síndico Carlos Pereira e auto-expira ao encerramento da
   assembleia + 24h.

5. A Master Síndico atua apenas como meio tecnológico — não
   se responsabiliza pelo mérito das deliberações.

[ ] Li e aceito os termos acima
```

---

## Componentes

`AdminInviteAcceptScreen`, `AdminDashboard`, `AdminAssemblyChecklist`, `AdminObservationForm`, `AdminAuditTrailViewer`, `LinkStatusBadge`.

## Cross-links

- [[./pre-assembly|pre-assembly]]
- [[./live-session|live-session]]
- [[./post-assembly|post-assembly]]
- [[../../../../04-requirements/functional/assembly|functional/assembly]] — ASM-001 (admin link), ASM-010 (proxy validate)
- [[../../../../01-domain/aggregates/AssemblyAdministratorLink|AssemblyAdministratorLink]] (D-127 entity)
- [[../../../../02-architecture/adr/0039-iam-cloudflare-style-master-sindico|ADR-0039 IAM]] (PermissionGroup catalog)
- [[../../../../02-architecture/adr/0028-lgpd-keyed-hmac|ADR-0028 HMAC keyed]] (token)
- [[../../../../STATE]] — D-127, D-116

## Pendências

- ⚠️ UI de configuração do síndico para "validação cruzada de observações" (default ON ou OFF?).
- ⚠️ Provider de assinatura digital ICP-Brasil (Bry, ITI) — ADR M4+.
- ⚠️ Capability fina-granular: `assembly.observation.add.requires_sindico_approval` vs `auto_publish` — modelar em PermissionGroup.
