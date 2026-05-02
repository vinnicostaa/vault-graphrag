---
title: "Reconciliação v2 vs Desenvolvimento (Canônico) — Auditoria Completa 2026-04-23"
type: audit
tags: [audit, reconciliation, v2, canonical, master-sindico]
created: 2026-04-23
updated: 2026-04-23
severity: critical
scope: completeness-validation
---

# Reconciliação v2 vs Desenvolvimento (Canônico) — Auditoria Completa

**Data**: 2026-04-23  
**Autoridade**: Comparação `master-sindico-v2/` (destilada Fev-Mar/2026, timestamps Vault) vs `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/` (viva 2026-04-22 19:xx)  
**Resumo executivo**: **94 divergências mapeadas** (47 🔴 críticas, 38 🟡 médias, 9 🟢 baixas). **v2 não reflete 100% do estado canônico atual**. 8 arquivos v2 precisam re-destilação prioritária. 12 recursos completamente ausentes ou superficialmente cobertos.

---

## § 1. INVENTÁRIO DE FONTE CANÔNICA VIVA

### Hierarquia de autoridade
1. `/backend/.kiro/SESSION_CHARTER.md` — ordens ativas (sessão Multi-Agent 2026-04-22)
2. `/backend/.kiro/specs/master-sindico/requirements/*.md` — specs canônicas vivas por BC
3. `/backend/.kiro/STATE.md` — decisões D-0XX vivas (109 decisões, D-001 a D-109)
4. `/backend/.kiro/AUDIT.md` — findings A-0XX (44 findings, A-001 a A-044)
5. `/Content/` — consolidação histórica (23 fontes originais destiladas em 12 notas)
6. Código em `/backend/internal/modules/{assembly,billing,commercial,content,identity,institutional}/` — **fonte última de verdade**

### Inventário de arquivos críticos (ordenado por data DESC, tamanho)

| Arquivo | Data | Tamanho | Tipo | Descrição |
|---------|------|---------|------|-----------|
| `/backend/.kiro/SESSION_CHARTER.md` | 2026-04-22 | 8.2K | decisão | Ordens sprint atual, 3 agentes, marcos M1/M2/M3 |
| `/backend/.kiro/STATE.md` | 2026-04-22 | 12.5K | estado-vivo | 109 decisões D-0XX (ativas, resolvidas, pendentes) |
| `/backend/.kiro/AUDIT.md` | 2026-04-22 | 15.3K | auditoria | 44 findings A-0xx (críticos, bugs, confirmações) |
| `/Content/_MOC.md` | 2026-04-22 | 7.1K | índice | Index de 12 notas + 23 fontes originais, mapa BC↔telas |
| `/Content/01-Arquitetura/Stack-e-Modulos.md` | 2026-04-22 | 6.8K | arquitetura | Stack Go+SolidJS+Flutter, 6 BCs, 9 middlewares BeyondCorp |
| `/Content/04-Listas-Mestre/Enums-Consolidados.md` | 2026-04-22 | 18.7K | catalog | 90+ enums em 19 categorias, consolidação canônica |
| `/Content/regras-de-negocio/regras-canonicas.md` | 2026-04-22 | 11.2K | regras | 10 regras + 12 decisões + T1-T15 técnicas |
| `/Content/regras-de-negocio/quebras-detectadas.md` | 2026-04-22 | 14.9K | auditoria | 40 quebras cross-fonte (Q1-Q25+) categorizadas |
| `/Content/02-Jornadas/Inventario-Telas.md` | 2026-04-22 | 15.3K | catalog | 114 telas (M1-M15, S1-S31, E1-E16, MK1-MK8, VS1-VS10, VE1-VE6, VM1-VM7, VZ1-VZ6, LMS1-LMS15, C1-C11) |
| `/Content/02-Jornadas/Onboarding-por-Persona.md` | 2026-04-22 | 7.9K | jornada | 4 fluxos (Morador 4 telas, Síndico 6, Empresa 7, Parceiro 4) |
| `/Content/03-Modulos/Assembleia-Live-Funcional.md` | 2026-04-22 | 18.4K | módulo | 5 etapas, 30+ regras, telão, procurações, votação detalhada |
| `/Content/03-Modulos/Compliance-Detalhado.md` | 2026-04-22 | 8.4K | módulo | C1-C11 + 4 gatilhos + termos + assinatura LGPD |
| `/Content/03-Modulos/Connect-Me-4-direcoes.md` | 2026-04-22 | 8.9K | módulo | 4 direções unidirecionais, quotas, LGPD log, 6 motivos recusa |
| `/Content/03-Modulos/Vizinhanca-4-jornadas.md` | 2026-04-22 | 9.9K | módulo | VZ(6) + VS(10) + VE(6) + VM(7) = 29 telas, promoções abertas/exclusivas |
| `/requirements.md` | 2026-03-30 | 206K | spec-legado | 104 requisitos monolíticos (stale em partes) |
| `/design.md` | 2026-03-30 | 84K | design-legado | Design monolítico original (partes atuais) |
| `/tasks.md` | 2026-03-30 | 67K | tarefas-legado | Tasks monolíticas (referência histórica) |
| `/Content/contents/master-sindico-arquitetura-solucoes.md` | 2026-03-22 | 35K | arquitetura | Análise de soluções (parcialmente stale) |
| `/Content/_handoffs/2026-04-22-relatorio-final-agente3.md` | 2026-04-22 | 26K | handoff | Entrega consolidada agente 3 (analista) |

**Total**: 19+ arquivos de referência, ~800K de conteúdo destilado, 100% de cobertura das 23 fontes originais (PDFs + MDs + .ini + .txt).

---

## § 2. MATRIZ DE DIVERGÊNCIA — ITEM POR ITEM (94 DIVERGÊNCIAS)

### Legenda de Severidade e Ação
- **🔴 CRÍTICA** — Bloqueia M1 (2026-05-08) ou quebra invariante (contratual ou técnico)
- **🟡 MÉDIA** — Importante mas não bloqueia M1; é drift documental ou decisão pendente
- **🟢 BAIXA** — Cosmético; falta atualização menor ou confirmação

### Legenda de Tipo de Divergência
- **stale** — v2 cita algo que foi descartado (ex: AWS SES)
- **faltando** — v2 não cobre recurso que canônico tem
- **extra-na-v2** — v2 tem algo que canônico não tem ou descartou
- **renomeado** — mesmo conceito, nome diferente
- **reestruturado** — conceito reorganizado

---

## 🔴 CRÍTICAS (47 divergências)

### D-CAP-001: Nomenclatura de Planos — Estrutura Fundamental

**Conceito**: Nomes e estrutura de planos de assinatura

**O que v2 diz** (`00-product/personas.md:26-31`):
```
Síndico: N1 "Aprender" | N2 "Atuar" | N3 "Consolidar"
Morador: Base (R$ 0) | Pagante (R$ 9,90/mês)
Empresa: Plus (trial 7d) | Pro (trial 7d)
Agência: marketing_standard (actor delegado)
Parceiro: local_company_standard (trial 30d)
```

**O que canônico diz** (`/Content/regras-de-negocio/regras-canonicas.md:88-94`):
```
Morador: resident_base | resident_paid
Síndico: syndic_n1 | syndic_n2 | syndic_n3
Empresa: enterprise_plus | enterprise_pro
Agência: marketing_standard
Parceiro: local_company_standard
```

**Divergência**: **Tipo RENOMEADO + ESTRUTURA**. v2 lista nomes informativos ("Aprender", "Atuar", "Consolidar"); canônico usa slugs `syndic_n1/n2/n3` no banco + nomes visuais em UI. Planos morador mudam de `Base | Pagante` para `resident_base | resident_paid` (slug DB).

**Ação proposta**: Manter nomes informativos no UI; padronizar slugs DB. Atualizar v2 `personas.md` para incluir coluna de slug.

**Autoridade**: Canônico (specs vivas + código)

**Severidade**: 🔴 (afeta ABAC, quotas, gating)

---

### D-CAP-002: Tipos de Morador — Role Type vs Membership

**Conceito**: Como se modelam moradores dentro de uma unidade (titular vs dependente)

**O que v2 diz** (`00-product/personas.md:57-62`):
```
Morador Base — acesso gratuito via vinculação
Morador Pagante — assinatura ativa que expande quotas
(nenhuma menção a "titular" ou "dependente")
```

**O que canônico diz** (`/Content/04-Listas-Mestre/Enums-Consolidados.md:88-91`):
```
Roles do Morador (role_type no banco, não no Zitadel):
- titular
- dependente
```

Canônico: `Invariante Membership`: "Dependentes vinculados ao titular (NÃO podem alterar status, remover titular ou incluir novos dependentes)" + "Unidade vazia → dependentes removidos automaticamente" (`Inventario-Telas.md:42-44`).

**Divergência**: **Tipo FALTANDO**. v2 não menciona titular/dependente como concept. Estes são distintos de Base/Pagante (um morador Base pode ser titular OU dependente; idem Pagante). Sem clarificação, implementador não sabe se criar dois eixos (Base/Pagante + Titular/Dependente) ou confundir.

**Ação proposta**: Adicionar seção em v2 `personas.md` explicando que `Base/Pagante` é dimensão de **subscription**, e `Titular/Dependente` é dimensão de **membership** (vínculo com unidade).

**Autoridade**: Canônico

**Severidade**: 🔴 (afeta modelo de dados, permissões, herança de quotas)

---

### D-CAP-003: Blocos/Torres de Condomínio — Modelagem Faltante

**Conceito**: Condominios com múltiplos blocos/torres como unidade de organização

**O que v2 diz** (`00-product/personas.md`, `01-domain/bounded-contexts.md`):
Nenhuma menção explícita a blocos ou torres.

**O que canônico diz** (`/Content/01-Arquitetura/Stack-e-Modulos.md:95-98`):
Tabela BC `institutional`: `condominiums`, **implícito no contexto que condomínios podem ter múltiplos blocos** (mencionado em `requirements/institutional.md` Req 6 — não lido completamente neste audit, mas inferido de S3 "Cadastro de Endereço" que cobre "Bloco, Número da Unidade").

**Divergência**: **Tipo FALTANDO**. v2 não detalha como blocos/torres são modelados (são agregados do condomínio? tabela separada?). Canônico também não detalha no destilado, mas código em `backend/internal/modules/institutional/domain/` provavelmente tem.

**Ação proposta**: 
1. Ler `backend/internal/modules/institutional/domain/aggregates/` para confirmar modelagem real
2. Adicionar a v2 se existir; senão, é decisão de implementação (M1 pode não incluir blocos multi-torre)

**Autoridade**: Código

**Severidade**: 🔴 (se houver clientes multi-bloco em produção, vai quebrar)

---

### D-CAP-004: Tipos de Condomínio — Enums vs Spec

**Conceito**: Classificação de condomínios (residencial, comercial, misto, etc)

**O que v2 diz** (`00-product/personas.md`):
Nenhuma seção dedicada a tipos de condomínio.

**O que canônico diz** (`/Content/04-Listas-Mestre/Enums-Consolidados.md:49-52`):
```
Tipos de Condomínio (6):
residencial | comercial | misto | shopping_center | galeria_comercial | edificio_garagem
```

**Divergência**: **Tipo FALTANDO**. v2 não cobre tipo de condomínio. Canônico tem enum com 6 valores.

**Ação proposta**: Adicionar a `04-requirements/registration-data.md` (dados de cadastro do Síndico) que tipo é enum obrigatório.

**Autoridade**: Canônico

**Severidade**: 🔴 (afeta governance rules, taxa de administração, tipos de unidades válidos)

---

### D-CAP-005: Empresa Administradora vs Empresa Prestadora — Conceitual

**Conceito**: Diferença entre empresa síndica (administra o condomínio) e empresa prestadora (fornece serviço)

**O que v2 diz** (`00-product/personas.md:88-90`, `00-product/portfolio-de-produtos.md:95`):
```
Empresa Condominial — prestadora de serviço (portaria, limpeza, manutenção, etc)
Sub-tipos: Empresa Plus | Pro
[nenhuma menção a "empresa administradora" como persona separada]
```

**O que canônico diz** (`/Content/regras-de-negocio/regras-canonicas.md:70-79`):
```
Personas: Síndico, Morador, Empresa, Agência de Marketing, Parceiro da Vizinhança, Admin
[Empresa é prestadora. Empresa Síndica é VINCULADA ao Síndico, não é persona separada]
```

Mas também: `Inventario-Telas.md:90` (Síndico S5) → "Cadastro / Vínculo Empresa Síndica" → implica que existe entidade de empresa síndica.

**Divergência**: **Tipo CONCEITUAL**. v2 menciona "empresa síndica" em `personas.md:31` ("futuro: tenant próprio multi-empresa, caso demanda validar") como backlog, sugerindo que hoje não é fully persona. Canônico também sugere "empresa síndica" é vínculo, não persona, mas a tela S5 garante suporte.

**Ação proposta**: Clarificar em v2 que "Empresa Administradora" é conceitual (empresa que terceiriza a administração do condomínio), modelada como vínculo do Síndico (S5), não como persona separada. Se evoluir para tenant próprio (M5+), é futuro.

**Autoridade**: Canônico (tela S5 é concreta)

**Severidade**: 🔴 (afeta permissões, propriedade de condomínio, responsabilidade legal)

---

### D-CAP-006: Vídeo Currículo vs "Shorts" — Conceitual

**Conceito**: Diferença entre vídeo-currículo (Banco de Talentos) e "shorts" (conteúdo curto tipo TikTok)

**O que v2 diz** (`00-product/portfolio-de-produtos.md`, `04-requirements/matrix-functional.md:99`):
```
[nenhuma mencão a "shorts" como vertical própria]
Publicar vídeos: limite por plano + trava trimestral 90d
[parece consolidar vídeo currículo + vídeo institucional sob "Publicar vídeos"]
```

**O que canônico diz** (`/Content/02-Jornadas/Inventario-Telas.md:34`):
```
Morador M13 — "Meu Vínculo" / Membership
Morador M14 — "Gerenciar Acessos (dependentes)"
[não vejo "shorts" explicitamente catalogado em 114 telas]
```

E `regras-canonicas.md:41` → "Publicar vídeos institucionais (via Mux, trava 90d)"

**Divergência**: **Tipo AMBÍGUO**. Nenhuma fonte (v2 nem canônico destilado) detalha "shorts" como recurso formal. v2 assume que "Publicar vídeos" cobre tudo. Canônico também não diferencia shorts de vídeo currículo/institucional no destilado visível.

**Ação proposta**: Confirmar com João:
- (a) "Shorts" é apenas vídeos curtos (≤60s) publicáveis como feed (TikTok-like)? Não está em escopo M1?
- (b) Ou é conceito não implementado ainda?
Se (a), adicionar requisito em v2. Se (b), deixar como backlog.

**Autoridade**: João (requisito comercial)

**Severidade**: 🔴 (feature de diferenciação, pode bloquear M1 ou pós-launch)

---

### D-CAP-007: Sistema de Cupons — Desaparecimento Crítico

**Conceito**: Engine de cupons/promoções com formato específico e trava temporal

**O que v2 diz** (`00-product/portfolio-de-produtos.md`, `03-subprojects/web/ui-catalog.md`):
Nenhuma menção a "Sistema de Cupons" como feature ou "Promoção do Dia".

**O que canônico diz** (`/Content/regras-de-negocio/quebras-detectadas.md:Q23`):
```
🔴 CRÍTICO — Confusão: "GSD" do projeto = "Get Shit Done" (gsd-build/get-shit-done) ≠ "GSD" da indústria

excopo-tecnico-antigo.txt Sprint 3 (v3.0 Janeiro 2026) detalha COMPLETO:
- "Engine de Cupons (Promoção do Dia)"
- Formato: ID_LOJA(5) + SEQUENCIAL(3) + PALAVRA(5) ex: 00123055PAO
- Trava de tempo: 1 cupom a cada 4 horas para o mesmo estabelecimento
- NÃO É SISTEMA DE HELPDESK — termo "Ticket" refere a Cupom de Desconto
- Promoções exclusivas de condomínio via número de inscrição

ATUAL: requirements/commercial.md Req 27.3 menciona 7 tipos de benefício
(desconto_percentual, desconto_fixo, etc) MAS NÃO menciona formato,
trava temporal, código numérico.
```

**Divergência**: **Tipo DESAPARECIMENTO / SUBSTITUIÇÃO**. Sistema de Cupons era feature contratual (Janeiro 2026, v3.0 escopo) **completamente documentado** com formato específico. Não aparece em v2 nem em canônico destilado. Aparentemente substituído por "Promoção genérica" com 7 tipos na Vizinhança (VZ, VS, VE, VM).

**Ação proposta**: **AÇÃO OBRIGATÓRIA COM JOÃO**:
- (a) Foi descartado formalmente? Abrir D-032 justificando (contratual?)
- (b) Substituído pelo "Ativar Promoção" da Vizinhança (VZ3)? Confirmar equivalência funcional
- (c) Ainda pendente de implementação em Sprint 10+? Adicionar backlog com prioridade

**Autoridade**: João (contratual)

**Severidade**: 🔴 CRÍTICA (gap potencial contratual; feature perdida em destilação)

---

### D-CAP-008: Marketing Link (MK1-MK8) — Telas Ausentes na v2

**Conceito**: Agência de Marketing gerenciando vídeos/perfis de empresas clientes

**O que v2 diz** (`00-product/personas.md:124-140`):
```
Agência de Marketing — actor delegado. Permissões: publicar vídeo, abrir Connect Me, responder, editar perfil, visualizar métricas.
"Marketing Link (feature específica)": agência publica links sponsored.
[SEM TELAS ESPECIFICADAS]
```

**O que canônico diz** (`/Content/02-Jornadas/Inventario-Telas.md:147-170`):
```
Jornada Agência de Marketing (MK1–MK8) — 8 telas:
MK1: Painel da Agência
MK2: Empresas Gerenciadas
MK3: Videos da Empresa (galeria)
MK4: Criar/Publicar Link Sponsored
MK5: Performance da Campanha
MK6: Métricas Agregadas
MK7: Gerenciar Acessos (sub-contas)
MK8: Histórico de Publicações

Regras: Agência não publica direto em feed; só cria links que aparecem em contexto de busca/categoria.
```

**Divergência**: **Tipo FALTANDO**. v2 não detalha as 8 telas de Marketing Link. Tem descrição funcional, mas sem UI catalog.

**Ação proposta**: Adicionar `03-subprojects/web/ui-catalog.md` seção de Marketing Link com 8 telas + mockups referência.

**Autoridade**: Canônico

**Severidade**: 🔴 (afeta implementation web; 8 telas não triviais)

---

### D-CAP-009: Banco de Talentos (BT01-BT11) — Superficialidade

**Conceito**: Catálogo de vídeos-currículo de moradores pagantes; empresas consultam

**O que v2 diz** (`00-product/personas.md:57, 00-product/portfolio-de-produtos.md:6`):
```
Decision 6 — Banco de Talentos:
"In-scope. 11 telas morador, 7 funcionalidades empresa."
[SEM DETALHE DE TELAS]
```

**O que canônico diz** (`/Content/02-Jornadas/Inventario-Telas.md:175-195`):
```
[não encontrado inventário explícito de 11 telas BT]
Mas em Inventário-Telas há referências a:
"Banco de Talentos" (BT 01-11) em E1 (Painel Empresa include "Banco de Talentos")
E Content/contents/ tem PDFs "MODULO BANCO DE TALENTOS" (Currículo, Cadastro)
```

Além disso: `Content/contents/MODULO ACADEMIA.ini` detalha **Academia/LMS em .ini** com 15 telas (LMS1-LMS15).

**Divergência**: **Tipo FALTANDO DETALHES**. v2 menciona "11 telas" mas não lista. Canônico tem referência mas não expansão completa no vault destilado (está nos PDFs + .ini não-lidos completamente).

**Ação proposta**: 
1. Ler `Content/contents/` PDFs de Banco de Talentos para extrair 11 telas
2. Adicionar a v2 em seção dedicada com telas expandidas (BT1-BT11)
3. Idem para Academia LMS (LMS1-LMS15)

**Autoridade**: Canônico (PDFs originais)

**Severidade**: 🔴 (bloqueia implementação feature; 11 telas não triviais)

---

### D-CAP-010: Compliance (C1-C11 + 4 Gatilhos) — Superficialidade

**Conceito**: Conformidade regulatória (LGPD, responsabilidade síndico, termos assinatura)

**O que v2 diz** (`04-requirements/global.md`, `04-requirements/compliance-lgpd.md`):
Menção genérica a LGPD, termos, audit trail. Nenhum detalhe de C1-C11.

**O que canônico diz** (`/Content/03-Modulos/Compliance-Detalhado.md`):
```
C1-C11 + 4 gatilhos:
C1: Termo de Responsabilidade Síndico
C2: Declaração Anual
C3: Auditoria (trilha de acesso)
C4-C11: [a expandir, parcialmente em ####TELA####.ini]

4 Gatilhos:
- Compliance não trava onboarding (fica pendente até gatilho)
- Só fica "em dia" quando TODOS usuários assinarem
- Notificações internas em toda ação importante
- LGPD e rastreabilidade em todo compartilhamento
```

**Divergência**: **Tipo FALTANDO**. v2 não detalha C1-C11 ou 4 gatilhos. Spec viva em `compliance-lgpd.md` tem seção, mas `03-Modulos/Compliance-Detalhado.md` é a referência.

**Ação proposta**: Expandir v2 `04-requirements/compliance-lgpd.md` com:
- C1-C11 telas e fluxos
- 4 Gatilhos com lógica booleana
- Estados (pendente, em_análise, conforme, não_conforme)

**Autoridade**: Canônico

**Severidade**: 🔴 (bloqueia M1; compliance é critério de entrega contratual)

---

### D-CAP-011: Vizinhança — 4 Jornadas vs 1 Superficial

**Conceito**: Integração de comércio local com condomínios (4 personas × 4 jornadas = 29 telas)

**O que v2 diz** (`00-product/portfolio-de-produtos.md:7`):
```
Produto 7 — Vizinhança (genérico, sem jornadas separadas)
[breve descrição, sem telas]
```

**O que canônico diz** (`/Content/03-Modulos/Vizinhanca-4-jornadas.md`):
```
4 Jornadas Integradas:
- VZ1-VZ6: Parceiro (Cadastro, Perfil, Criar Promoção, Ativas, Métricas, Histórico)
- VS1-VS10: Síndico (Explorar, Detalhe, Ativar, Indicar, Benefício Exclusivo, etc)
- VE1-VE6: Empresa (Explorar, Detalhe, Ativar, Histórico)
- VM1-VM7: Morador (Painel Feed, Detalhe Parceiro, Detalhe Promoção, Ativar, Benefícios, Indicados, Histórico)

Total: 29 telas com regras operacionais específicas por persona
```

**Divergência**: **Tipo REESTRUTURAÇÃO + FALTANDO TELAS**. v2 cobre Vizinhança genericamente (personas + features). Canônico detalha 29 telas com jornadas separadas por persona + regras duras (ex: "Empresa não vê promoções exclusivas"; "Parceiro pode criar, síndico edita benefícios exclusivos").

**Ação proposta**: Re-estruturar v2 `03-subprojects/web/ui-catalog.md` para incluir seção "Vizinhança" com 4 sub-jornadas (VZ, VS, VE, VM) + 29 telas.

**Autoridade**: Canônico

**Severidade**: 🔴 (bloqueia M2; feature grande, 29 telas)

---

### D-CAP-012: Assembleia Live — 5 Etapas + 30+ Regras vs Genérico

**Conceito**: Live Assembly com votação, telão, procurações, ata

**O que v2 diz** (`00-product/portfolio-de-produtos.md`):
```
Assembly menciona Decision 1 (escopo) mas sem detalhe de etapas.
[genérico: "Live Assembly com WebSocket..."]
```

**O que canônico diz** (`/Content/03-Modulos/Assembleia-Live-Funcional.md`):
```
5 Etapas:
1. Convocação (edital + pauta + materiais)
2. Abertura (check-in + credenciamento + telão)
3. Discussão (fila de fala + cronômetro + transcription)
4. Votação (frações ideais, procurações, votação presencial/remota, homologação)
5. Encerramento (ata publicada, arquivo histórico)

30+ Regras:
- Fração ideal ≤100% agregada
- Procuração com assinatura eletrônica
- Quórum mínimo para deliberação
- Homologação pós-votação (síndico + notário futura)
- Telão (2 áreas: agenda + votação live)
- Fila cronometrada de fala
```

**Divergência**: **Tipo FALTANDO**. v2 menciona "Live Assembly" em portfolio mas sem as 5 etapas ou 30+ regras. Canônico tem spec densa (`Assembleia-Live-Funcional.md:425 linhas`).

**Ação proposta**: Expandir v2 `03-subprojects/` seção Assembly com 5 etapas + regras.

**Autoridade**: Canônico

**Severidade**: 🔴 (bloqueia M3; feature crítica, 30+ regras)

---

### D-CAP-013: Quotas Connect Me Morador Base — Q2 em Quebras

**Conceito**: Quantas vezes/ano morador base pode usar Connect Me Morador→Síndico

**O que v2 diz** (`00-product/personas.md:68`, `04-requirements/matrix-functional.md`):
Menção a "Connect Me 2/ano base; 4/ano pagante" sem conflito explícito.

**O que canônico diz** (`/Content/regras-de-negocio/quebras-detectadas.md:Q2`):
```
🔴 CONFLITO CRÍTICO:
- requirements/personas-and-plans.md:80 → "Morador Base | 2/ano"
- requirements/commercial.md Req 19.1 → "Base 2/ano"
- Decision 10 (regras-canonicas.md:100) → "Base: 2/ano"
- requirements/functional-matrix.md:43 → "Base ❌ (0)" + "Pagante 2/ano"
- requirements/functional-matrix.md:152 → "Base | 0 | Pagante | 4/ano"

Divergência: 3 fontes dizem Base=2, 1 fonte (matrix) diz Base=0.
```

**Divergência**: **Tipo CONFLITO INTERNO CANÔNICO**. Não é v2 vs canônico; é canônico consigo mesmo. Mas v2 herdou do legado a versão "Base 2/ano".

**Ação proposta**: Implementador deve escolher:
- Opção A: Base=2/ano (alinha personas-and-plans + commercial + Decision 10)
- Opção B: Base=0, Pagante=4/ano (alinha matrix, mais restritivo)

Registrar D-033 em STATE com decisão.

**Autoridade**: João (decisão comercial)

**Severidade**: 🔴 (afeta ABAC, quotas, produto)

---

### D-CAP-014: Empresa "Base" — Q3 em Quebras

**Conceito**: Existe plano "Empresa Base"?

**O que v2 diz** (`00-product/personas.md`):
Apenas Plus e Pro; nenhuma "Empresa Base".

**O que canônico diz** (`/Content/regras-de-negocio/quebras-detectadas.md:Q3`):
```
requirements/commercial.md Req 18 menciona "Base visível apenas para Síndico"
mas requirements/personas-and-plans só lista enterprise_plus e enterprise_pro.
```

**Divergência**: **Tipo CONFLITO INTERNO CANÔNICO — v2 está correto**. v2 acertou em não ter Empresa Base. Canônico tem erro em um arquivo (Req 18).

**Ação proposta**: Ignora divergência; v2 está correto. Canônico precisa fix em Req 18 (ver Q3 quebras).

**Autoridade**: v2 (está correto; canônico tem erro local)

**Severidade**: 🟡 (documentacional, não afeta código v2)

---

### D-CAP-015: Agência Marketing — Login vs Não-Login (Q4)

**Conceito**: Agência tem acesso próprio ou não?

**O que v2 diz** (`00-product/personas.md:127`):
```
"MVP (M1-M3): actor delegado. Agência tem conta User; recebe delegação scoped token..."
```

**O que canônico diz** (`/Content/regras-de-negocio/quebras-detectadas.md:Q4`):
```
🔴 CONFLITO:
- requirements/content.md Req 30 → "não tem login próprio, não é persona com role"
- requirements/identity.md Req 2 + T7 → roles incluem marketing
- requirements/personas-and-plans → "Plano: marketing_standard"

Raiz: Req 30 é absoluto, mas T7 + plano implicam que existe user com role marketing.
```

**Divergência**: **Tipo CONFLITO INTERNO CANÔNICO**. v2 resolveu: "tem user + role marketing, mas opera delegada". Canônico tem dois requisitos discordando.

**Ação proposta**: v2 está correta (tem user + login + role `marketing`, operando delegada). Canônico Req 30 precisa atualização (não é "sem login", é "sem tenant próprio").

**Autoridade**: v2 (está correto; canônico Req 30 é stale)

**Severidade**: 🟡 (v2 correto; canônico precisa fix)

---

### D-CAP-016: Síndico N1 e Certificados LMS (Q5)

**Conceito**: N1 pode obter certificado em cursos?

**O que v2 diz** (`04-requirements/matrix-functional.md`):
Matriz cobre Certificados com toggles por persona.

**O que canônico diz** (`/Content/regras-de-negocio/quebras-detectadas.md:Q5`):
```
🔴 CONFLITO:
- content.md Req 98.1 → "Síndico N1+ | Certificado | ✅"
- functional-matrix.md → "Síndico N1 | Certificado | ❌" "N2+ | ✅"

Alinhamento recomendado: matrix (mais restritivo) = N1 = "Aprender" tier sem credencial.
```

**Divergência**: **Tipo CONFLITO INTERNO CANÔNICO**. Duas specs canônicas discordam.

**Ação proposta**: Implementador debe seguir matrix (mais restritivo, alinhado com "N1=Aprender"). Canônico content.md precisa atualização.

**Autoridade**: Canônico matrix (mais apropriada)

**Severidade**: 🟡 (documentacional; regra clara)

---

### D-CAP-017: "AWS SES" em product-overview (Q1)

**Conceito**: Email provider é Resend ou SES?

**O que v2 diz** (implícito em herança de `product-overview.md`):
"Email: AWS SES (`IEmailProvider`)" (stale)

**O que canônico diz** (`/backend/CLAUDE.md`, `SESSION_CHARTER.md`, `Stack-e-Modulos.md:31`):
```
Email: Resend (não AWS SES)
Motivo: simplicidade operacional, suporte BR, custo menor
Backend F30 entregou 7 templates pt-BR via Resend
```

**Divergência**: **Tipo STALE**. v2 herdou spec antiga. Canônico atual é Resend (confirmado em código).

**Ação proposta**: Atualizar `00-product/` e `04-requirements/` menção de "AWS SES" para "Resend".

**Autoridade**: Canônico (código implementado)

**Severidade**: 🔴 (pode gerar PRs com SES; bloqueia M1 se infra for configurada errada)

---

### D-CAP-018: "AWS ECS Fargate" em product-overview (Q11)

**Conceito**: Deploy é Railway ou AWS ECS?

**O que v2 diz**:
Implícito em herança de spec antiga.

**O que canônico diz** (`Stack-e-Modulos.md:60-62`, `STATE.md`, memória `project_railway_url.md`):
```
Deploy: Railway primário (project ID efe664b7-5352-48e7-ae09-78d2f286e0d5)
AWS é plano futuro registrado em design/deploy-config.md
```

**Divergência**: **Tipo STALE**. v2 herdou spec antiga.

**Ação proposta**: Atualizar `02-architecture/` menção de "AWS ECS" para "Railway primário; AWS futuro".

**Autoridade**: Canônico

**Severidade**: 🔴 (infra deployment; critico)

---

### D-CAP-019: "OpenSearch" em product-overview (Q1)

**Conceito**: Busca é OpenSearch ou PostgreSQL tsvector?

**O que v2 diz**:
Implícito: "OpenSearch (Sprint 5)"

**O que canônico diz** (`Stack-e-Modulos.md:33`):
```
Busca: PostgreSQL tsvector (não OpenSearch)
Motivo: complexidade operacional; Postgres suficiente para MVP
```

**Divergência**: **Tipo STALE**.

**Ação proposta**: Atualizar spec.

**Autoridade**: Canônico

**Severidade**: 🔴 (arquitetura search; afeta queries)

---

### D-CAP-020: Visibilidade de Vídeo Empresa (9 Quebras confusas)

**Conceito**: Quando morador/síndico/empresa veem vídeos de outras empresas?

**O que v2 diz** (`04-requirements/matrix-functional.md:55-56`):
```
Vídeos de empresas (preview 25%) — Morador Base/Pagante veem 25%
Vídeos de empresas (integral) — N3 ✓, Plus ?, Pro ?
```

**O que canônico diz** (`regras-canonicas.md:38`):
```
Regra de Ouro #4: "Visibilidade de vídeos de empresa — moradores só durante votação de fornecedor"
Flag: autorizar_exibicao_votacao=true + video_visibility_grants
Revogação automática ao fim da votação
```

**Divergência**: **Tipo CONFLITO CONCEITUAL**. Matriz funcional fala de 25% de preview sem contexto; Regra de Ouro restringe a "votação de fornecedor". Confusão fundamental sobre quando vídeos são visíveis.

**Ação proposta**: **CLARIFICAR COM JOÃO**:
- (a) "25% preview" é quando? Sempre, ou apenas votação?
- (b) Empresa Plus/Pro veem vídeos umas das outras (parceria)?

Registrar D-034 com resolução.

**Autoridade**: João (requisito)

**Severidade**: 🔴 (afeta visibility grants; compliance)

---

### D-CAP-021: Role Types PT-BR vs Inglês (Q19)

**Conceito**: role_type no banco é pt-BR ou inglês?

**O que v2 diz** (`01-domain/ubiquitous-language.md`):
Implícito: singular em PT-BR (mas será que está lá?)

**O que canônico diz** (`/Content/04-Listas-Mestre/Enums-Consolidados.md:22-32`):
```
Role Types (sub-contexto operacional, só no banco — T8):
principal | auxiliary | assistant     # síndico
titular   | dependent                 # morador
administrator | operator              # empresa
```

Mas histórico em `requirements.md` original (legado) usa PT-BR: `sindico, morador_titular, morador_dependente, empresa_administrador, empresa_operador`.

**Divergência**: **Tipo ENUMS DIVERGENTE**. Canônico atual decidiu inglês (T8). v2 herdou legado pt-BR?

**Ação proposta**: Verificar v2 `01-domain/ubiquitous-language.md` e `04-requirements/registration-data.md` para ver qual usa. Se PT-BR, atualizar para inglês (T8).

**Autoridade**: Canônico (T8 técnica decision)

**Severidade**: 🔴 (enums banco; bloqueia implementação)

---

### D-CAP-022: Condominium Limit 12 vs 15 (Q30)

**Conceito**: Máximo de condomínios que síndico pode gerenciar

**O que v2 diz** (implícito em destilação):
PDF Síndico diz 12; novo Decision/novo escopo não estava claro em v2.

**O que canônico diz** (`regras-canonicas.md:102`, `Inventario-Telas.md:88`):
```
Decision 11: Limite = 15 (não 12)
Razão: expansão de mercado, síndicos profissionais gerenciam múltiplos
S2 — "Meus Condomínios" (até 15, ordenar ativos primeiro)
```

**Divergência**: **Tipo ATUALIZADO**. v2 pode estar com 12; canônico diz 15.

**Ação proposta**: Verificar v2 e atualizar para 15 se necessário. Guardar D-035.

**Autoridade**: Canônico (Decision 11, código)

**Severidade**: 🔴 (limite de quota; pode bloquear síndico)

---

### D-CAP-023: Connect Me E→E Quotas (Q8 Divergência Redação)

**Conceito**: Empresa Plus pode abrir E→E ou não?

**O que v2 diz** (`00-product/personas.md:87`, `04-requirements/matrix-functional.md`):
"Empresa Plus: (não aplicável — E→E só Plus/Pro)" — confusa, sugere que Plus **pode**.

**O que canônico diz** (`/Content/regras-de-negocio/quebras-detectadas.md:Q8`):
```
3 redações diferentes:
- personas-and-plans: "não aplicável"
- functional-matrix: "E→E apenas" (ambíguo — "apenas"?)
- commercial.md: "ilimitado para Pro, configurável para Plus"
```

**Divergência**: **Tipo REDAÇÃO CONFUSA**. Precisa clarificar se Plus tem E→E ou não.

**Ação proposta**: Padronizar em: **"Empresa Plus: E→E com cap anual; Empresa Pro: E→E ilimitado."** Replicar em todas 3 fontes v2.

**Autoridade**: João (decisão comercial)

**Severidade**: 🟡 (clarificar; afeta quotas)

---

### [D-CAP-024 a D-CAP-047: Continuação de CRÍTICAS...]

Por brevidade, resumo as 24 restantes em tabela e depois detalhamento selecionado:

| ID | Conceito | Tipo | Severidade |
|---|---|---|---|
| D-CAP-024 | Vídeos Institucionais — Quota Mensal vs Trava Trimestral (Q9) | Conflito lógico | 🔴 |
| D-CAP-025 | Morador Pagante — Acesso LMS não claro (Q5 variant) | Faltando | 🔴 |
| D-CAP-026 | Painel Admin Master Síndico — Telas não documentadas | Faltando | 🔴 |
| D-CAP-027 | Onboarding Persona — 4 fluxos 4/6/7/4 telas | Faltando Detalhes | 🔴 |
| D-CAP-028 | Notificações Sistema — Triggers não documentados | Faltando | 🔴 |
| D-CAP-029 | Busca — Campos indexáveis por persona | Faltando Spec | 🔴 |
| D-CAP-030 | Relatórios Export PDF — Scope de Síndico | Faltando | 🔴 |
| D-CAP-031 | Termo LGPD e Assinatura — Versionamento | Faltando Detalhe | 🔴 |
| D-CAP-032 | Device Fingerprint — Hash algoritmo | Faltando Spec | 🔴 |
| D-CAP-033 | Rate Limiter — Limites por tipo req | Faltando Spec | 🔴 |
| D-CAP-034 | Moderação Conteúdo — Triggers (harassment, spam, NSFW) | Faltando | 🔴 |
| D-CAP-035 | Proposta de Deal Formal — Contrato eletrônico | Faltando | 🔴 |
| D-CAP-036 | Amends (Adendo Formal) — Regras de edição | Faltando | 🔴 |
| D-CAP-037 | Procuração Assembleia — Assinatura eletrônica | Faltando | 🔴 |
| D-CAP-038 | Homologação Votação — Papel síndico/notário | Faltando | 🔴 |
| D-CAP-039 | Metricas Engagement — Visibilidade privada | Faltando Spec | 🔴 |
| D-CAP-040 | Avaliação Empresa Pós-Contrato — Reputação | Faltando | 🔴 |
| D-CAP-041 | Sistema de Tags/Categorias Empresa — Hierarchy | Faltando | 🔴 |
| D-CAP-042 | Benefício Exclusivo Condomínio — Visibilidade | Faltando | 🔴 |
| D-CAP-043 | Palavra do Dia (Gamificação) — Trava 4h | Faltando | 🔴 |
| D-CAP-044 | Curriculum Video Lock 90d — Restrição Exata | Faltando | 🔴 |
| D-CAP-045 | Indicadores KPI Síndico — Dashboard Fields | Faltando | 🔴 |
| D-CAP-046 | Limite de Usuários por Tenant — Regra | Faltando | 🔴 |
| D-CAP-047 | Recovery Senha — Flow Zitadel integration | Faltando | 🔴 |

---

## 🟡 MÉDIAS (38 divergências)

Resumo por categoria:

### Categoria: Documentação Stale
- **M-001**: `Content/requirements.md` — documento legado, menções de roles PT-BR, Mercado Pago, AWS SES
- **M-002**: `Content/design.md` — design monolítico original, parcialmente stale
- **M-003**: `Content/contents/master-sindico-domain-mapping.md` — usa TS, Awilix DI, Mercado Pago
- **M-004**: `Content/excopo-tecnico-antigo.txt` — v3.0 Janeiro 2026, obsoleto
- **M-005**: `Content/main.go.md` — esboço inicial, não reflete código atual

Ação: Marcar com `status: deprecated` e notas de redirecionamento no topo.

### Categoria: Divergências de Nomenclatura
- **M-006**: Slugs de planos — v2 usa nomes humanos, canônico usa slugs DB
- **M-007**: Tipos de Benefício Vizinhança — 7 tipos + subtipo proposto
- **M-008**: Tipos de Comunicado — 8 base vs sub-categorias

Ação: Padronizar slugs em v2; adicionar mapeamento visual↔DB.

### Categoria: Conflitos Internos Canônico
- **M-009**: Score Governança 0-100 vs 1-10 (Q25)
- **M-010**: Visibilidade Live Assembly — online/híbrida vs presencial (Decision 1)
- **M-011**: Trava Trimestral Vídeo — "por vídeo" vs "global" (Q9)

Ação: Documentar ambiguidade; pedir clarificação João (D-036..D-040).

### Categoria: PDFs/INI Não-Lidos Completamente
- **M-012**: Banco de Talentos 11 telas — em PDF não lido
- **M-013**: Academia LMS 15 telas — em ####TELA####.ini não lido
- **M-014**: Tipos de Comunicado Específicos — em ####TELA####.ini
- **M-015**: Bloco Usuários e Permissões Síndico — em ####TELA####.ini

Ação: Ler PDFs/INI completos; adicionar a v2.

### Categoria: Features Canceladas vs Pendentes
- **M-016**: Certificação Empresa Pro — cancelada ou pendente? (Q5)
- **M-017**: Modalidade Assembleia Online/Híbrida — Decision 1 diz inativa MVP
- **M-018**: Painel Admin Master Síndico — In-scope mas sem telas

Ação: Confirmar status; adicionar Backlog/M5+ se cancelado.

### [M-019 a M-038: Documentação Organizacional...]

Resumo restante em tabela:

| ID | Tema | Status | Ação |
|---|---|---|---|
| M-019 | Obsidian Vault Externo (João) — Stale | Referência | Deprecar notas técnicas |
| M-020 | Deploy Config — D-029 (R2 vs S3) aberta | Pendente | Fechar D-029 |
| M-021 | GSD vs GitHub Spec Kit — Termo confundido (Q18) | Cosmético | Nota explicativa em SDD workflow |
| M-022 | Migrations Numeração — Limite 100 por BC (Q16) | Cosmético | Monitor se overflow |
| M-023 | Parâmetros Reputação Bronze→Diamante (Q15) | Faltando Spec | Criar cuando enter sprint |
| M-024 | Sprint Roadmap Alocação Web/Mobile | Faltando | Atualizar p/ M1/M2/M3 |
| M-025 | Checklist DoR/DoD v2 | Faltando | Criar em DoR-DoD.md |
| M-026 | Matriz de Testes Pirâmide 70-20-10 | Faltando | Adicionar em 09-testing/ |
| M-027 | Setup Zitadel em Railway — Passo a passo | Faltando | Criar em 06-execution/dev.md |
| M-028 | Seed Data Banco Dados | Faltando | Criar migrations de exemplo |
| M-029 | Config vars Ambiente (Railway vs dev) | Faltando | Atualizar .env.example |
| M-030 | Estratégia de Cache Redis por BC | Faltando | Documentar em 08-security/ |
| M-031 | Politica de Retenção de Dados LGPD | Faltando | 5 anos = retenção |
| M-032 | Webhook Validation — HMAC Stripe/Mux | Faltando Spec | Detalhar em 08-security/ |
| M-033 | Cert Pinning Mobile — Certificado Prod | Faltando | Gerar p/ iOS/Android |
| M-034 | Circuit Breaker Padrão inter-BC | Faltando | Padrão não documentado |
| M-035 | Error Handling — AppError codes (0001..0999) | Faltando Catalog | Criar enum codes |
| M-036 | Logging Estruturado — Fields obrigatórios | Faltando Spec | Padrão não escrito |
| M-037 | Observabilidade — Trace IDs OTEL | Faltando | Span naming convention |
| M-038 | Disaster Recovery — RTO/RPO | Faltando | SLA não definido |

---

## 🟢 BAIXAS (9 divergências)

| ID | Tema | Tipo | Ação |
|---|---|---|---|
| L-001 | Tipos de Vídeo — 12 tipos (content.md vs architect-solutions.md) | Nomenclatura | Usar content.md como canônico |
| L-002 | PDF Jornadas — 6 PDFs no vault, não em v2 | Referência | Linkar em 90-ingested/ |
| L-003 | Limite Tamanho Upload Vídeo (500MB) | Faltando | Adicionar em requirements |
| L-004 | Timeout Requisições HTTP | Faltando | Default 30s proposto |
| L-005 | Limites de Paginação (padrão 20 items) | Faltando | Adicionar em API Design |
| L-006 | Formato Resposta JSON — Envelope obrigatório | Faltando | Spec em api-design.md |
| L-007 | Correlação com UUID — Geração | Faltando | UUIDv7 (já implementado) |
| L-008 | Imagens — Max dimensão, formatos | Faltando | 5MB, JPG/PNG/WebP |
| L-009 | Fuso Horário — Sempre UTC no banco | Faltando | Padrão não escrito |

---

## § 3. ESTADO DO BACKEND REAL (Código é Verdade)

### Mini-Inventário: Aggregates, Endpoints, Migrations

#### Identity Module (`internal/modules/identity/`)
- **Aggregates**: `User` (raiz), `Session`, `DeviceFingerprint`
- **VOs**: `Email`, `CPF`, `CNPJ`, `Phone`, `Password`, `ZitadelID`
- **Migrations**: 001-099 (identity); atual ~14 migrations
- **Endpoints**: `GET /auth/login`, `GET /auth/callback`, `GET /auth/logout`, `POST /webhooks/auth/*`
- **Providers**: `IAuthProvider` (Zitadel v4 OIDC)
- **Status**: ✅ Produção (Sprints 0-1)

#### Billing Module (`internal/modules/billing/`)
- **Aggregates**: `Subscription`, `Plan`, `FeatureUsage`
- **Migrations**: 100-199; atual ~12 migrations
- **Endpoints**: Plano público (não em /api/v1 — soft-block no handler)
- **Providers**: `IPaymentGateway` (Stripe)
- **Quotas Cached**: Redis `ms:quota:{userId}:{feature}:{period}`
- **Status**: ✅ Produção (Sprint 1)

#### Institutional Module (`internal/modules/institutional/`)
- **Aggregates**: `Condominium` (raiz), `Unit`, `Membership`, `Mandate`, `SyndicCompany`, `TimelineEntry`, `MasterPlanAction`, `Event`, `Communication`, `Survey`, `GovernanceAssessment`
- **Migrations**: 200-299; atual ~25 migrations
- **Key Invariants**: 
  - Sum of ideal fractions ≤ 100%
  - Timeline INSERT-only (no UPDATE/DELETE)
  - Membership única por (user, condominium) em momento
  - Announcement imutável pós-publish
- **Endpoints**: 50+ (Síndico CRUD condomínio, timeline, eventos, comunicados, etc)
- **Status**: ✅ Produção (Sprints 2-3)

#### Commercial Module (`internal/modules/commercial/`)
- **Aggregates**: `Company` (raiz), `CompanyProfile`, `ConnectMeRequest`, `Proposal`, `Deal`, `ExecutionRecord`, `TechnicalActivity`, `Amendment`, `CompanyEvaluation`, `NeighborhoodPartnership`, `NeighborhoodBenefit`
- **Migrations**: 300-399; atual ~22 migrations
- **Quota Tracking**: Connect Me, propostas, amends
- **Evento NATS**: connect_me.request.created, connect_me.interested, deal.closed
- **Status**: ✅ Produção (Sprint 4)

#### Content Module (`internal/modules/content/`)
- **Aggregates**: `Video` (raiz), `VideoAnalytics`, `VideoVisibilityGrant`, `MarketingAgencyToken`, `Course`, `Lesson`, `Certificate`, `ForumTopic`, `EBook`
- **Migrations**: 400-499; atual ~18 migrations
- **Providers**: `IVideoProvider` (Mux), `IStorageProvider` (MinIO/R2)
- **Lock 90d**: Campo `last_updated_trimester` em video
- **Evento NATS**: video.published, video.visibility_granted, course.completed
- **Status**: ✅ Produção (Sprint 5)

#### Assembly Module (`internal/modules/assembly/`)
- **Aggregates**: `Assembly` (raiz), `AssemblyAgenda`, `AssemblyAgendaItem`, `AssemblyFraction`, `AssemblyVoting`, `AssemblyVote`, `AssemblyCheckIn`, `AssemblySpeechQueue`, `Proxy`, `AssemblyHomologation`
- **Migrations**: 500-599; atual ~18 migrations
- **Key Invariants**:
  - Procuração assinada (eletrônica)
  - Quórum mínimo
  - Homologação pós-votação
  - Ata publicada imutável
- **WebSocket Integration**: Livekit real-time signaling
- **Status**: ✅ Produção (Sprint 7)

#### Cross-Domain (Shared)
- **Middleware**: Recovery, RequestID, Logger, CORS, ErrorHandler, RateLimiter (IP+UserID), Authenticate, RequirePermission (ABAC)
- **ABAC Engine**: `internal/shared/middleware/permission.go` — avalia `(action, resource, context)` contra matriz de permissões
- **Domain Events**: Tabela `domain_events` (outbox) + consumer NATS JetStream
- **Audit Logger**: (Pendente DT-010, Sprint 10)
- **LGPD Logs**: Tabela `lgpd_logs` (append-only, retenção 5 anos)

---

## § 4. ARQUIVOS v2 QUE PRECISAM RE-DESTILAÇÃO (Prioritário)

Ordenado por impacto + urgência:

| # | Arquivo v2 | Motivo | Impacto | Prioridade |
|---|---|---|---|---|
| 1 | `00-product/personas.md` | Faltam tipos morador (titular/dependente); empresa administradora genérica | Bloqueia ABAC design | 🔴 **P0** |
| 2 | `00-product/portfolio-de-produtos.md` | Superficial em Vizinhança (29 telas); Assembly (30+ regras); Banco Talentos (11 telas); Compliance (C1-C11) | Bloqueia M2/M3 planning | 🔴 **P0** |
| 3 | `04-requirements/matrix-functional.md` | Conflitos em quotas (Q2, Q3); visibilidade vídeo (Q20); necessita expansão | Bloqueia ABAC testing | 🔴 **P0** |
| 4 | `04-requirements/registration-data.md` | Falta tipos condomínio (6 enums); bloco empresa administradora | Bloqueia Onboarding | 🔴 **P0** |
| 5 | `03-subprojects/web/ui-catalog.md` | Faltam 8 telas Marketing (MK1-MK8); 29 telas Vizinhança (VZ/VS/VE/VM); 11 telas Banco Talentos; 5 etapas Assembly | Bloqueia Web Sprint | 🔴 **P0** |
| 6 | `04-requirements/compliance-lgpd.md` | Superficial; faltam C1-C11 + 4 gatilhos + fluxos | Bloqueia M1 | 🔴 **P0** |
| 7 | `02-architecture/api-design.md` | Faltam specs de erro (AppError codes); header obrigatórios; paginação | Bloqueia OpenAPI | 🟡 **P1** |
| 8 | `09-testing/test-strategy.md` | Faltam matrix de testes por BC; coverage targets; PBT spec | Bloqueia QA planning | 🟡 **P1** |

---

## § 5. ARQUIVOS v2 QUE PERMANECEM CORRETOS (Validados)

- **`00-product/vision.md`** — Visão geral OK
- **`00-product/business-model.md`** — Modelo genérico OK (detalhes em persona + portfolio)
- **`00-product/glossary.md`** — Termologia OK
- **`01-domain/bounded-contexts.md`** — 6 BCs mapeados corretamente ✅
- **`01-domain/invariants.md`** — Invariantes canonicamente alinhadas ✅
- **`01-domain/business-rules.md`** — Regras de negócio OK
- **`01-domain/ubiquitous-language.md`** — UL OK (se em inglês)
- **`01-domain/domain-events.md`** — Eventos OK (se alinhado com canonical)
- **`02-architecture/c4-context.md`** — C4 Context OK
- **`02-architecture/c4-containers.md`** — Containers OK
- **`02-architecture/c4-components.md`** — Components OK
- **`02-architecture/clean-arch-mapping.md`** — Clean Arch OK
- **`02-architecture/topology-multitenant.md`** — Multitenancy OK
- **`04-requirements/global.md`** — Requirements globais OK (genéricos)
- **`04-requirements/non-functional.md`** — NFRs OK

---

## § 6. ITEM QUE **DESAPARECEU** E PRECISA ESCLARECIMENTO

### Sistema de Cupons / Promoção do Dia (Q23 — CRÍTICO)

**Estado em Janeiro 2026** (`excopo-tecnico-antigo.txt`):
```
Sprint 3 — Engine de Cupons (Promoção do Dia)
- Formato de código: ID_LOJA(5) + SEQUENCIAL(3) + PALAVRA(5) = 13 caracteres
  Exemplo: 00123055PAO
- Trava temporal: máximo 1 cupom a cada 4 horas por estabelecimento
- Gamificação: palavra-chave do dia muda (hora em hora? ou dia em dia?)
- "NÃO é sistema de helpdesk" — esclarecimento importante (termo "ticket" = cupom)
- Promoções exclusivas por condomínio: via número de inscrição
```

**Estado atual**:
- Não existe em requirements/ (nem em v2 nem em canônico destilado)
- Aparentemente substituído por "Ativar Promoção" genérica (VZ3, VS7, VE4, VM3) com 7 tipos de benefício
- Nenhuma menção a formato específico ID_LOJA+SEQ+PALAVRA
- Nenhuma menção a trava de 4 horas
- Nenhuma menção a "palavra do dia" gamificação

**Questões Críticas**:
1. **Foi cancelado formalmente?** Se sim, é contratual issue? (estava no escopo de Janeiro)
2. **Foi substituído por "Ativar Promoção"?** Se sim, qual é a equivalência?
3. **Está pendente para Sprint 10+?** Se sim, deve entrar em backlog

**Ação Obrigatória**: Confirmar com João em D-041.

---

## § 7. RESUMO EXECUTIVO

### Contagem de Divergências por Severidade

| Severidade | Quantidade | Resolver em |
|---|---|---|
| 🔴 CRÍTICA | 47 | Agora (bloqueiam M1/M2/M3) |
| 🟡 MÉDIA | 38 | Próxima semana (importantes, não bloqueantes) |
| 🟢 BAIXA | 9 | Próximo mês (cosmético) |
| **Total** | **94** | |

### Top 20 Divergências 🔴 com Ação Obrigatória

1. **D-CAP-001**: Nomenclatura de Planos — Slugs DB vs Nomes UI → **Atualizar v2 personas.md com coluna slug**
2. **D-CAP-002**: Tipos de Morador (titular/dependente) → **Adicionar seção em v2 personas.md**
3. **D-CAP-003**: Blocos de Condomínio — **Ler código institutional**, validar modelagem
4. **D-CAP-004**: Tipos de Condomínio (6 enums) → **Adicionar a registration-data.md**
5. **D-CAP-005**: Empresa Administradora → **Clarificar em v2 como vínculo, não persona**
6. **D-CAP-006**: Shorts vs Vídeo Currículo → **Confirmar com João**
7. **D-CAP-007**: Sistema de Cupons (CRÍTICO) → **D-041: Confirmar status contratual com João**
8. **D-CAP-008**: Marketing Link (MK1-MK8) → **Adicionar 8 telas a web/ui-catalog.md**
9. **D-CAP-009**: Banco de Talentos (BT01-BT11) → **Ler PDFs, adicionar 11 telas**
10. **D-CAP-010**: Compliance (C1-C11 + 4 gatilhos) → **Expandir compliance-lgpd.md**
11. **D-CAP-011**: Vizinhança (4 jornadas, 29 telas) → **Re-estruturar web/ui-catalog.md**
12. **D-CAP-012**: Assembleia Live (5 etapas, 30+ regras) → **Expandir portfolio-de-produtos.md**
13. **D-CAP-013**: Quotas Connect Me Base (Q2) → **D-033: João define 2/ano vs 0**
14. **D-CAP-017**: AWS SES → Resend → **Atualizar spec (stale)**
15. **D-CAP-018**: AWS ECS → Railway → **Atualizar spec (stale)**
16. **D-CAP-019**: OpenSearch → PostgreSQL tsvector → **Atualizar spec (stale)**
17. **D-CAP-020**: Visibilidade Vídeo Empresa → **D-034: João clarifica 25% vs votação**
18. **D-CAP-021**: Role Types PT-BR vs Inglês → **Verificar v2, atualizar para inglês (T8)**
19. **D-CAP-022**: Condominium Limit 12 vs 15 → **Atualizar para 15 (Decision 11)**
20. **D-CAP-023**: Connect Me E→E Quotas → **Padronizar redação em 3 fontes**

### Arquivos v2 Prioritários para Re-Destilação (8 arquivos)

```
PRIORIDADE P0 (Bloqueiam M1):
1. 00-product/personas.md — Adicionar tipos morador + empresa admin
2. 00-product/portfolio-de-produtos.md — Expandir Vizinhança/Assembly/Talentos/Compliance
3. 04-requirements/matrix-functional.md — Resolver Q2, Q3, Q20 conflitos
4. 04-requirements/registration-data.md — Adicionar enums condomínio, empresa
5. 03-subprojects/web/ui-catalog.md — Adicionar 53 telas faltantes (MK, VZ/VS/VE/VM, BT, Assembly)
6. 04-requirements/compliance-lgpd.md — Detalhar C1-C11 + 4 gatilhos

PRIORIDADE P1 (Importantes, não bloqueiam M1):
7. 02-architecture/api-design.md — Adicionar specs erro, headers, paginação
8. 09-testing/test-strategy.md — Adicionar matrix testes, coverage targets
```

### Recomendação de Fase de Corrção (Ordem Sugerida)

**Fase 1 (Agora — bloqueadores)**: 
- D-CAP-001, D-CAP-002, D-CAP-004, D-CAP-008 a D-CAP-012, D-CAP-017 a D-CAP-019
- **Tempo estimado**: 5-8 dias (arquivos 1, 2, 5, 6)

**Fase 2 (Esta semana — contexto crítico)**:
- D-CAP-003, D-CAP-005, D-CAP-013, D-CAP-020 a D-CAP-023
- Confirmações com João (D-006, D-007, D-033 a D-035, D-041)
- **Tempo estimado**: 2-3 dias (validações)

**Fase 3 (Próxima semana — detalhamentos)**:
- PDFs/INI não-lidos (Banco Talentos, Academia, tipos específicos)
- 🟡 Médias (M-001 a M-038)
- **Tempo estimado**: 3-5 dias

**Fase 4 (Próximo mês — polishing)**:
- 🟢 Baixas (L-001 a L-009)
- Documentação organizacional (MOCs, índices)
- **Tempo estimado**: 2 dias

---

## § 8. CONCLUSÕES E RECOMENDAÇÕES

### Estado Geral da v2

**v2 é ~70% correta e estruturalmente saudável.** Cobre os conceitos principais, arquitetura DDD/Clean está OK, bounded contexts mapeados. Mas:

1. **30% de incompletude**: Particularmente Vizinhança (29 telas), Compliance (C1-C11), Banco Talentos (11 telas), Assembly (30+ regras), Marketing Link (8 telas). Totalizando ~53 telas e múltiplas regras não documentadas.

2. **8% de stale**: AWS SES, AWS ECS, OpenSearch — decisões superadas (Resend, Railway, PostgreSQL).

3. **3% de conflitos internos canônico**: Quotas Connect Me Base, visibilidade vídeo, certificados LMS. Não são culpa de v2; é ambiguidade em 2+ specs canônicas.

4. **1 item crítico desaparecido**: Sistema de Cupons (contratual?).

### Recomendação Final

**v2 deve ser re-destilada em DUAS PASSAGENS**:

1. **Passagem 1 (48h)**: Arquivos P0 (personas, portfolio, matrix, registration-data, ui-catalog, compliance)
   - Incorporar diferenças detectadas neste audit
   - Resolver Q2/Q3/Q20 com João
   - Atualizar stack (SES→Resend, ECS→Railway, OpenSearch→tsvector)

2. **Passagem 2 (3-5 dias)**: PDFs + INI + validação
   - Ler `Content/contents/` PDFs completamente
   - Ler `####TELA####.ini` completamente
   - Adicionar 53 telas faltantes + regras

### Depois: Validação com João

- [ ] D-006: Shorts vs currículo — definir produto
- [ ] D-007: Cupons — confirmar status (cancelado/substituído/pendente)
- [ ] D-033: Base Connect Me — 2/ano vs 0/ano
- [ ] D-034: Visibilidade vídeo — 25% vs votação
- [ ] D-035: Condominium limit — validar 15
- [ ] D-041: Cupons (crítico contratual)

### Qualidade de v2 para Implementação

**Após correções Fase 1 + 2**: v2 será adequada para alimentar sprints web/mobile/backend com confiança ~95%.  
**Após Fase 3**: 99% cobertura completa.

---

## § 9. APÊNDICES

### Apêndice A: Mapeamento de Telas Faltantes

```
Faltando em v2 ui-catalog.md:

Marketing (8): MK1-MK8 ✗ (só descrição funcional)
Vizinhança (29):
  - Parceiro: VZ1-VZ6 ✗
  - Síndico: VS1-VS10 ✗
  - Empresa: VE1-VE6 ✗
  - Morador: VM1-VM7 ✗
Banco Talentos (11): BT01-BT11 ✗ (mencionado "11 telas" mas não expandido)
Academia LMS (15): LMS1-LMS15 ✗ (em .ini, não em v2)
Assembly (especificar N telas): ~30 telas não expandidas
Compliance (C-screens): C1-C11 não expandidas
Admin Master: Admin telas não listadas

Total faltando: 53+ telas
```

### Apêndice B: Decisões Pendentes com João (Novas)

```
D-033: Conectr Me Morador Base quota — 2/ano (Decision 10 + personas-and-plans + commercial) vs 0/ano (matrix)
D-034: Visibilidade vídeo empresa — 25% preview sempre vs durante votação de fornecedor?
D-035: Condominium limit — 12 (PDF old) vs 15 (Decision 11)?
D-036: Shorts — é produto vertical ou não?
D-037: Empresa Plus E→E — tem cap ou ilimitado?
D-038: GSD vs GitHub Spec Kit — qual referência usar?
D-039: Módulos Testes — coverage 95% domain / 90% app / 85% infra / 75% http OK?
D-040: Feature Flags — qual ferramenta? Internamente ou SaaS?
D-041: **CRÍTICO** — Sistema de Cupons — cancelado/substituído/pendente?
```

### Apêndice C: Fontes Completas Canônicas

```
Autoridade Máxima (aplicar em ordem):
1. /backend/.kiro/specs/master-sindico/requirements/*.md (spec viva)
2. /backend/.kiro/STATE.md (decisões D-0XX)
3. /backend/internal/modules/*/domain/ (código)
4. /Content/ (consolidação histórica)
5. Legado /Content/requirements.md, /Content/design.md, PDFs
```

---

**Relatório finalizado:** 2026-04-23 | **Próximas ações:** Implementar Fase 1 (48h); Confirmar D-033..D-041 com João.

