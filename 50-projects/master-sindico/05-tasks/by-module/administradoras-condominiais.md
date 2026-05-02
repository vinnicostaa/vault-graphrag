---
title: Tasks Administradoras Condominiais (D-114)
type: note
tags:
  - task-list
  - master-sindico
  - by-module
project: master-sindico
milestone_target: m1-m2
status: backlog
created: 2026-04-24T00:00:00.000Z
---

# Tasks Administradoras Condominiais

Backlog priorizado para sub-produto 11 (Administradoras Condominiais / Empresa-Síndica). Canonizado via [[../../STATE#d-114|D-114]] (2026-04-24).

**Contexto**: Administradora é empresa (role `enterprise` + subcategoria `administradoras-condominios`) que pode ser **síndica profissional** de um ou mais condomínios, via vínculo `EmpresaSindicaLink`. Backend expõe 5 endpoints em `empresa_sindica_handler.go`; UI zero. Cliente Master Síndico (admin plataforma) precisa do mínimo para **auditar e controlar**.

## Tasks priorizadas

### M1 — Sprint 10+ (mínimo viável, pós-2026-05-18 se necessário)

#### T-ADMSIND-01 — S-ADM-01 Dashboard Administradora

**Prioridade**: P0 (essencial para admin operar)
**Esforço**: 2-3 dias (web) + 0.5 dia (backend se endpoint precisar agregação)

**Escopo**:
- Tela que empresa (role `enterprise` subtype `administradoras`) acessa via menu
- Lista de condomínios que gerencia (ativos + encerrados)
- Por condomínio: nome, endereço, síndico atual (titular individual ou auto-gerido), data vínculo, status mandato, Score Governança agregado, Score Compliance agregado, alerta flag (pendências)
- Filtros: status mandato, cidade, score range
- CTA: "Operar" (redireciona para S6 com flag `as_admin`)

**Backend**:
- `GET /api/v1/empresa-sindica/:company_id/condominiums` — já existe no código (conforme análise F6)
- Verificar se resposta tem scores agregados — se não, usar 2 queries paralelas ou agregador

**UI**: [[../../00-product/sub-produtos/11-administradoras-condominiais|admin-administradora/S-ADM-01 (a criar)]] (criar subpasta e arquivo)

#### T-ADMSIND-02 — S-ADM-02 Aceitar convite de síndico

**Prioridade**: P0 (essencial para vincular)
**Esforço**: 2 dias

**Escopo**:
- Tela que recebe link de convite via email (token URL `/convites/administradora/:token`)
- Mostra: nome do condomínio convidante, síndico solicitante, permissões propostas (subset das 5 canônicas)
- 5 permissões configuráveis (baseado em análise F6):
  - Operar assembleias (mesa + homologar votações + encerrar)
  - Validar registros de execução
  - Publicar comunicados
  - Editar plano diretor
  - Acessar dashboard financeiro do condomínio
- CTA: Aceitar (registra `EmpresaSindicaLink`) ou Recusar com motivo

**Backend**:
- `POST /api/v1/empresa-sindica/invites/:token/accept`
- `POST /api/v1/empresa-sindica/invites/:token/reject`

**UI**: [[../../00-product/sub-produtos/11-administradoras-condominiais|admin-administradora/S-ADM-02 (a criar)]]

### M2 — Expansão (2026-06+)

#### T-ADMSIND-03 — S-ADM-03 Operar condomínio como administradora

**Prioridade**: P1
**Esforço**: 1-2 dias (reusa S6 com flag)

**Escopo**:
- Redireciona para S6 (Central de Governança do Síndico) com flag `as_admin = true`
- Badge visual "Operando como Administradora — {nome}" no topo
- Ações liberadas/bloqueadas conforme permissões aceitas em S-ADM-02
- Todas as ações logam em `admin_audit_log` com `actor_role=administradora + actor_empresa_id`

#### T-ADMSIND-04 — S-ADM-04 Revogar vínculo de administradora

**Prioridade**: P1
**Esforço**: 1 dia

**Escopo**:
- Síndico titular pode revogar vínculo com administradora (tela S2 tem CTA "Gerenciar administradora")
- Modal de confirmação com motivo
- Emite evento `EmpresaSindicaLink.Revoked` — administradora perde acesso imediato
- Audit log completo

#### T-ADMSIND-05 — E-ADM-01 Aceitar como administradora (upgrade perfil)

**Prioridade**: P1
**Esforço**: 2 dias

**Escopo**:
- Empresa com subcategoria `administradoras-condominios` pode ativar perfil de administradora (flag `is_active_administrator`)
- Tela de onboarding específico: certificações, CRECI, registro CNPJ como administradora profissional
- Marcadores autodeclarados específicos (ABRACS, seguro RC-administradora, etc.)
- Approval workflow: auto-aprovado em MVP; review admin MS em M3+

### M3 — Features avançadas (2026-07+)

#### T-ADMSIND-06 — Multi-condomínio dashboard consolidado

**Escopo**: administradora que gerencia 10+ condomínios tem visão consolidada: indicadores financeiros agregados, assembleias em andamento em tempo real, alertas críticos.

**Esforço**: 4-5 dias

#### T-ADMSIND-07 — Relatórios consolidados PDF/Excel

**Escopo**: admin gera relatórios PDF/Excel consolidados multi-condomínio para apresentar a clientes (condomínios).

**Esforço**: 3 dias

## Observações

- **Todas tasks ADMSIND-*** consumem design-tokens canônicos ([[../../02-architecture/design-tokens]]).
- **Permissões granulares**: ABAC matrix precisa cobrir cada combinação role=empresa subtype=administradora + action + condominium_id (com grant explícito via EmpresaSindicaLink).
- **Audit log específico**: além do audit global, eventos de administradora gerenciando condomínio ficam em tabela separada para auditoria fácil (empresas-síndicas são relacionamento B2B sensível — LGPD + compliance).
- **Conflito de interesse**: ADR futuro sobre como prevenir administradora que gerencia condomínio votar em Connect Me indicando empresa-irmã.

## Links

- [[../_moc]]
- [[../../STATE#d-114|D-114]]
- [[../../00-product/sub-produtos/11-administradoras-condominiais]]
- [[admin-plataforma]] — análogo para Admin Master Síndico
- [[../../03-subprojects/admin-privilegios]]
