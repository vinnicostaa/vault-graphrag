---
title: MOC — UI Catalog Marketing / Agência (MK1-MK8)
type: moc
tags: [master-sindico, ui-catalog, web, marketing, moc]
project: master-sindico
persona: marketing
created: 2026-04-23
---

# MOC — UI Catalog Marketing / Agência

Mapa de conteúdo para as **8 telas da Jornada da Agência de Marketing** (MK1-MK8). Persona: `marketing` (login Zitadel próprio — D-059/D-060). Plan tier: any (a agência não compra plano; é convidada por empresa Plus/Pro).

## Telas (8)

| ID | Nome | Sub-produto | Plan |
|---|---|---|---|
| [[MK1]] | Painel da Agência (5 cards) | marketing-agencias | any |
| [[MK2]] | Empresas Atendidas | marketing-agencias | any |
| [[MK3]] | Gerenciar Empresa (switch contexto) | marketing-agencias | any |
| [[MK4]] | Publicar Vídeo (12 tipos + checkbox assembleia) | marketing-agencias | any |
| [[MK5]] | Biblioteca de Vídeos (editar/ocultar) | marketing-agencias | any |
| [[MK6]] | Dashboard da Empresa (por empresa) | marketing-agencias | any |
| [[MK7]] | Dashboard Geral (consolidado) | marketing-agencias | any |
| [[MK8]] | Marketing Link Recebido | marketing-agencias | any |

## Regras transversais (todas as telas MK)

### Limitações duras (gates ABAC frontend + backend)

A agência **NÃO acessa**:
- ❌ Connect Me (qualquer direção)
- ❌ Registros de execução
- ❌ Comunicados técnicos
- ❌ Linha do Tempo do condomínio
- ❌ Dados de síndicos
- ❌ Dados de moradores
- ❌ Dashboard de governança do síndico

A agência **acessa apenas**:
- ✅ Vídeos institucionais das empresas que delegaram acesso
- ✅ Métricas agregadas dos próprios vídeos
- ✅ Marketing Links recebidos

### Modelo de delegação (D-059/D-060)

- **Login Zitadel próprio** (role `marketing`)
- **NÃO** é actor delegado por impersonation token
- **Grants ABAC scoped** por `agency_id` + `empresa_id`
- **Vínculo formal** só por convite empresa via [[../empresa/E14]]
- **Marketing Link ([[MK8]])** ≠ vínculo: apenas registra intenção; não cria delegação

### Visibilidade dos vídeos (regra crítica D-064)

- **Moradores NÃO acessam vídeos institucionais normalmente**
- Vídeos só aparecem aos moradores **durante votação de fornecedor em assembleia**
- Após votação encerrada, acesso volta a ser bloqueado
- Checkbox em [[MK4]] "Autorizar exibição em processos de escolha de fornecedor" controla esta visibilidade

### Continuidade

- **R3 — Ocultar não remove histórico**: vídeo oculto preservado no DB
- **Lock 90 dias** em vídeos publicados (D-066 — confirmar)
- **Empresa pode revogar acesso** a qualquer momento via [[../empresa/E14]] → agência vira "Encerrada" instantaneamente; vídeos publicados permanecem; gestão futura bloqueada

## Sub-produto relacionado

- [[../../../../00-product/sub-produtos/10-marketing-agencias]] — destilação canônica
- [[../../../../00-product/sub-produtos/04-conteudo-videos]] — sub-produto vídeos institucionais

## Fontes canônicas (leitura obrigatória)

- `vault/90-archive/master-sindico-pdfs-cliente-originais/JORNADA DA EMPRESA DE MARKETING.md` (MK1-MK8 + LIMITAÇÕES + INTEGRAÇÃO)
- `vault/90-archive/from-development-content-2026-04-23/02-Jornadas/Inventario-Telas.md` (linhas 147-167 — inventário canônico)
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Connect-Me-4-direcoes.md` (Marketing Link — feature distinta)
- `vault/50-projects/master-sindico/00-product/sub-produtos/10-marketing-agencias.md` (D-059/D-060 + 6 enums)

## Catálogo paralelo

- [[../empresa/_moc]] — 16 telas Empresa (E1-E16) — origina convite + Marketing Link
- [[../sindico/_moc]] — Síndico (não interage com agência diretamente)
- [[../morador/_moc]] — Morador (vê vídeos só em assembleia)

## Links

- [[../../../../CLAUDE]]
- [[../../ui-catalog]]
- [[../../../../client-canvas/Visibilidade Videos]]
- [[../../../../client-canvas/Modelo ABAC]]
- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Inventario-Telas]]

## Gaps consolidados (para futura iteração)

1. **Backend Go** ainda não tem módulo `marketing/` dedicado — feature distribuída em `content/` + `commercial/` + `identity/`
2. **Lock 90d** para edição de vídeos: política exata pendente (afeta todos os campos ou só tipo/título?)
3. **Onboarding agência** (primeiro acesso após aceitar convite): fluxo não está nesta entrega
4. **Múltiplos usuários da agência**: locking de edição simultânea + gestão de papéis internos pendente
5. **Anti-spam Marketing Link** por (empresa, agência): regra concreta pendente — sugerir 1/30d
6. **Tela perfil público da agência** (vitrine que origina [[../empresa/E16]]): fora desta entrega
