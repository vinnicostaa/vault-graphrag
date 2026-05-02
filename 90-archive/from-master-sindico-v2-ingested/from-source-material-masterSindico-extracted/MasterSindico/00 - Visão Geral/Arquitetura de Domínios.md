---
title: "Arquitetura de Domínios (canvas)"
type: canvas
tags: [canvas, obsidian, converted]
source: 50-projects/master-sindico/MasterSindico_extracted/MasterSindico/00 - Visão Geral/Arquitetura de Domínios.canvas
converted: 2026-04-22
---

# Arquitetura de Domínios

> Canvas Obsidian convertido. Estrutura original: JSON com nodes + edges.

## Nodes

- **group**: 💼 HIGH-DOMAIN: COMERCIAL
- **group**: 🏛️ HIGH-DOMAIN: INSTITUCIONAL
- **group**: ⚙️ CROSS-DOMAIN
- **group**: 📚 HIGH-DOMAIN: CONTEÚDO
- **group**: 🔐 HIGH-DOMAIN: IDENTIDADE
- **text**: **IAM** / Auth · ABAC · Planos · Billing
- **text**: **Assembleia** / Ao Vivo · Votação · Telão
- **text**: **Notificações** / Email · Push · In-App
- **text**: **Condomínio** / Unidades · Vínculos · Mandatos
- **text**: **Vizinhança** / Parceiros · Promoções · Métricas
- **text**: **Mídia** / Vídeos · Player · Paywall
- **text**: **Academia LMS** / Cursos · Lives · Fórum · Biblioteca
- **text**: **Governança** / Timeline · Plano Diretor · Compliance
- **text**: **Connect Me** / Marketplace Unidirecional
- **text**: **Perfis** / Visibilidade · Busca · Status
- **text**: **Serviços** / Propostas · Contratos · Validação

## Edges

- node_condo → node_gov (tenant context)
- node_gov → node_service (Timeline imutável)
- node_connect → node_service (oportunidades)
- node_service → node_gov (validação síndico)
- node_media → node_profile (conteúdo)
- node_lms → node_gov (integração)
- node_assembly → node_gov (resultados)
- node_notify → node_gov (eventos)
- node_notify → node_connect (eventos)
- node_notify → node_assembly (eventos)
- node_iam → node_gov (autenticação)
- node_iam → node_connect (plano define acesso)
- node_iam → node_profile (plano define visibilidade)
