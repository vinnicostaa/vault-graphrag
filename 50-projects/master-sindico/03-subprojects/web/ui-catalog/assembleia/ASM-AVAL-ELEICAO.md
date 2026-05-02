---
title: ASM-AVAL-ELEICAO — Avaliação Escondida de Gestão em Eleição
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia, eleicao, lgpd]
project: master-sindico
persona: morador
category: ASM
screen_id: ASM-AVAL-ELEICAO
sub_produto: assembleia-live
plan_requirement: any
status: specification
stack: web
milestone: m3
created: 2026-04-24
---

# ASM-AVAL-ELEICAO — Avaliação Escondida de Gestão em Eleição

## Finalidade

Tela **obrigatória e confidencial** exibida ao morador quando a assembleia ao vivo chega a um item de pauta tipo `eleicao_sindico`. Bloqueia o voto no candidato até o morador submeter avaliação de percepção da gestão atual. Alimenta `governance_score` histórico do síndico substituído, sem vazar respostas individuais.

## Fonte canônica

- [[../../../../STATE#D-104 — Avaliação escondida de gestão em eleição ATIVADA (fecha Q40)|D-104]]
- [[../../../../01-domain/invariants#INV-ASM-023]]
- [[../../../../04-requirements/functional/assembly#ASM-ELE-AVAL]]
- `ARQUITETURA FUNCIONAL DO MÓDULO ASSEMBLEIA.pdf` §4.4 (arquivado em `90-archive/master-sindico-pdfs-cliente-originais/`)
- [[../../../../STATE#D-103 — Score duplo no perfil do síndico (fecha Q25)|D-103]] (alimenta INV-GOV-001)

## Persona e ABAC

- **Persona primária**: Morador (role `resident`, `role_type ∈ {titular, dependente}`) — apenas moradores que fizeram **check-in** na assembleia ativa
- **Plan tier mínimo**: qualquer (base, paid) — morador base também é forçado a avaliar
- **ABAC action**: `assembly.hidden_assessment.submit`
- **Restrições**:
  - `RequireTenantMatch: true` — mandate_id do morador deve bater com o condomínio da assembleia
  - Tenta submeter sem pauta ativa `eleicao_sindico` → 409
  - Tenta submeter 2ª vez na mesma eleição → 409 `já avaliou`
  - `admin` role **não pode** acessar avaliação individual (regra absoluta — ver §Regras)

## Fluxo da tela

1. Morador está na assembleia ao vivo e o presidente abre pauta `eleicao_sindico`
2. Sistema detecta que morador ainda não submeteu avaliação desta eleição → **redireciona automaticamente** para ASM-AVAL-ELEICAO (bloqueia tela de voto)
3. UI exibe:
   - Header com mandato do síndico avaliado (ex: "Avaliação da gestão: João Silva, mandato 2024-2026")
   - **Banner destacado**: "⚠️ Esta avaliação é **confidencial**. O síndico atual não verá nenhum dado individual — apenas a média agregada ao final do mandato."
   - Lista de **5 perguntas** escala 1-10 (RadioGroup horizontal 1..10) — textos canônicos em [[../../../../04-requirements/functional/assembly#asm-ele-aval|ASM-ELE-AVAL]] via [[../../../../STATE#d-110|D-110]]
   - Botão "Submeter" desabilitado até todas respondidas
4. Morador responde todas; clica Submit
5. Sistema valida ABAC + chama `POST /api/v1/assembly/:id/hidden-assessments`
6. Backend:
   - `morador_hash = HMAC(mandate_id, user_id)` (via secret rotating — [[../../../../02-architecture/adr/0028-lgpd-keyed-hmac]])
   - INSERT em `hidden_assessments (mandate_id, morador_hash, answers, submitted_at)`
   - Publica evento `HiddenAssessmentSubmitted(mandate_id, morador_hash, submitted_at)` — **SEM** payload de respostas
   - Retorna 201
7. UI exibe modal: "✓ Obrigado, sua avaliação foi registrada." + botão "Prosseguir para votação"
8. **Não exibe** score próprio (regra dura)
9. Redirect automático para ASM-VOTO (pauta candidato)

## Componentes

- `HiddenAssessmentForm` (novo — ver [[../../design-system]] seção "Fase 12")
  - Variants: `questions=3|5`
  - Props: `questions: Question[]`, `mandateId`, `onSubmit(answers)`
  - Lógica interna: NavGuard bloqueia back button / refresh
- `NavGuard` (novo) — hook/componente bloqueia route change + unload
- `ConfidentialBanner` (novo) — destaca aviso LGPD
- `RadioGroup` (Kobalte) — range 1-10 horizontal
- `SubmitButton` (existente) — disabled até form válido
- `ConfirmationModal` (existente) — pós-submit sem mostrar score

## Estados

- **Loading**: spinner "Carregando avaliação..." enquanto backend verifica se já submeteu
- **Empty (not-needed)**: morador já submeteu ou pauta não é eleição → não exibe, redirect para ASM-VOTO (tela nunca aparece para esse caso)
- **Error**:
  - 409 "já avaliou": banner "Você já submeteu avaliação desta eleição" + redirect para ASM-VOTO
  - 409 "assembleia não ativa": banner + redirect para tela de lista de assembleias
  - 403 "não faz parte do condomínio": banner de erro e logout-hint
  - 500: retry 3x; se falhar, "Tente novamente ou procure suporte"
- **Success**: modal de confirmação + auto-redirect para ASM-VOTO (2s)

## Integração com backend

| Ação UI | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Carregar status de avaliação | `/api/v1/assembly/:id/hidden-assessments/me/status` | GET | - | `{already_submitted: bool, is_current_pauta_eleicao: bool}` |
| Submeter avaliação | `/api/v1/assembly/:id/hidden-assessments` | POST | `{answers: [{question_id, score}]}` | 201 `{ok: true}` |

Endpoints marcados **inferido** — a formalizar em [[../../../backend/requirements/assembly]] no Sprint 10.

## Regras de negócio críticas

1. **Confidencialidade absoluta**: NENHUMA role — inclusive `admin` — pode consultar avaliação individual. Endpoint `/individual/:morador_id` retorna 403 **sempre** (regra no middleware, não só ABAC).
2. **Morador NÃO vê próprio score** após submeter — apenas confirmação genérica.
3. **Síndico atual NÃO vê linha por linha** — apenas agregação mensal publicada em `governance_score` ao fim do mandato.
4. **morador_hash** é HMAC-keyed com secret rotating (ADR-0028) — impede correlação cross-mandato.
5. **Audit log registra fato**, não respostas (`HiddenAssessmentSubmitted(mandate_id, morador_hash, submitted_at)`).
6. **Idempotência**: mesma (mandate_id, morador_hash) só pode submeter 1× — `UNIQUE (mandate_id, morador_hash)` em DB.
7. **NavGuard**: morador não pode escapar da tela (browser back/refresh/close abre modal "Tem certeza? Sua participação na eleição será bloqueada").
8. **Deadline**: se morador não submeter em ~5min, presidente pode pular o item (regra de assembleia geral). Avaliação fica como "skipped" (sem impacto no score).

## Ligações

- **Tela origem**: ASM-PAINEL-SIND ou ASM-TELAO (item de pauta eleição aberto)
- **Tela destino**: ASM-VOTO (após submit) ou ASM-AVAL-ELEICAO novamente (se erro)
- **Sub-produto**: [[../../../../00-product/sub-produtos/05-assembleia-live]]
- **Backend**: [[../../../backend/requirements/assembly]]
- **Mobile equivalente**: [[../../../mobile/ui-catalog/assembly/ASM-AVAL-ELEICAO]]
- **Invariante**: [[../../../../01-domain/invariants#INV-ASM-023]]
- **Compliance**: [[../../../../04-requirements/compliance-lgpd]] §LGPD-Art.46 (audit trail)

## Gaps/ressalvas

- **Lista literal de perguntas** (3-5) precisa ser confirmada com produto — stub inicial tem 5 perguntas (transparência, decisões, demandas, patrimônio, compliance); validar na Sprint 10
- **Fórmula de agregação** para `governance_score` (fatia 25% em INV-GOV-001): média simples das respostas por pergunta, ou ponderada? Spec a definir
- **UX do deadline**: quando presidente pula item, morador vê mensagem específica? A definir
- **"Recusa a avaliar"**: decisão aberta — morador pode se recusar explicitamente e perder voto no candidato, ou sistema força?

## Links

- [[_moc]]
- [[../../../CLAUDE]]
- [[../../../../01-domain/invariants]]
- [[../../../../STATE]]
- [[../../../../04-requirements/functional/assembly#ASM-ELE-AVAL]]
