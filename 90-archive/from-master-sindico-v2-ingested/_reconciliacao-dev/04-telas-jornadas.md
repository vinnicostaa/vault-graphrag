---
title: Reconciliação — Telas (141+) + Jornadas canônicas vs v2
type: note
tags: [reconciliation, telas, ui-catalog, jornadas, onboarding, master-sindico, v2, fase-7]
source: Content/02-Jornadas/{Inventario-Telas,Onboarding-por-Persona} + Content/03-Modulos/* + requirements.md Req 1-98.1 + ui-catalog.md v2
created: 2026-04-23
updated: 2026-04-23
---

# Reconciliação — Catálogo Exaustivo de Telas + Jornadas vs v2

> **Origem**: Agent Explore (Fase 7e) leu `Inventario-Telas.md`, `Onboarding-por-Persona.md`, os 4 módulos detalhados (`Assembleia-Live-Funcional`, `Compliance-Detalhado`, `Connect-Me-4-direcoes`, `Vizinhanca-4-jornadas`), `requirements.md` (Req 1-98.1, 3934 linhas), e v2 `ui-catalog.md` atual. Escopo: catálogo exaustivo das 141+ telas canônicas + 16 jornadas end-to-end.

---

## § 1. Inventário agregado — 141+ telas em 10 famílias

| Família | ID Range | Count | Descrição | Personas | v2 atual |
|---|---|---|---|---|---|
| **M** — Morador | M1-M15 | 15 | Home, seletor, onboarding, timeline, plano diretor, eventos, votação, vínculo | Morador Base/Plus/Pro | ~80% |
| **S** — Síndico | S1-S31 | 31 | Governança, PD, Timeline, eventos, Connect Me, validações, comunicados, dashboard, compliance, assembleia | Síndico N1/N2/N3 | ~70% |
| **E** — Empresa | E1-E16 | 16 | Painel, Connect Me, registros, dashboard, perfil, usuários, marketing | Empresa Trial/Base/Plus/Pro | ~90% |
| **MK** — Marketing | MK1-MK8 | 8 | Painel agência, empresas, conteúdo, biblioteca, dashboard, marketing link | Agência (actor delegado) | ~100% |
| **VS** — Vizinhança Síndico | VS1-VS10 | 10 | Painel vizinhança, explorar, promoções, benefícios exclusivos, parceiros indicados | Síndico | ~50% |
| **VE** — Vizinhança Empresa | VE1-VE6 | 6 | Explorar, promoções, histórico | Empresa | ~50% |
| **VM** — Vizinhança Morador | VM1-VM7 | 7 | Feed bairro, parceiros, promoções, benefícios exclusivos | Morador | ~40% |
| **VZ** — Vizinhança Parceiro | VZ1-VZ6 | 6 | Cadastro, perfil, promoção, métricas, histórico | Parceiro Vizinhança | ~30% |
| **LMS** — Academia | LMS1-LMS15 | 15 | Cursos, treinamentos, lives, fórum, biblioteca, certificados | Síndico/Empresa/Morador (NOT Parceiro) | **~0%** |
| **C** — Compliance | C1-C11 | 11 | Painel, declaração anual, assinaturas, conflitos, auditoria, risco, LGPD, score, dossiê | Síndico | **~0%** |
| **BT** — Banco Talentos | BT 01-11 | 11 | Vídeo-currículo, perfil, áreas de interesse, disponibilidade, pretensão, experiência, LGPD + empresa (busca, favoritos) | Morador / Empresa | **~0%** |
| **ASM** — Assembleia | (5 blocos, ~30+ telas) | 30+ | Criação, pauta, pré-assembleia (ciência, enquetes, procurações), ao vivo (check-in, votação, fala, telão), encerramento, pós-assembleia | Síndico, Administradora, Morador | ~40% |
| **TOTAL** | — | **141+** | — | — | **~60%** |

---

## § 2. Família M — Morador (M1-M15)

### Resumo:

| ID | Nome | Rota | Status v2 | Notas |
|---|---|---|---|---|
| M1 | Painel Morador | `/app/dashboard` | 80% | Home pessoal |
| M2 | Seletor Condomínios | `/app/condominios` | 90% | Lista cards |
| M3 | Buscar Condomínio | `/app/condominios/adicionar` | 100% | Busca por ID 8 chars |
| M4 | Cadastro Unidade | `/app/condominios/adicionar/:condId/unidade` | 100% | Tipo/bloco/vínculo/papel |
| M5 | Termo Ciência | `/app/condominios/adicionar/:condId/termo` | 100% | Versionado + LGPD |
| M6 | Status Unidade | `/app/condominios/:condId/unidade/:unitId/status` | 100% | 6 status + automações |
| M7 | **Avaliação Gestão (obrigatória 2x/ano)** | `/app/condominios/:condId/avaliacao-obrigatoria` | 60% | **Bloqueia M8 até completar** |
| M8 | Home Morador pós-onboarding | `/app/condominios/:condId/dashboard` | 100% | = M1 after complete |
| M9 | Linha do Tempo | `/app/condominios/:condId/timeline` | 90% | Somente `visibility='publico'` |
| M10 | Plano Diretor (leitura) | `/app/condominios/:condId/plano-diretor` | 90% | Leitura morador |
| M11 | Eventos | `/app/condominios/:condId/eventos` | 100% | Ciência + participação |
| M12 | Votação Fornecedor | `/app/condominios/:condId/votacao/:sessionId` | 70% | Vídeo grant desbloqueado durante votação |
| M13 | Meu Vínculo | `/app/conta/vinculo` | 30% | Editar/encerrar |
| M14 | Gerenciar Acessos (dependentes) | `/app/conta/acessos` | 40% | Até 2 dependentes |
| M15 | Encerrar Vínculo | (modal via M13) | 30% | Confirmação |

### Invariantes família M:
- **M7 bloqueia M8** — avaliação 2x/ano obrigatória (Req 15)
- **M9** mostra SOMENTE conteúdo `visibility='publico'`
- Morador NÃO publica (só lê)
- Dependentes NÃO podem alterar status ou remover titular
- M14: máx 2 dependentes por unit (INV-029)

---

## § 3. Família S — Síndico (S1-S31)

Núcleo da plataforma. 10 cards em S6. Síndico Trial = 15 dias com limites severos.

### Planos:
- **N1 (Base)**: visibilidade restrita, 2 Connect Me/ano, sem PD complexo, **sem publicar vídeos**
- **N2 (Plus)**: padrão completo, 4 Connect Me/ano, vídeo institucional 4/mês (lock 90d)
- **N3 (Pro)**: ilimitado, até 15 condomínios

### Resumo (principais):

| ID | Nome | Notas |
|---|---|---|
| S1 | Entrada Governança | Boas-vindas + [Selecionar] / [Cadastrar novo] |
| S2 | Meus Condomínios | Abas Ativos/Encerrados, máx 15 por síndico |
| S3 | Cadastrar Condomínio (Endereço+CEP) | Via ViaCEP Adapter (Req 66) |
| S4 | Dados Gestão (Mandato) | Tipo + datas; encerrar = gatilho Compliance |
| S5 | Vínculo Empresa Síndica | Convite por token; permissões configuráveis |
| **S6** | **Home da Governança** | **Dashboard 10 cards cross-módulo** |
| S7 | Plano Diretor (home) | 26 áreas + 26 tipos + 9 riscos |
| S8 | Cadastrar/Editar Ação PD | Voice input via Web Speech API |
| S9 | Lista Ações PD | = S7 possivelmente |
| S10 | Detalhe Ação PD | Risk score + histórico + timeline |
| S11 | Linha do Tempo (home) | Append-only, imutável |
| S12 | Criar Atividade (publish Timeline) | Confirmação anti-erro obrigatória |
| S13 | Editar Atividade | **Não permite edit direto** — só **adendo formal** |
| S14 | Histórico Timeline | Original + adendos lado-a-lado |
| S15 | Eventos (home) | 13 tipos, status, RSVP |
| S16 | Criar Evento | Notificar push+email+SMS |
| S17 | Lista Eventos | = S15 possivelmente |
| S18 | Connect Me (hub) | Quota anual; enviados/recebidos/histórico |
| S19 | Criar Connect Me (S→E) | Dados síndico OCULTOS até empresa "Tenho Interesse" |
| S20 | Connect Me Recebido (M→S) | 5 categorias; **unidirecional** |
| S21 | Registros Execução (hub) | Empresa submete; síndico valida |
| S22 | Detalhe Registro Execução | Validar / solicitar ajuste / rejeitar |
| S23-S25 | Comunicados (home, criar, lista) | 8 tipos canônicos + priority |
| **S26** | **Validações Pendentes (hub)** | **Tela-pivô: 5 abas (Registros, Comunicados, Procurações, Promoções, CM)** |
| S27 | Dashboard customizado | Widgets drag-and-drop |
| S28-S31 | Compliance (entrada, termo, declaração, auditoria) | **= C1, C2 overlap** (11 telas) |

### Invariantes família S:
- S6 + S26 são telas-pivô cross-módulo
- S13: sem edit direto — só adendo (Req 25)
- Limite duro: 15 condomínios (Decision 11)
- Síndico Trial: 1 condomínio só, [Publicar timeline] e [Criar comunicado] bloqueados
- PD status muda **automaticamente** via Timeline (Req 12 AC8) — síndico nunca atualiza manualmente

---

## § 4. Família E — Empresa (E1-E16)

Painel empresarial. Tenant independente.

### Planos:
- **Trial 7d**
- **Base** (soft-block após trial)
- **Plus**: responde Connect Me, 8 vídeos/mês
- **Pro**: tudo do Plus + Connect Me E→E + cupons Vizinhança + 12 vídeos/mês

### Resumo:

| ID | Nome | Notas |
|---|---|---|
| E1 | Painel Empresarial | 10 cards |
| E2 | Oportunidades (Connect Me) | Síndico OCULTO até "Tenho Interesse" |
| E3 | Recusar Oportunidade | 6 motivos estruturados |
| E4 | Detalhe Oportunidade | Síndico revelado + LGPD log gravado |
| E5 | Registrar Execução | Rascunho → enviado → validado síndico |
| E6 | Atividade Técnica | Similar E5 mas sem service formal |
| E7 | Comunicados Técnicos | Publicar + (opcional) validar síndico |
| E8 | Status Envios | Hub: aguardando/validados/rejeitados |
| E9 | Detalhe Registro | Editar se rejeitado |
| E10 | Dashboard Empresarial | Propostas ganhas/perdidas, avaliação média |
| E11 | Perfil Institucional | Dados onboarding (Req 6.3) |
| E12 | Usuários Empresa | Administrador / Operador Técnico |
| E13 | Gestão Agência | Agências vinculadas |
| E14 | Convidar Agência | Email + token |
| E15 | Gerenciar Acessos Agência | Publicar vídeos, gerenciar biblioteca |
| E16 | **Marketing Link** | **Formulário, NÃO cria vínculo automático** — registro de intenção |

### Invariante crítica:
- E16 **Marketing Link ≠ Connect Me**: formulário de interesse; vínculo só via E13 convite formal
- Tabela backend: `marketing_link_requests` (separada de ConnectMe)

---

## § 5. Família MK — Marketing/Agência (MK1-MK8)

Agência = actor delegado. Não é tenant próprio. Impersona via scoped token.

| ID | Nome | Descrição |
|---|---|---|
| MK1 | Painel Agência | Dashboard multi-empresa |
| MK2 | Seletor Empresa | Scoped token ativo por 1 empresa |
| MK3 | Biblioteca Conteúdos | Vídeos + templates por empresa |
| MK4 | Produzir Vídeo | Upload + metadados + publicar |
| MK5 | Dashboard Analytics | Métricas por empresa (delegada) |
| MK6 | Campanhas Marketing Link | Gestão de oferta recebida |
| MK7 | Relatórios Consolidados | Export PDF/CSV cross-empresa |
| MK8 | Audit Log Delegation | O que fez em nome de quem, quando |

---

## § 6. Famílias Vizinhança (VS, VE, VM, VZ) = 29 telas

### VS — Vizinhança Síndico (10)
VS1 Painel Vizinhança, VS2 Explorar, VS3 Benefícios Moradores, VS4 Convites, VS5 Benefícios Exclusivos, VS6 Parceiros Indicados, VS7 Histórico, VS8 Moderação, VS9 Seal, VS10 Métricas

### VE — Vizinhança Empresa (6)
VE1-VE6: Explorar parceiros + promoções + histórico de contratos

### VM — Vizinhança Morador (7)
VM1-VM7: Feed do bairro + parceiros próximos + promoções exclusivas + benefícios

### VZ — Vizinhança Parceiro (6)
VZ1 Cadastro, VZ2 Perfil, VZ3 Promoção 4h lock, VZ4 Palavra do Dia, VZ5 Métricas, VZ6 Histórico

### Regras:
- Promoção aberta bairro vs exclusiva condomínio
- Lock 4h rigoroso em cupom
- Palavra-chave do dia (gamification Pro)
- Seal "Síndico-aprovado" via votação leve
- Parceiro NÃO recebe Connect Me aberto

---

## § 7. Família LMS — Academia (LMS1-LMS15) — **0% em v2**

| ID | Nome |
|---|---|
| LMS1 | Home Academia |
| LMS2 | Trilhas (Síndico/Empresa/Morador) |
| LMS3 | Catálogo Cursos |
| LMS4 | Detalhe Curso |
| LMS5 | Player Aula |
| LMS6 | Quiz |
| LMS7 | Certificado gerado |
| LMS8 | Fórum |
| LMS9 | Biblioteca (guias, manuais, ebooks) |
| LMS10 | Lives (abertas + exclusivas) |
| LMS11 | Lançamento de solução |
| LMS12 | Debate |
| LMS13 | Palestra técnica |
| LMS14 | Entrevista |
| LMS15 | Meu progresso |

### Regras:
- 10 categorias de curso
- 3 trilhas: síndico, empresa, morador
- 2 níveis acesso: gratuito / pago
- Parceiro **NÃO** tem acesso

---

## § 8. Família C — Compliance (C1-C11) — **0% em v2**

| ID | Nome |
|---|---|
| C1 | Painel Compliance (status geral) |
| C2 | Declaração Anual |
| C3 | Assinaturas Pendentes |
| C4 | Conflitos de Interesse |
| C5 | Auditoria (trilha) |
| C6 | Imutabilidade (adendos) |
| C7 | Risk Map (8 tipos risco, 4 níveis) |
| C8 | Conformidade Contratual |
| C9 | LGPD (lgpd_logs) |
| C10 | Score Compliance (0-100) |
| C11 | Dossiê Exportável (PDF) |

### 4 gatilhos que tornam Compliance OBRIGATÓRIO:
1. Encerrar mandato
2. Gerar relatório final
3. Marcar negócio fechado
4. Exportar dossiê

---

## § 9. Família BT — Banco de Talentos (BT 01-11) — **0% em v2**

| ID | Nome |
|---|---|
| BT01 | Home Banco de Talentos |
| BT02 | Perfil profissional (18 áreas de interesse) |
| BT03 | Vídeo-currículo (90s, lock 90d) |
| BT04 | Áreas de Interesse (18 opções) |
| BT05 | Modalidade Contratação (9 opções) |
| BT06 | Disponibilidade (4 horários) |
| BT07 | Pretensão salarial |
| BT08 | Experiências (5 vínculos máx, Decision 6) |
| BT09 | Formação + CNH + NR |
| BT10 | Termo LGPD |
| BT11 | Confirmação |

### Empresa view:
- Busca por área + CEP + raio
- Favoritos
- Tags
- Convite via Connect Me fluxo especial

---

## § 10. Família Assembleia (5 blocos, 30+ telas) — ~40% em v2

### Bloco 1: Criação (Síndico)
- Novo Assembleia → tipo (ordinária/extraordinária/implantação/ratificação)
- Pauta (agenda items)
- Convocação (edital PDF via gov.br Lei 14.063)

### Bloco 2: Pré-Assembleia
- Publicação edital
- Ciência moradores
- Enquete sim/não (pré-voto)
- Procurações (outorga)
- Fração ideal validation
- Inadimplência check

### Bloco 3: Ao Vivo (WebSocket)
- Check-in (app / QR code / terminal presencial)
- Telão 2 áreas (pauta + fila de fala)
- Votação real-time por fração ideal
- Fila de fala cronometrada (2 ou 3 min)
- Modo contingência (3 razões: internet/device/digital unavailable)

### Bloco 4: Encerramento
- Homologação quórum
- Geração automática ata
- Assinatura digital gov.br

### Bloco 5: Pós-Assembleia
- Ata imutável publicada
- Relatório executivo
- Histórico
- Timeline entry

### Invariantes assembleia:
- `AssemblyStatus`: em_preparacao → publicada → em_andamento → encerrada
- Vote único por (agenda_item, voter) — UNIQUE DB (A-025 ✅)
- Minutes imutável pós-publish (INV-041)
- Modalidade MVP: presencial; hibrida/online preparado mas inativo

---

## § 11. 16 Jornadas End-to-End (canônicas)

1. Onboarding Morador (4 passos)
2. Onboarding Síndico (6 passos)
3. Onboarding Empresa (7 passos)
4. Onboarding Parceiro (4 passos)
5. Votação em Assembleia (5 etapas)
6. Connect Me Síndico→Empresa
7. Connect Me Morador→Síndico
8. Connect Me Empresa→Empresa (Plus/Pro)
9. Connect Me Síndico→Parceiro
10. Upload Vídeo Institucional
11. Upload Vídeo-Currículo Morador (BT)
12. Publicação Cupom Vizinhança (4h lock)
13. Convocação + Realização Assembleia Live
14. Trilha LMS + Certificado
15. Compliance Gatilho C1-C11
16. Harassment Report + Moderação
17. Exclusão LGPD (DELETE /me)

---

## § 12. Matriz de gaps vs v2 atual

| Métrica | Canônico | v2 atual | Gap | Crítico? |
|---|---|---|---|---|
| Total telas | 141+ | ~90 | **51+** | 🟡 |
| Compliance C1-C11 | 11 | 0 | **11** | 🔴 |
| LMS1-LMS15 | 15 | 0 | **15** | 🔴 |
| BT 01-11 | 11 | 0 | **11** | 🔴 |
| Assembleia WebSocket/real-time | 30+ | ~10 | 20+ | 🔴 |
| Vizinhança (VS+VE+VM+VZ) | 29 | ~8 | 21 | 🟡 |
| Morador M1-M15 | 15 | 12 | 3 | 🟢 |
| Síndico S1-S31 | 31 | 22 | 9 | 🟡 |
| Empresa E1-E16 | 16 | 14 | 2 | 🟢 |
| Marketing MK1-MK8 | 8 | 8 | 0 | ✅ |

---

## § 13. Arquivos v2 que precisam re-destilação/expansão

1. `03-subprojects/web/ui-catalog.md` — expandir de 90 pra 141+ telas
2. `03-subprojects/mobile/ui-catalog.md` — subset mobile claro
3. `00-product/personas.md` — corrigir jornadas (remover N1/N2/N3 tier, usar slugs)
4. `04-requirements/functional/*.md` — adicionar telas por BC
5. **NOVO**: `04-requirements/functional/compliance.md` (C1-C11 + 4 gatilhos)
6. **NOVO**: `04-requirements/functional/lms.md` (LMS1-LMS15)
7. **NOVO**: `04-requirements/functional/banco-talentos.md` (BT 01-11)
8. `04-requirements/functional/assembly.md` — expandir com 5 blocos + WebSocket real-time + telão + fila fala

---

## § 14. Conclusões

- v2 é **~60% do canônico** (90/141 telas)
- **3 famílias inteiras faltam**: Compliance, LMS, Banco Talentos = **37 telas críticas**
- Assembleia precisa expansão WebSocket + real-time (telão, votação, check-in, fila de fala cronometrada)
- Vizinhança tem ~70% vs canônico (faltam telas empresa/parceiro)
- Síndico ~70%: faltam Compliance (S28-S31 = C1-C11)
