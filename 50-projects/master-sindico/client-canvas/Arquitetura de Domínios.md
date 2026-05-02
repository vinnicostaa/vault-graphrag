---
title: Arquitetura de Domínios (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Arquitetura de Domínios

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  group_com["📦 💼 HIGH-DOMAIN: COMERCIAL"]:::group
  group_inst["📦 🏛️ HIGH-DOMAIN: INSTITUCIONAL"]:::group
  group_cross["📦 ⚙️ CROSS-DOMAIN"]:::group
  group_cont["📦 📚 HIGH-DOMAIN: CONTEÚDO"]:::group
  group_id["📦 🔐 HIGH-DOMAIN: IDENTIDADE"]:::group
  node_iam["IAM<br/>Auth · ABAC · Planos · Billing"]
  node_assembly["Assembleia<br/>Ao Vivo · Votação · Telão"]
  node_notify["Notificações<br/>Email · Push · In-App"]
  node_condo["Condomínio<br/>Unidades · Vínculos · Mandatos"]
  node_viz["Vizinhança<br/>Parceiros · Promoções · Métricas"]
  node_media["Mídia<br/>Vídeos · Player · Paywall"]
  node_lms["Academia LMS<br/>Cursos · Lives · Fórum · Biblioteca"]
  node_gov["Governança<br/>Timeline · Plano Diretor · Compliance"]
  node_connect["Connect Me<br/>Marketplace Unidirecional"]
  node_profile["Perfis<br/>Visibilidade · Busca · Status"]
  node_service["Serviços<br/>Propostas · Contratos · Validação"]
  node_condo -->|tenant context| node_gov
  node_gov -->|Timeline imutável| node_service
  node_connect -->|oportunidades| node_service
  node_service -->|validação síndico| node_gov
  node_media -->|conteúdo| node_profile
  node_lms -->|integração| node_gov
  node_assembly -->|resultados| node_gov
  node_notify -->|eventos| node_gov
  node_notify -->|eventos| node_connect
  node_notify -->|eventos| node_assembly
  node_iam -->|autenticação| node_gov
  node_iam -->|plano define acesso| node_connect
  node_iam -->|plano define visibilidade| node_profile
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (16)

- **[GROUP]** `group_com` — 💼 HIGH-DOMAIN: COMERCIAL
- **[GROUP]** `group_inst` — 🏛️ HIGH-DOMAIN: INSTITUCIONAL
- **[GROUP]** `group_cross` — ⚙️ CROSS-DOMAIN
- **[GROUP]** `group_cont` — 📚 HIGH-DOMAIN: CONTEÚDO
- **[GROUP]** `group_id` — 🔐 HIGH-DOMAIN: IDENTIDADE
- `node_iam` — **IAM** · Auth · ABAC · Planos · Billing
- `node_assembly` — **Assembleia** · Ao Vivo · Votação · Telão
- `node_notify` — **Notificações** · Email · Push · In-App
- `node_condo` — **Condomínio** · Unidades · Vínculos · Mandatos
- `node_viz` — **Vizinhança** · Parceiros · Promoções · Métricas
- `node_media` — **Mídia** · Vídeos · Player · Paywall
- `node_lms` — **Academia LMS** · Cursos · Lives · Fórum · Biblioteca
- `node_gov` — **Governança** · Timeline · Plano Diretor · Compliance
- `node_connect` — **Connect Me** · Marketplace Unidirecional
- `node_profile` — **Perfis** · Visibilidade · Busca · Status
- `node_service` — **Serviços** · Propostas · Contratos · Validação

## Edges (13)

- `node_condo` → `node_gov` — _tenant context_
- `node_gov` → `node_service` — _Timeline imutável_
- `node_connect` → `node_service` — _oportunidades_
- `node_service` → `node_gov` — _validação síndico_
- `node_media` → `node_profile` — _conteúdo_
- `node_lms` → `node_gov` — _integração_
- `node_assembly` → `node_gov` — _resultados_
- `node_notify` → `node_gov` — _eventos_
- `node_notify` → `node_connect` — _eventos_
- `node_notify` → `node_assembly` — _eventos_
- `node_iam` → `node_gov` — _autenticação_
- `node_iam` → `node_connect` — _plano define acesso_
- `node_iam` → `node_profile` — _plano define visibilidade_

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]