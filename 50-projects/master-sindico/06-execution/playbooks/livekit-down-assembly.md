---
title: "Playbook — Livekit Down durante Assembleia (CRÍTICO M3)"
type: playbook
tags: [playbook, incident, master-sindico, ops, assembly, livekit, legal]
severity: P0
trigger: "Livekit room reject rate > 10% durante assembleia ao vivo"
owner: oncall-rotation
created: 2026-04-24
updated: 2026-04-24
---

# Playbook — Livekit Down durante Assembleia ao Vivo

> Bounded context: `assembly`. Provider `ILiveProvider` → `LivekitProvider` (retry 100/300/900ms; reconciliação NotFound — A-033/A-034 fechados). Saga A-028 (`StartLiveSession` com `EndRoom` compensation).
>
> **Crítico para M3** (assembleia 100% online). Em M1/M2 assembleia híbrida — fallback presencial possível. Template já criado em M1 para preparação.

---

## 1. Trigger

Alertas (Sentry / Grafana):

- **`livekit.room.reject_rate > 10%`** durante assembleia com `status=in_progress`
- **`livekit.participant.disconnect_rate > 30%`** janela 1min em sala ativa
- **`livekit.sfu.latency_p95 > 500ms`** janela 2min
- **`livekit.egress.recording_failure`** (gravação obrigatória para ata legal)

Customer report durante assembleia: "perdi conexão", "não consigo entrar", "som cortando".

Dashboards:

- Grafana: `Per-BC dashboard / assembly` → painel "Livekit live sessions" + "WebSocket lag"
- Sentry: tag `bc:assembly` + `provider:livekit` + `assembly.status:in_progress`
- Livekit Cloud Status (se cloud): <https://status.livekit.io>

---

## 2. Severidade + impacto cliente + impacto LEGAL

**P0 absoluto** — ato civil sob curso. Interrupção pode invalidar deliberações:

| Cenário | Sev | Impacto |
|---|---|---|
| Sala não aceita novos participantes durante assembleia | **P0** | Quórum não atinge mínimo legal (Lei 4.591/64 + CC 1.354) |
| Participantes existentes caem em massa | **P0** | Quórum recalcular — pode invalidar votação em curso |
| Egress (gravação) cai | **P0** | Ata legal sem gravação — risco de impugnação 30d (CC 1.349) |
| Livekit funciona mas votação no app falha | **P0** | Votação separada do live; voto pode estar comprometido (ver invariante INV-079) |

SLO afetado: `assembly` 99.9% **durante assembleia ativa** + 0% votos perdidos (observability §3.1).

**Continuidade jurídica é mais crítica que tecnologia** — síndico tem 2min para decisão.

---

## 3. Triagem em < 2min (TEMPO REDUZIDO — assembleia em curso)

### 3.1 Verificar status Livekit + assembleia ativa

```bash
# ✅ Status Livekit (cloud)
curl -s https://status.livekit.io/api/v2/status.json | jq '.status.indicator'

# ✅ Listar rooms ativas via Livekit CLI
livekit-cli list-rooms --url "$LIVEKIT_URL" --api-key "$LIVEKIT_API_KEY" --api-secret "$LIVEKIT_API_SECRET"

# ✅ Logs Railway — assembly + livekit
railway logs --service api-prod --tail 300 \
  | grep -E 'livekit|assembly\..*\.live' | head -40
```

### 3.2 Estado do DB — assembleia em curso

```bash
railway connect postgres-prod
```

```sql
-- ✅ Assembleias atualmente in_progress
SELECT
  id,
  condominium_id,
  type,
  modality,
  status,
  started_at,
  EXTRACT(EPOCH FROM (NOW() - started_at))/60 AS duration_min
FROM assembly.assemblies
WHERE status = 'in_progress'
ORDER BY started_at;

-- ✅ Quórum atual da assembleia X
WITH assembly_id AS (SELECT '<ASSEMBLY_ID>'::uuid AS id)
SELECT
  COUNT(*) FILTER (WHERE presence_type IN ('present_in_person','present_online')) AS present_count,
  COUNT(*) FILTER (WHERE presence_type = 'absent') AS absent_count,
  COUNT(*) AS total_attendance
FROM assembly.attendance_records, assembly_id
WHERE attendance_records.assembly_id = assembly_id.id;

-- ✅ Itens da agenda — algum em votação aberta?
SELECT
  id,
  type,
  status,
  voting_opened_at,
  voting_closed_at
FROM assembly.agenda_items
WHERE assembly_id = '<ASSEMBLY_ID>'::uuid
ORDER BY ordering;

-- ✅ Votos já cast no item em votação (preserve evidência!)
SELECT
  agenda_item_id,
  voter_id,
  vote_type,
  cast_at,
  weight_fracao_ideal
FROM assembly.votes
WHERE agenda_item_id IN (
  SELECT id FROM assembly.agenda_items
  WHERE assembly_id = '<ASSEMBLY_ID>'::uuid AND status = 'voting_open'
);
```

### 3.3 Egress (gravação) ainda rolando?

```bash
# ✅ Listar egress ativos
livekit-cli list-egress --url "$LIVEKIT_URL" --api-key "$LIVEKIT_API_KEY" --api-secret "$LIVEKIT_API_SECRET"

# Se gravação morreu durante assembleia: PRESERVAR CHUNK PARCIAL urgentemente
# Egress salva em S3 (configurado no infra)
aws s3 ls s3://ms-assembly-egress/<ASSEMBLY_ID>/ --recursive
```

---

## 4. Decisão crítica em < 2min (SÍNDICO + ONCALL)

**O síndico decide a continuidade jurídica; o oncall decide a continuidade técnica.** Em paralelo.

### Opção A — Pausar assembleia (RECESSO)

**Quando**: outage curto esperado (< 15min), Livekit retorna baseline rapidamente, votação ainda não fechou.

```text
[ROTEIRO síndico anuncia ao vivo OU via banner app]
"Senhores, estamos com instabilidade técnica na transmissão. Decreto recesso
de 15 minutos. Retomaremos a sessão às <HH:MM>. Pedimos que mantenham seus
dispositivos prontos para reconexão. Esta interrupção será registrada na ata."
```

Backend: NÃO mudar `status=paused` (não existe estado `paused` no enum atual — `AssemblyStatus` tem 5 estados: rascunho/publicada/em_andamento/encerrada/cancelada). Manter `in_progress` e registrar no audit.

```sql
-- ✅ Audit log do recesso
INSERT INTO cross_domain.audit_log_entries (id, actor_user_id, entry_type, target_id, payload, created_at)
VALUES (gen_random_uuid(), '<SYNDIC_USER_ID>', 'assembly.recess_declared',
        '<ASSEMBLY_ID>', jsonb_build_object('reason','livekit_outage','expected_resume_at','<HH:MM>'),
        NOW());
```

### Opção B — Modo presencial-fallback (votação via app sem A/V)

**Quando**: assembleia híbrida (M1/M2); votação no app é independente do Livekit; live era apenas comunicação. Síndico continua presencialmente.

```text
[ROTEIRO]
"Senhores, a transmissão online está indisponível. Continuaremos a assembleia
presencialmente. Os participantes online podem acompanhar e votar pelo app
- a votação não depende da transmissão. Continuamos com o item <N>."
```

Backend: nada precisa mudar — votação `POST /assemblies/:aid/agenda-items/:iid/votes` é independente do Livekit.

### Opção C — Abandonar e reconvocar (juridicamente caro)

**Quando**: outage prolongado (> 30min) sem perspectiva de retomada + nenhum item já votado.

**Custo legal**: nova convocação exige edital com **antecedência mínima 8 dias** (CC 1.354 + convenção típica). Cliente perde calendário.

```text
[ROTEIRO]
"Senhores, devido a indisponibilidade técnica que impede a continuidade legal
da sessão, declaro encerrada esta assembleia sem deliberações. A convocação
de nova assembleia será publicada em até <prazo>, com novo edital."
```

Backend: marcar `status=cancelada`:

```sql
-- ✅ Cancelar assembleia (state machine: in_progress → cancelada permitido)
UPDATE assembly.assemblies
SET status = 'cancelada',
    cancellation_reason = 'livekit outage 2026-MM-DD — sem deliberações',
    cancelled_at = NOW(),
    updated_at = NOW()
WHERE id = '<ASSEMBLY_ID>';

INSERT INTO cross_domain.audit_log_entries (id, actor_user_id, entry_type, target_id, payload, created_at)
VALUES (gen_random_uuid(), '<SYNDIC_USER_ID>', 'assembly.cancelled',
        '<ASSEMBLY_ID>', jsonb_build_object('reason','livekit_outage','duration_min',45),
        NOW());
```

---

## 5. Triagem técnica paralela (oncall, enquanto síndico decide)

### 5.1 Restart adapter

A-033/A-034 já trazem retry backoff 100/300/900ms automático. Se ainda falha:

```bash
# ✅ Restart pod API (drain conexões → adapter reabre com backoff)
railway service restart api-prod

# ⚠️ Cuidado: restart de pod kills WebSocket conns ativos.
# Em assembleia LIVE, prefira NÃO restartar — A-033 retry deve absorver.
```

### 5.2 Migração emergencial Livekit Cloud → self-host (M3+ planejado)

Em M1/M2 não há fallback. Se migrar to ser implementado em M3:

- Manter `LIVEKIT_URL_FALLBACK` env var
- Adapter tenta primary, se falha 3x cai em fallback
- Tokens compatíveis (mesmo `LIVEKIT_API_KEY`)

### 5.3 Preservar gravação parcial

```bash
# ✅ Forçar stop do egress atual (salva chunk parcial em S3)
livekit-cli stop-egress \
  --url "$LIVEKIT_URL" \
  --api-key "$LIVEKIT_API_KEY" \
  --api-secret "$LIVEKIT_API_SECRET" \
  --id <EGRESS_ID>

# ✅ Listar artefatos S3 da assembleia
aws s3 sync s3://ms-assembly-egress/<ASSEMBLY_ID>/ /tmp/assembly-recovery/

# ✅ Backup imediato em bucket WORM (Object Lock 30d) — ata legal
aws s3 cp /tmp/assembly-recovery/ s3://ms-assembly-archive-worm/<ASSEMBLY_ID>/ --recursive
```

---

## 6. Continuidade jurídica

### 6.1 Ata de continuidade

Toda interrupção precisa ser registrada na ata final (CC 1.354):

```text
"Em <DD/MM/YYYY>, às <HH:MM>, durante a assembleia <título>, foi declarado
recesso técnico devido a indisponibilidade da transmissão online. A sessão
foi <retomada às HH:MM | encerrada sem deliberações posteriores>. Estavam
presentes <N> condôminos representando <X%> da fração ideal no momento da
interrupção. <listagem>. As deliberações já tomadas até o momento da
interrupção são <listadas/preservadas/anuladas conforme decisão>."
```

### 6.2 Re-voto vs voto registrado (invariante INV-079)

Caso votação **já fechou** no app antes do crash:

- **Voto vale.** UNIQUE(agenda_item_id, voter_id) garante 1 voto por participante; voto está persistido em DB transacionado.
- Confirmação SQL:

```sql
-- ✅ Votos persistidos com cast_at < interrupção
SELECT COUNT(*), SUM(weight_fracao_ideal)
FROM assembly.votes
WHERE agenda_item_id = '<ITEM_ID>'
  AND cast_at < '<INTERRUPTION_TS>';
```

Caso votação estava **em curso** (não fechou):

- **Re-voto necessário** se síndico optar por opção A (recesso) e item ainda em `voting_open`.
- Síndico fecha votação atual (`POST /agenda-items/:iid/close-voting`) → reabre nova rodada → audita motivo.

### 6.3 Quórum recalcular pós-recovery

```sql
-- ✅ Recalcular quórum após re-checkin (pós-recesso)
WITH assembly_id AS (SELECT '<ASSEMBLY_ID>'::uuid AS id)
SELECT
  COUNT(DISTINCT user_id) FILTER (
    WHERE presence_type IN ('present_in_person','present_online')
      AND (returned_at IS NULL OR returned_at > NOW() - INTERVAL '5 minutes')
  ) AS quorum_now,
  SUM(u.fracao_ideal) FILTER (
    WHERE ar.presence_type IN ('present_in_person','present_online')
  ) AS quorum_fracao_ideal_pct
FROM assembly.attendance_records ar
JOIN institutional.units u ON ar.unit_id = u.id
WHERE ar.assembly_id = '<ASSEMBLY_ID>'::uuid;
```

> **Gap crítico backend** (§8.1 backend/README): `Unit.fracao_ideal` ainda não está implementado em DB (tarefa Sprint 10). Quórum por fração ideal está hackeado em `AgendaItem.fractionIdeal` string. Em M1, **assembleia jurídica plena depende dessa migration**.

---

## 7. Recovery

### 7.1 Verificar volta ao baseline

```bash
# ✅ Smoke test: criar room teste + entrar
livekit-cli create-token \
  --url "$LIVEKIT_URL" \
  --api-key "$LIVEKIT_API_KEY" \
  --api-secret "$LIVEKIT_API_SECRET" \
  --room smoke-test --identity smoke-user \
  --duration 5m

# Cliente teste conecta via SDK; espera-se latência < 400ms p95
```

### 7.2 Retomar assembleia (se Opção A)

Síndico em UI clica "Retomar"; backend não mudou estado durante recesso. Audit:

```sql
INSERT INTO cross_domain.audit_log_entries (id, actor_user_id, entry_type, target_id, payload, created_at)
VALUES (gen_random_uuid(), '<SYNDIC_USER_ID>', 'assembly.recess_resumed',
        '<ASSEMBLY_ID>', jsonb_build_object('recess_duration_min', 12),
        NOW());
```

### 7.3 Emitir ata final com timestamp da interrupção

Use case `Minutes.Generate` deve incluir:

- Timeline de eventos do `audit_log_entries` filtrado por `assembly_id`
- Lista de presentes ao início + lista pós-recesso (diff)
- Itens deliberados antes da interrupção (preservados)
- Itens não-deliberados (reapresentar em próxima)

```sql
-- Exportar timeline pra ata
SELECT entry_type, payload, created_at, actor_user_id
FROM cross_domain.audit_log_entries
WHERE target_id = '<ASSEMBLY_ID>'
ORDER BY created_at;
```

### 7.4 Janela de impugnação

Por convenção típica BR + CC 1.349, condôminos têm **30 dias** para impugnar deliberações. Documentar no comunicado oficial pós-assembleia:

```text
"Os condôminos têm prazo de 30 dias contados de <data> para impugnação
das deliberações desta assembleia, nos termos do art. 1.349 do Código
Civil e da convenção condominial."
```

---

## 8. Comms templates

### 8.1 Banner ao vivo no app (durante incident)

```text
"Atenção: estamos com instabilidade na transmissão. O síndico vai anunciar
os próximos passos. A votação no aplicativo continua funcionando normalmente."
```

### 8.2 Slack `#ms-ops`

```text
[INCIDENT] Sev P0 — Livekit outage DURANTE ASSEMBLEIA
Assembleia: <ID> @ condomínio <CID>
Detectado: <HH:MM UTC>
Sintoma: room reject rate <X%> | participantes online: <antes>→<agora>
Decisão síndico: <recesso | fallback presencial | cancelar>
Egress: <ativo|caído — chunk parcial preservado em S3>
Quórum: <X%> antes; <Y%> agora; mínimo <Z%>
Itens deliberados: <N> preservados | <M> em curso (re-voto necessário?)
Mitigação técnica: <retry adapter ativo | restart pod | aguardando Livekit>
Postmortem: obrigatório.
```

### 8.3 E-mail aos participantes (pós-assembleia, claro e jurídico)

```text
Assunto: [Master Síndico] Comunicado — Assembleia <título> — <data>

Prezados condôminos,

A assembleia realizada em <DD/MM/YYYY> sofreu uma interrupção técnica entre
<HH:MM> e <HH:MM>, conforme registrado em ata.

Status:
- Deliberações já tomadas até <HH:MM>: VÁLIDAS — registradas em ata.
- Itens em votação no momento da interrupção: <preservados | reapresentados em
  nova rodada após retomada>.
- A gravação parcial da sessão está preservada e disponível para os condôminos
  via aplicativo.

Os condôminos têm prazo de 30 dias contados desta data para impugnação das
deliberações, nos termos do art. 1.349 do Código Civil.

Em caso de dúvida, entrar em contato com a administração do condomínio.

Equipe Master Síndico
```

### 8.4 ANPD (LGPD)

**Aplicável SE gravação parcial vazou ou Livekit declarou breach**:

- Gravação contém PII sensível: rostos, vozes, votos, opiniões pessoais
- Vídeo de assembleia é considerado dado pessoal (LGPD Art. 5º, I)
- Se vazou → P0 absoluto → seguir `cross-tenant-leak.md` §6
- T+1h DPO; T+72h ANPD via Res. 15/2024

---

## 9. Pós-incidente

### 9.1 Postmortem (obrigatório, ≤ 48h, blameless)

Toda interrupção de assembleia **sempre** dispara postmortem, mesmo que < 5min — implicação legal.

### 9.2 Métricas a coletar

- Duração total da interrupção (T_detect → T_resume)
- Participantes desconectados em massa (count + %)
- Itens em votação afetados (preservados vs re-votados)
- Egress: gravação completa, parcial ou perdida
- Decisão tomada (recesso/fallback/cancelar)
- Eventual impugnação 30d pós-assembleia (acompanhar)

### 9.3 Action items típicos

- Implementar `assembly.status = paused` (estado dedicado para recesso) — Sprint 11
- Backup contínuo egress em bucket WORM (Object Lock 30d) — confirmar config M2
- Health check pré-assembleia: 30min antes do início, smoke test Livekit + alerta proativo
- Migration `00211_add_unit_type_and_fracao_ideal.sql` — pré-requisito para quórum por fração ideal real (Sprint 10 alta prioridade)
- Treinamento síndico: roteiro de comunicação durante interrupção (UI helper text)

---

## 10. Escalação

| Quando | Quem | Como |
|---|---|---|
| Sev P0 durante assembleia ativa | Tech Lead + DPO + Suporte Síndico | PagerDuty escalation IMEDIATA |
| Decisão jurídica (cancelar vs continuar) | Tech Lead + Síndico + (opcional: jurídico cliente) | reunião em < 5min |
| Livekit Cloud outage > 30min | Livekit Support | <https://docs.livekit.io/cloud/support> + status page; planos pagos têm SLA |
| Egress falhou — gravação perdida | Tech Lead + DPO | preservar evidência; comunicar cliente sobre ausência de gravação |
| Impugnação por condômino pós-assembleia | Suporte cliente + jurídico cliente | encaminhar; backend tem audit_log completo |

Contatos:

- Livekit project ID: ver `1Password / Master Síndico / Livekit`
- Livekit API key/secret: `LIVEKIT_API_KEY`, `LIVEKIT_API_SECRET` em Railway
- DPO interno: `dpo@mastersindico.com.br`
- Suporte síndico (in-app): canal dedicado durante M1

---

## 11. LGPD checklist

- [ ] Gravação parcial preservada com controle de acesso (não pública)?
- [ ] Logs nossos não expuseram tokens Livekit JWT em claro? (`grep -E 'lk_token|livekit' sentry/exports/`)
- [ ] Participantes ID/CPF não vazaram em logs do adapter? (verificar)
- [ ] Egress chunks em S3 com criptografia + Object Lock?
- [ ] Comunicação aos titulares mencionou janela 30d impugnação?
- [ ] Se PII vazou (gravação pública por engano, etc) → escalar `cross-tenant-leak.md` §6 (T+1h DPO; T+72h ANPD)

Livekit é processador (Art. 5º, VIII LGPD); nós controladores. **Voto = dado sensível** (opinião política condominial). Em breach lateral, **somos** responsáveis pela notificação ANPD + titulares.

---

## 12. Vizinhos

- [[../incident]] — fluxo geral incident response
- [[../rollback]] — procedimento rollback Railway (CUIDADO durante assembleia ativa)
- [[../../11-audit/postmortems/template]]
- [[../../03-subprojects/backend/security]] §6 (HMAC), §16 (A-033/A-034 fechados)
- [[../../02-architecture/observability]] §3.1 (SLO assembly 99.9% durante live)
- [[../../08-security/threat-model]] §6.4 (Livekit compromise), §4.4 (WebSocket STRIDE), §8 (Vote integrity)
- [[../../01-domain/bounded-contexts|bounded-contexts]] — assembly context + Saga A-028
- [[mux-down]] — Mux estava em estudo para live (cancelado em favor de Livekit)
- [[cross-tenant-leak]] — gravação ou voto vazando entre condomínios
