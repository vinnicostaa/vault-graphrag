---
title: "Sub-produto 5 — Assembleia Live"
type: spec
tags: [sub-produto, assembleia, livekit, votacao, ata, master-sindico, v2, fase-8]
source: requirements/assembly.md + cross-domain.md + institutional.md + functional-matrix.md + backend/assembly/* + Content/03-Modulos/Assembleia-Live-Funcional.md + Obsidian/Dominio-Assembleia + Obsidian/System-Architecture-Assembly
created: 2026-04-23
updated: 2026-04-23
aliases: [assembleia-live, modulo-assembleia, live-assembly]
status: destilado
dual_check: pending
---

# Sub-produto 5 — Assembleia Live

> Sub-produto mais ambicioso tecnicamente e de maior valor jurídico da plataforma. Automatiza o ciclo condominial brasileiro válido: configuração → edital → ciência → quórum → votação com fração ideal → ata imutável com score de transparência. Livekit é provedor abstrato (swap possível). MVP = presencial; hibrida/online inativas.

Links essenciais: [[_moc]] · [[../../STATE|STATE]] · [[../../AUDIT|AUDIT]] · [[01-governanca-institucional]] · [[02-connect-me]] · [[03-reputacao]] · [[04-conteudo-videos]] · [[06-lms-academia]] · [[09-compliance]] · [[11-administradoras-condominiais]]

---

## § 1. Fontes consultadas

| # | Path | Data | Papel |
|---|---|---|---|
| 1 | `backend/.kiro/specs/master-sindico/requirements/assembly.md` | 2026-04-22 | Canônico Req 48-60.12 (3 fases) |
| 2 | `backend/.kiro/specs/master-sindico/requirements/cross-domain.md` | 2026-04-22 | Req 61-68 (Livekit = Req 68), Req 103 (visibilidade vídeo), Req 37 (audit append-only) |
| 3 | `backend/.kiro/specs/master-sindico/requirements/institutional.md` | 2026-04-22 | Req 11 (Timeline `assembly_result`), Req 9 (Unit fração ideal) |
| 4 | `backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` | 2026-04-22 | Paywalls: voto exige Morador Pagante+, criar = síndico trial+ |
| 5 | `backend/internal/modules/assembly/module.go` | 2026-04-22 | Wiring completo (4 handlers + liveProvider) |
| 6 | `backend/internal/modules/assembly/domain/entities/assembly.go` | 2026-04-22 | Aggregate root state machine |
| 7 | `backend/internal/modules/assembly/domain/entities/agenda_item.go` | 2026-04-22 | Lock após publish + fraction string |
| 8 | `backend/internal/modules/assembly/domain/entities/vote.go` | 2026-04-22 | APPEND-ONLY, sem setters |
| 9 | `backend/internal/modules/assembly/domain/entities/minutes.go` | 2026-04-22 | `TransparencyComponents` 10 campos |
| 10 | `backend/internal/modules/assembly/domain/entities/proxy.go` | 2026-04-22 | Token hex 32B, validUntil = scheduledDate+24h |
| 11 | `backend/internal/modules/assembly/domain/entities/attendance_record.go` | 2026-04-22 | checkOutAt nullable |
| 12 | `backend/internal/modules/assembly/domain/entities/science_record.go` | 2026-04-22 | Imutável, editalVersion default "v1" |
| 13 | `backend/internal/modules/assembly/domain/enums/assembly.go` | 2026-04-22 | 9 enums reais |
| 14 | `backend/internal/modules/assembly/domain/providers/i_live_provider.go` | 2026-04-22 | `ILiveProvider` 4 métodos |
| 15 | `backend/internal/modules/assembly/infrastructure/providers/livekit_provider.go` | 2026-04-22 | A-033/A-034 retry backoff 100/300/900ms |
| 16 | `backend/internal/modules/assembly/application/use_cases/live_use_cases.go` | 2026-04-22 | Saga compensação EndRoom |
| 17 | `backend/internal/modules/assembly/application/use_cases/assembly_use_cases.go` | 2026-04-22 | End gera Minutes + score |
| 18 | `backend/internal/modules/assembly/application/use_cases/vote_use_cases.go` | 2026-04-22 | `HasVoted` pré-check + UNIQUE DB |
| 19 | `backend/internal/modules/assembly/application/use_cases/proxy_use_cases.go` | 2026-04-22 | `generateProxyToken` crypto/rand 32B hex |
| 20 | `backend/internal/modules/assembly/application/use_cases/attendance_use_cases.go` | 2026-04-22 | CheckIn só em `em_andamento` |
| 21 | `backend/internal/modules/assembly/application/use_cases/science_use_cases.go` | 2026-04-22 | Bloqueia se edital não publicado |
| 22 | Migrations `00500_create_assembly_enums.sql` → `00507_create_minutes.sql` | 2026-04-13 | 8 tabelas + 9 enums PG |
| 23 | `Development/Content/03-Modulos/Assembleia-Live-Funcional.md` | 2026-04-22 | **Fonte canônica de produto** — 5 blocos + ~40 regras |
| 24 | `Development/Content/02-Jornadas/Inventario-Telas.md` | 2026-04-22 | Seção Assembleia: 5 etapas, ~30+ telas |
| 25 | `Obsidian Vault/Clients/MasterSindico/02 - Domínios/Domínio - Assembleia.md` | 2026-03-10 | v3.0 — endpoints REST + WS event names |
| 26 | `Obsidian Vault/Clients/MasterSindico/01 - Arquitetura/System Architecture - Assembly.md` | s/d | Infra WS + NATS + Redis + outbox |

---

## § 2. Propósito (ciclo jurídico válido BR)

Cita `Development/Content/03-Modulos/Assembleia-Live-Funcional.md:17-24 (2026-04-22)`:
> "Organização prévia da assembleia · Convocação e rastreabilidade de ciência · Preparação de quórum, procurações, fração ideal e elegibilidade de voto · Condução da assembleia (presencial, híbrida ou online preparadas mas inativas no MVP) · Apoio à fala, votação e deliberação · Transparência operacional em tempo real · Geração de histórico, trilha de auditoria e base documental pós-assembleia".

**Objetivo contratual**: digitalizar Art. 1349 CC + Lei 4591/1964 (convenção, convocação 5 dias, quórum, ata). Gera comprovante judicialmente aceitável (append-only + ciência rastreada + assinatura digital gov.br Lei 14.063/2020).

---

## § 3. O que NÃO é

Cita `Content/03-Modulos/Assembleia-Live-Funcional.md:26-37 (2026-04-22)`:

| NÃO substitui | NÃO é |
|---|---|
| Convenção do condomínio | Google Meet / Zoom (não é webinar) |
| Administradora | Formulário Google para voto |
| Presidente da mesa | WhatsApp de dúvida |
| Assessoria jurídica | Blockchain de votos |
| Formalização externa da assinatura | Assembleia assíncrona por dias |

**É**: meio tecnológico de organização + ambiente de transparência + apoio operacional.

---

## § 4. Personas

| Persona | Poder | Fonte |
|---|---|---|
| **Síndico (principal)** | Cria, publica edital, inicia, preside, encerra, homologa | `institutional.md:220` R1 matriz |
| **Síndico auxiliar** | Pode validar execução/registro, sem presidir | `institutional.md:222` |
| **Administradora** | Recebe token convite; modo read+observações; sobe fração ideal + inadimplência; valida procurações; libera voto de unidade regularizada; homologa | `Content/Assembleia-Live-Funcional.md:56-68 (2026-04-22)` |
| **Conselho/Auditor** | Read + observações (opcional cadastrar antes) | `Content/Assembleia-Live-Funcional.md:235-239` |
| **Morador titular** | Ciência, confirma presença, procuração, check-in, voto (fração dele) | `assembly.md:236-284` Req 56-57 |
| **Dependente** | Sem voto; leitura apenas | `institutional.md:200` Req 9 |
| **Fornecedor presente** | Cadastrado no painel inicial (nome, representante) — observador | `Content/Assembleia-Live-Funcional.md:238` |
| **Admin Master (privilegiado)** | Read global + audit trail; não vota | `functional-matrix.md:131-142` |

---

## § 5. Tiers × role (plano global Stripe-style)

Cita `functional-matrix.md:36-40 (2026-04-22)`:

| Ação | Morador Base (free) | Morador Pagante | Síndico Trial | Síndico Base | Síndico Plus | Síndico Pro | Admin |
|---|---|---|---|---|---|---|---|
| Ver pauta | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Dar ciência | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Votar** | ❌ paywall | ✅ | n/a | n/a | n/a | n/a | ✅ |
| Procuração | ❌ | ✅ | n/a | n/a | n/a | n/a | ✅ |
| Criar assembleia | n/a | n/a | ✅ | ✅ | ✅ | ✅ | ✅ |
| Publicar pauta | n/a | n/a | ✅ | ✅ | ✅ | ✅ | ✅ |
| Painel presidente | n/a | n/a | ✅ | ✅ | ✅ | ✅ | ✅ |
| Relatório export | n/a | n/a | ⏳ trial | ⏳ | ✅ | ✅ | ✅ |
| Homologar | n/a | n/a | ✅ | ✅ | ✅ | ✅ | ✅ |

**Nota tier global**: spec legacy usa N1/N2/N3, mas overlay canônico é trial/base/plus/pro (`personas-and-plans.md`). Admin = role privilegiada, não tier. Votar = primeiro hard-paywall: só Pagante+ pode votar (sentido jurídico: trial grátis vê pauta mas não muda resultado).

**Hard paywall**: `functional-matrix.md:186-195` — `Votar em assembleia | Morador Base`.

---

## § 6. Os 5 Blocos

### Bloco 1 — Configuração Estrutural do Condomínio

Cita `Content/Assembleia-Live-Funcional.md:50-68 (2026-04-22)`:

| Campo base | Valor |
|---|---|
| Nome, endereço, blocos, unidades | obrigatórios |
| Síndico principal (confirmar) | 1 |
| Subsíndico | opcional |
| Conselho | opcional |
| Administradora | opcional (vínculo por CNPJ Master OU convite externo por token) |
| Auditor | opcional |

**Vínculo administradora externa** (6 passos):
1. Síndico informa empresa + CNPJ + responsável + CPF + e-mail + telefone
2. Sistema envia convite
3. Administradora valida acesso por token
4. Confirma CPF do responsável
5. Aceita termo de responsabilidade
6. Ganha acesso APENAS ao módulo assembleia daquele condomínio (sandbox)

**Permissões administradora** (11 toggles): visualizar em construção · adicionar observações · gerar edital automático · anexar edital pronto · subir planilha fração ideal · subir planilha inadimplência · validar procurações · liberar voto de unidade regularizada · registrar voto presencial assistido · homologar votação · encerrar assembleia.

---

### Bloco 2 — Pré-Assembleia

#### 2.1 Criação (`assembly.md:18-41 (2026-04-22)` + `Content/...Funcional.md:74-85`)

| Campo | Enum/tipo | Obrigatório |
|---|---|---|
| `title` | string 3-200 chars | sim |
| `description` | text | não |
| `type` | `ordinaria\|extraordinaria\|especial` (código) / ou `implantação` (produto) | sim |
| `modality` | `presencial\|remota\|mista` — **MVP presencial só** | sim |
| `scheduled_date` | TIMESTAMPTZ (futura, ≥5 dias p/ notificação) | sim |
| 1ª + 2ª convocação | horários distintos | produto |
| `location` | endereço | se presencial |
| `link_transmissao` | URL | remota/mista (inativo MVP) |
| `speech_time_minutes` | SMALLINT `IN (2,3)` | sim (CHECK DB) |
| Prorrogação fala | +2min (limite 5) | produto |
| `quorum_type` | `simple_majority\|qualified_majority\|unanimous` | sim |
| `created_by` | user_id | sim |

**Regra anti-duplo**: máx 1 assembleia ativa/condomínio (`assembly.md:34`). Cancelamento até 3 dias antes.

#### 2.2 Pauta — itens (cita `assembly.md:48-85 (2026-04-22)` + produto 5 tipos `Content/...Funcional.md:98-112`)

**Divergência enum**: código atual tem 5 tipos ≠ produto.

| Código (`enums/assembly.go`) | Produto (`Content/...Funcional.md`) |
|---|---|
| `aprovacao_contas` | `informativo` |
| `eleicao_sindico` | `votacao_simples` |
| `obras_melhorias` | `votacao_fracao_ideal` |
| `reajuste_taxa` | `escolha_fornecedor` |
| `outros` | `eleicao` |

→ **Gap G-02**: enum `AgendaItemType` em código categoriza por tema (contas/eleição/obras/taxa/outros), spec canônica usa mecânica de voto (informativo/simples/fração/fornecedor/eleição). Precisa reconciliação; duas dimensões ortogonais.

Item contém: `title`, `description`, `item_type`, `order_position` SMALLINT, `fraction_ideal` NUMERIC(6,4) nullable, `is_locked` BOOL, `status` (pendente/votando/aprovado/rejeitado/retirado), + atributos produto: impacto financeiro sim/não, valor estimado, quórum exigido p/ item, anexo doc + vídeo explicativo.

Produto: pauta ilimitada; spec MUST: mín 1, máx 20.

#### 2.3 Fração ideal (`assembly.md:66-72` + `Content/...Funcional.md:170-177`)

**Upload planilha Excel** (MVP):
- Colunas: `bloco · unidade · fracao_ideal · cpf_titular`
- Soma deve `= 100%` ou parâmetro equivalente
- Sem duplicidade `(bloco,unidade)`
- Sem campos obrigatórios vazios
- Storage: **NUMERIC(6,4)** em PostgreSQL (código) vs NUMERIC(5,4) em spec (`assembly.md:71`) — **Gap G-32**: migration 00502 usa 6,4 enquanto spec diz 5,4. Ambos atendem precisão jurídica; divergência documental.
- Persistência: import único, permanece salvo; nova import só por correção/mudança estrutural
- Lock após publicação: adendo só (Req 25)

**Gap crítico Fase 7d (G-01)**: `Unit` (tabela `00202_create_units_memberships.sql`) NÃO tem campo `fracao_ideal`. Fração ideal atualmente vive apenas em `agenda_items.fraction_ideal` (peso do item — errado, deveria ser peso da unidade). Produto prevê planilha→persistência na entidade Unit. **Votar com fração ideal hoje requer passar `fraction_ideal` no payload do voto** (`CastVoteCommand.FractionIdeal *string`, `vote_use_cases.go:24`) — não há enforcement de que o valor corresponde à unidade do votante.

#### 2.4 Edital (`assembly.md:117-133` + `Content/...Funcional.md:117-124`)

**2 formas**:
1. **Anexo livre**: administradora/síndico anexa PDF pronto
2. **Gerador automático**: template preenchido com identificação condomínio + tipo + data + horários 1ª/2ª convocação + modalidade + local/link + ordem do dia (import pauta) + regras participação + procuração + votação + logo admin + campo assinatura

**Quem pode gerar**: síndico OU administradora com permissão.

**Persistência**: `assemblies.edital_pdf_url TEXT` + `edital_published_at TIMESTAMPTZ` (`00501_create_assemblies.sql:14-15`). Use case `PublishEditalUseCase` (`assembly_use_cases.go:132-161`).

**Assinatura digital**: Sprint 3+ (`assembly.md:128`), Lei 14.063/2020 (gov.br).

#### 2.5 Ciência obrigatória (`assembly.md:137-152` + `Content/...Funcional.md:128-133`)

- Modal/banner imperativo: "Li e entendi a pauta"
- **App fica travado** até ciência (middleware antes de qualquer request)
- Registra `acknowledged_at`, `edital_version` (default `"v1"`), user_id
- Tracking: **6 meses** (log de quem deu ciência, quando, quem não deu, quem não abriu)
- Auto-unlock pós-encerramento
- Síndico/admin veem %, lista nominal pendentes, quem NÃO abriu
- Entidade: `ScienceRecord` imutável (`entities/science_record.go:6-25`)
- UNIQUE `(assembly_id, user_id)` (migration `00506`)

#### 2.6 Confirmação de participação (opcional)

4 valores: `participarei presencialmente · participarei online · ainda não defini · não participarei` (`Content/...Funcional.md:135-137`).

#### 2.7 Lembretes automáticos (`assembly.md:89-114`)

| Tempo | Canal |
|---|---|
| T-3 dias | Push + Email + SMS (opcional) |
| T-1 dia | Push + Email |
| T-3 horas | Push |

Job via scheduler NATS/cron a cada hora. Tracking de quem clicou.

#### 2.8 Enquetes preliminares (`assembly.md:157-168` + `Content/...Funcional.md:143-146`)

Permitidas para `votacao_simples` + `votacao_fracao_ideal`. Opções **apenas sim/não** (produto) — spec prevê 3 tipos (sim/não/não sei, múltipla, escala 1-5). **Divergência G-33**.

**Sem valor jurídico** (disclaimer obrigatório). Resultado publicado na pauta. Valem até 24h antes da assembleia.

#### 2.9 Fornecedores (`Content/...Funcional.md:148-152` + Req 103 `cross-domain.md:462-477`)

- Moradores acessam perfil só durante período da assembleia
- **Fornecedor Master**: selo + vídeo institucional
- **Fornecedor externo**: dados básicos + proposta + escopo + aviso "empresa não verificada"
- **Visibilidade vídeo**: `video_visibility_grants` unlock momentâneo; NATS `assembly.voting.closed` → `video.visibility.revoked` (Req 103)

#### 2.10 Procurações (`assembly.md:172-189` + `Content/...Funcional.md:154-168`)

**Prazo para cadastrar**: até o síndico clicar "Iniciar assembleia" (`Content/...:155`).

**Fluxo morador**:
- Após ciência, pode cadastrar
- Campos: bloco+unidade representado, nome+CPF representado, bloco+unidade procurador, nome+CPF procurador, checkbox ciência+responsabilidade
- **Mensagem final**: "Você precisa apresentar a procuração fisicamente para validação"

**Fluxo administradora**:
- Visualiza dados
- Ações: aprovar (voto replica com peso da unidade representada, respeitando frações individuais) OU rejeitar com justificativa

**Regras spec**:
- Procurador = morador mesmo condomínio (spec) — mas produto permite externo (Gap G-34)
- Válida até 24h após encerramento (janela voto atrasado)
- **Replica voto exato do titular** (`assembly.md:181`): se titular vota, procurador não vota diferente
- Token hex 32B crypto/rand (`proxy_use_cases.go:92-97`)
- Status: `pendente → validada` (por admin) → usada; OU `expirada/revogada`
- `validUntil = scheduledDate + 24h` (código: `proxy_use_cases.go:77`)
- UNIQUE `(assembly_id, grantor_id)` (código checa `FindByGrantor`)
- Cancelamento titular: até 1h antes

#### 2.11 Inadimplência (`Content/...Funcional.md:179-186`)

- Upload **até 1h antes da 1ª convocação** (aviso crítico `Obsidian/Domínio-Assembleia.md:56`)
- Quem: administradora OU síndico
- Classifica unidades: apto/inapto
- **Liberação manual** (comprovante última hora): botão "Liberar voto" — só síndico/admin; campos: checkbox aprovação, responsável, horário
- **Falha em subir = bloqueio total publicação** (regra forte do PDF, não no spec)

#### 2.12 Simulador de quórum (`assembly.md:194-206` + `Content/...Funcional.md:188-189`)

Endpoint: `GET /assemblies/{id}/simulator?present_fraction=0.60` (spec). Output: total unidades, total frações, % ciência, % confirmação presença, procurações pendentes/aprovadas, **quórum projetado POR ITEM**. Cálculo puro, sem persistência.

#### 2.13 Termos obrigatórios pré-assembleia (`Content/...Funcional.md:191-201`)

5 aceites antes de ciência/participação:
1. Termo de ciência da convocação
2. Termo LGPD
3. Declaração de veracidade das informações
4. Declaração de identidade e legitimidade do acesso da unidade
5. Checkbox autorização de gravação (se houver)

**Texto declaração**: "Declaro, sob minha responsabilidade, que participo desta assembleia na condição legítima vinculada à unidade informada e que as informações prestadas são verdadeiras."

→ Isenção da Master Síndico quanto à verificação material.

---

### Bloco 3 — Assembleia ao Vivo

#### 3.1 Formas de check-in (3) (`assembly.md:212-233` + `Content/...Funcional.md:207-212`)

| Método | Fluxo | Observações |
|---|---|---|
| **App** | Botão ou QR próprio celular | Biometria/PIN Sprint 3+ |
| **QR Code** | Escaneia QR da assembleia (gerado p/ cada assembleia `Content/...:126`) | Cada assembleia = 1 QR próprio |
| **Terminal kiosk** | Tablet/notebook na entrada; CPF+bloco+unidade | Valida CPF contra base unidades |
| **Manual** | Síndico confirma presença (ex: deficiente visual) | Registra operador |

**Constraint**: CheckIn SÓ em `status = em_andamento` (`attendance_use_cases.go:55-57`). UNIQUE `(assembly_id, user_id)` (`00503`).

WebSocket event: `assembly.attendance.updated` (<200ms latência p95).

#### 3.2 Voto presencial assistido (`assembly.md:258-262` + `Content/...Funcional.md:214-218`)

Quem entrou por terminal → voto lançado por síndico/admin.

Registra: `unit`, `horário`, `operador_responsavel`, `is_assisted=true`, `termo_de_voto` (texto obrigatório — entidade `Vote` retorna `ErrVoteRequiresTermoDeVoto` se `isAssisted && termoDeVoto==nil`).

Voto assistido exige assinatura do termo: "morador confirma que autorizou voto" (`assembly.md:261`).

#### 3.3 Tratamento do abandono (`assembly.md:265-278` + `Content/...Funcional.md:220-230`)

**Estados presença**: `presente | ausente_momentaneo | por_procuracao` (enum `PresenceType`).

Produto adiciona 4º: `desconectado/saiu` (não mapeado no enum atual).

**Regra voto**:
- Votos já computados → continuam válidos
- Itens não votados enquanto ausente → **abstenção automática** (spec: após 15 min; produto: indisponível+abstenção)
- Se retornar → volta a poder votar nos abertos

**Transparência**: telão mostra presentes · online ativos · ausentes/desconectados (evita fraude, mostra base votante).

**Entidade**: `AttendanceRecord.MarkTemporaryAbsence(at)` seta `checkOutAt`; `MarkReturn(at)` zera `checkOutAt` e atualiza `checkedInAt` (`attendance_record.go:56-65`).

#### 3.4 Painel inicial (`Content/...Funcional.md:232-239`)

Cadastro obrigatório ANTES do início:
- Presidente da mesa
- Até 3 secretários
- Administradora presente: nome empresa + nome pessoa + função + CPF
- Auditor (opcional): nome + função + CPF
- Fornecedores presentes (opcional): nome empresa + representante

#### 3.5 Papéis do presidente (`Content/...Funcional.md:241-251`)

Abrir formalmente · conduzir ordem · chamar item · autorizar/ordenar falas · abrir/encerrar votação · declarar item resolvido/adiado/prejudicado · encerrar discussão · autorizar reabertura · declarar resultado · encerrar formalmente.

#### 3.6 Início da assembleia (`Content/...Funcional.md:253-255`)

Síndico/admin clica **Iniciar** → sistema **roda vídeo curto Master Síndico** explicando regras (fala, votação, deliberação).

Estado: `publicada → em_andamento` (entidade `Assembly.Start()` exige `IsEditalPublished()`, senão `ErrAssemblyEditalRequired`).

#### 3.7 Apresentação (`Content/...Funcional.md:257-260`)

Síndico/admin/auditor apresenta material SEM sair da plataforma. Formatos: PDF, imagem, vídeo. **PowerPoint deve ser convertido em PDF antes** (explícito no spec).

#### 3.8 Telão 2 áreas (`assembly.md:315-339` + `Content/...Funcional.md:262-275`)

**Área 1 — Apresentação**: documentos apoio (PDF/vídeo/imagem), vídeos empresa durante votação fornecedor.

**Área 2 — Painel operacional**:
- Pauta toda com destaque do item atual
- Unidades presentes
- ~~Unidades online~~ (desabilitar MVP)
- Unidades ausentes
- Fila de mãos levantadas
- Cronômetro da fala
- Status dos itens
- **Quórum presente** + **Quórum necessário**
- Relógio da assembleia

Componentes SolidJS: `PresidentScreen`, `OperationalPanel`. WebSocket 1 conexão + múltiplos listeners. **Latência <200ms p95** (NFR crítico).

#### 3.9 Status dos itens (`Content/...Funcional.md:277-281`)

4 status produto: `resolvido | adiado | não_resolvido | prejudicado`.

Código tem: `pendente | votando | aprovado | rejeitado | retirado` (enum `AgendaItemStatus`).

**Divergência G-03**: mapear — `aprovado→resolvido(sim)`, `rejeitado→resolvido(não)`, `retirado→prejudicado`, sem correspondente p/ `adiado`.

Tracking por item: hora abertura · hora entrada em votação · hora encerramento · duração total discussão.

#### 3.10 Controle de fala — fila (`assembly.md:342-367` + `Content/...Funcional.md:283-293`)

**Fluxo**:
1. Morador aperta "levantar mão"
2. Entra na fila (FIFO)
3. Pode abaixar a mão
4. Síndico/presidente chama unidade pelo microfone (alternado p/ melhor captação+transcrição)
5. Cronômetro inicia
6. **Apenas falas microfonadas vão p/ ata** — orientar síndico a avisar antes

**Tempo**: 2 ou 3 min (config). Extensão: +2min (até 5 total). Alerta 30s antes. Timeout automático → corte mic via Livekit API (`assembly.md:353` Req 60.5).

Painel síndico mostra: fila de espera, tempo restante.

Tabelas: `assembly_speech_queue`, `assembly_speeches` (spec — **não implementadas** no código atual — G-11).

WebSocket: `/ws/assemblies/{id}/speech`.

#### 3.11 Votação (`assembly.md:238-284` + `Content/...Funcional.md:297-311`)

**Tipos de voto produto**:
- Sim / Não / Abstenção (votação simples)
- Escala 1-5 + Abstenção (escalada)

**Tipos de voto código** (enum `VoteType`): `sim | nao | abstencao` (3). Escala 1-5 **não implementada** (G-05/G-18).

**4 tipos de votação (spec)**:
1. Simples → maioria >50%
2. Fração ideal → >50% de fração
3. Fornecedor → escolha entre 2+ propostas
4. Eleição → escolha de síndico

**Fluxo**:
1. Síndico abre votação (painel presidente)
2. Morador recebe notificação
3. Clica e vota (opção única, sem mudança pós-envio)
4. Resultado em tempo real no painel
5. Síndico encerra (ou timeout)
6. Resultado **imutável** (Rule 2)

**Regra procuração**: procurador vota → voto replicado p/ unidade representada considerando fração ideal.

**Invariantes**:
- `fraction_ideal`: NUMERIC(6,4), nunca float
- Voto imutável pós-envio
- Timestamp + IP + device fingerprint (audit trail)
- WebSocket <200ms p95
- **UNIQUE `(agenda_item_id, voter_id)`** DB (`00504_create_votes.sql:14`) — **A-025 RESOLVIDO**
- Use case checa `HasVoted` antes + DB UNIQUE backstop (`vote_use_cases.go:83-89`)

**Entidade Vote APPEND-ONLY** (só getters, sem setters — `entities/vote.go:78`).

**Telão durante votação** (`Content/...Funcional.md:306-307`): pauta completa · item atual · unidades presentes/ausentes · fila fala · cronômetro · relógio · quórum presente/necessário · **votos por unidade**.

**Ausência momentânea no voto**: 15 min p/ voltar (spec `assembly.md:267`); depois abstenção automática. Não há timer automático implementado (G-35).

#### 3.12 Modo contingência (`assembly.md:371-385` + `Content/...Funcional.md:353-359`)

Ativação **manual** (síndico OU admin). Razões (enum `ContingencyReason`): `internet_failure | device_failure | digital_voting_unavailable`.

Quando ativado:
- Continua assembleia com voto presencial assistido (em massa na mesa)
- Itens afetados marcados com flag "contingência ativada"
- WebSocket cai? App local salva votos em Redis `assembly:{id}:offline_votes:{user_id}`
- Auto-sync quando volta
- Conflito: último voto válido
- Notificação: "Modo offline — seus votos serão sincronizados"

Registra: hora ativação, hora normalização, ID do ativador.

**Tabelas spec não implementadas**: nenhum campo `contingency_active` na migration `00501`. **Gap G-08**.

#### 3.13 Termos legais (`assembly.md:388-407`)

Aceite obrigatório ANTES do primeiro check-in: regras participação · direitos/deveres · penalidades. Versão rastreada, timestamp+IP. Tabela `assembly_legal_acceptances` (spec — **não implementada** — G-12).

---

### Bloco 4 — Encerramento

#### 4.1 Homologação (`assembly.md:472-494` + `Content/...Funcional.md:309-311`)

**Obrigatória** após cada votação. Estado temporário → clique "Homologar Votação" (síndico OU admin) → **lock definitivo**.

**Produto** (`Content/...Funcional.md:310`): encerrar votação → homologação → finalização do item (resolvido/adiado/não_resolvido/prejudicado).

**Spec** (`assembly.md:476-488`): ata gerada → votação separada "Aprova a ata?" → >50% → `homologated_at` preenchido → imutável.

**Divergência**: duas homologações distintas — (a) por voto (produto) e (b) da ata inteira (spec). Código atual **não implementa homologação explícita**; `Assembly.End()` gera `Minutes` diretamente (`assembly_use_cases.go:255-325`). Gap G-09.

Tabela `assembly_homologations` (spec — **não implementada**).

**Enum `HomologationStatus`** (spec): `pending | approved | rejected`.

#### 4.2 Encerramento e geração ata (`assembly.md:414-443` + `Content/...Funcional.md:315-317`)

**Fluxo código** (`assembly_use_cases.go:247-325`):
1. Síndico clica "Encerrar"
2. Validação: sem votações abertas (scan items `status ∈ {pendente, votando}` → `allVoted=false`)
3. `Assembly.End()` → status `em_andamento → encerrada`, `ended_at = now`
4. Lista items + conta ciências
5. Calcula `TransparencyComponents` (10 campos)
6. Monta conteúdo markdown (título + data + tipo/modalidade + descrição + lista pauta)
7. `NewMinutes(score=calc)` → persiste `assembly_minutes` (UNIQUE por assembly_id, `00507`)
8. Log info + retorna `MinutesID + TransparencyScore`

**Registro final produto** (campos): data início, data término, presidente, secretários, administradora, auditor, fornecedores presentes, votos, procurações, inadimplentes, liberações de voto.

#### 4.3 Conteúdo ata (`assembly.md:425-438`)

- Data, hora, local
- Presentes (**lista anonimizada** — spec)
- Pauta e resultado de cada votação
- Fala de cada morador (texto ou resumo)
- Procurações usadas
- Assinatura digital do síndico (Sprint 3+)

**Ata IMUTÁVEL pós-publish** (`Minutes.Publish()` retorna erro se já publicada — `minutes.go:114-121`). Correção só via **adendo** (Req 25 commercial).

**Assinatura digital gov.br** (Lei 14.063/2020): Sprint 3+. PDF gerado via template HTML → wkhtmltopdf OU Puppeteer (async). Storage MinIO.

---

### Bloco 5 — Pós-Assembleia e Histórico

#### 5.1 Notificações pós-encerramento (`assembly.md:534-549`)

- "Assembleia encerrada, ata disponível"
- "Ata foi homologada"
- "Score de transparência: X/100"

Canais: push + email + SMS (opt-in). Personalizadas (mostra votações relevantes p/ cada morador).

#### 5.2 Relatórios (6 tipos) (`assembly.md:446-468` + `Content/...Funcional.md:322-328`)

| # | Tipo | Conteúdo |
|---|---|---|
| 1 | Presença | Lista presentes/ausentes, fração, horário check-in, método |
| 2 | Votação | Resultado detalhado por item, votos por opção |
| 3 | Procurações | Lista usadas (quem outorgou, quem representou) |
| 4 | Fala (spec) / Trilha auditoria (produto) | Registro falas + tempo + tópicos; OU trilha completa |
| 5 | Decisões (produto) / Consolidado (spec) | Resumo executivo 1-2 páginas |
| 6 | Transparência | Score 0-100 detalhado |

Formato: CSV + PDF. Endpoint: `GET /assemblies/{id}/reports/{type}?format=csv|pdf`. **Anônimo**: lista presença NÃO mostra nomes (só unidade/fração). Confidencial: só síndico+admin.

#### 5.3 Timeline (`institutional.md:248-268 (2026-04-22)`)

- Ata auto-publicada → `timeline_entries` tipo **`assembly_result`**
- Append-only imutável (Rule 2)
- Auto-update Plano Diretor via NATS `timeline.entry.created` → `UpdateMasterPlan`
- Evento NATS: `assembly.closed` → `timeline.entry.created` (`assembly.md:442`)

Histórico inclui: assembleia · edital · pauta · resultados · relatórios (linha do tempo `Content/...:330-331`).

#### 5.4 Score de Transparência (`assembly.md:498-530` + `Content/...Funcional.md:335-349`)

**Componentes código** (`entities/minutes.go:13-25`):
1. `EditalOnTime` (edital ≥5 dias antes) — 10pt
2. `QuorumReached` — 10pt
3. `AllItemsVoted` — 10pt
4. `ScienceRate >= 0.8` — 10pt
5. `MinutesPublishedIn24h` — 10pt
6. `ProxiesValidated` — 10pt
7. `PresenceDocumented` — 10pt
8. `AbsencesRegistered` — 10pt
9. `VotesHomologated` — 10pt
10. `ResultsTransparent` — 10pt

**Componentes produto** (`Content/...Funcional.md:338-347`): 10 levemente diferentes (percentual ciência, presença, votantes, leitura pauta, visualização docs, completude auditoria, regularidade homologação, cumprimento integral pauta, tempo médio por item, % itens com doc prévia).

→ **Divergência G-28**: dois conjuntos de 10 componentes. Código atual usa versão "compliance-centric"; produto usa versão "engagement-centric". Reconciliar.

Armazenamento: `assembly_minutes.transparency_score SMALLINT CHECK (0-100)`.

#### 5.5 Indicador de transparência (`assembly.md:554-565`)

Badge "Transparência: 85/100" no perfil do condomínio. Histórico gráfico (últimas 12 assembleias). Benchmark: "Acima da média"/"Média"/"Abaixo". Componente SolidJS `TransparencyBadge`.

---

## § 7. Aggregates — campos reais (código)

### 7.1 `Assembly` (aggregate root) — `entities/assembly.go:29-47`

```
id, condominiumID, title (3-200), description,
assemblyType (enum), modality (enum), status (enum), quorumType (enum),
speechTimeMinutes (2|3), scheduledDate,
editalPDFURL *string, editalPublishedAt *time.Time,
startedAt *time.Time, endedAt *time.Time,
createdBy, createdAt, updatedAt
```

Métodos mutation: `Publish()`, `Start()`, `End()`, `Cancel()`, `PublishEdital(url)`. Query: `IsEditalPublished()`, `CanStart()`.

### 7.2 `AgendaItem` — `entities/agenda_item.go:24-37`

```
id, assemblyID, title (3-200), description,
itemType (enum), status (enum), orderPosition,
fractionIdeal *string (decimal como string "15.50"),
isLocked bool, createdBy, createdAt, updatedAt
```

Métodos: `Lock()`, `OpenVoting()`, `CloseVoting(approved bool)`, `Withdraw()`, `SetFractionIdeal(str)` (parse 0-100).

### 7.3 `Vote` — `entities/vote.go:15-26` **APPEND-ONLY**

```
id, assemblyID, agendaItemID, voterID,
voteValue (enum 3), fractionIdeal *string,
isAssisted bool, termoDeVoto *string, proxyVoteID *string,
votedAt
```

Construtor valida: `isAssisted==true && termoDeVoto nil` → `ErrVoteRequiresTermoDeVoto`. **Sem setters** (`// --- Getters (sem setters — APPEND-ONLY) ---`, linha 78).

### 7.4 `Minutes` — `entities/minutes.go:65-73`

```
id, assemblyID,
content string (markdown),
transparencyScore int (0-100, CHECK DB),
isAutoGenerated bool,
publishedAt *time.Time, createdAt
```

UNIQUE `assembly_id` (`00507:5`). Método `Publish()` falha se já publicada.

### 7.5 `Proxy` — `entities/proxy.go:18-28`

```
id, assemblyID, grantorID, granteeID,
token (hex 32B crypto/rand),
status (enum 4), validUntil,
validatedAt *time.Time, createdAt
```

Métodos: `Validate()` (pendente → validada, checa expiration), `Expire()`, `IsValid()`.

### 7.6 `AttendanceRecord` — `entities/attendance_record.go:11-19`

```
id, assemblyID, userID,
presenceType (enum 3), proxyOf *string,
checkedInAt, checkOutAt *time.Time
```

Métodos: `MarkTemporaryAbsence(at)`, `MarkReturn(at)`.

### 7.7 `LiveSession` — **só existe virtualmente via ILiveProvider + roomNameFor**

```go
roomName = "assembly-" + assemblyID  // live_use_cases.go:225
```

Não há entidade persistida `LiveSession`. Estado vive em Livekit server + `assembly.status`. Observabilidade via `ListParticipants`. **Gap G-10**: não há histórico de sessões ao vivo (start/end timestamps).

### 7.8 `ScienceRecord` — `entities/science_record.go:8-14` **IMUTÁVEL**

```
id, assemblyID, userID,
editalVersion (default "v1"),
acknowledgedAt
```

Sem setters (linha 41). UNIQUE `(assembly_id, user_id)` `00506`.

---

## § 8. Enums

| Enum | Valores | Cardinalidade | Local |
|---|---|---|---|
| `AssemblyType` | `ordinaria · extraordinaria · especial` | **3** | `enums/assembly.go:7` — produto pede 4 (`ordinaria · extraordinaria · ratificacao · outra` OU `ordinaria · extraordinaria · implantação`). Divergência G-04. |
| `AssemblyStatus` | `rascunho · publicada · em_andamento · encerrada · cancelada` | **5** | `enums/assembly.go:43` |
| `AssemblyModality` | `presencial · remota · mista` | **3** | `enums/assembly.go:25` — MVP só presencial |
| `AgendaItemType` | `aprovacao_contas · eleicao_sindico · obras_melhorias · reajuste_taxa · outros` | **5** | `enums/assembly.go:82` — produto usa eixo mecânica (informativo/simples/fração/fornecedor/eleição) G-02 |
| `AgendaItemStatus` | `pendente · votando · aprovado · rejeitado · retirado` | **5** | `enums/assembly.go:103` — produto usa `resolvido/adiado/não_resolvido/prejudicado` G-03 |
| `QuorumType` | `simple_majority · qualified_majority · unanimous` | **3** | `enums/assembly.go:64` |
| `VoteType` (spec pede `VoteChoice`) | `sim · nao · abstencao` | **3** | `enums/assembly.go:124` — produto também prevê escala 1-5, não implementado G-05 |
| `VoteOrigin` (spec) | `app · presencial_assistido · procuracao` | **3** | **Não implementado explicitamente**; derivado de `is_assisted + proxy_vote_id` no `Vote` — G-06 |
| `CheckInMode` (spec) | `app · qr_code · terminal` | **3** (+manual=4) | **Não implementado como enum** — inferido por contexto — G-07 |
| `PresenceType` (AttendanceStatus) | `presente · ausente_momentaneo · por_procuracao` | **3** | `enums/assembly.go:140` |
| `SpeechDuration` | `2 · 3` min (CHECK DB `IN (2,3)`) | **2** | `00501:11` |
| `ContingencyReason` (spec) | `internet_failure · device_failure · digital_voting_unavailable` | **3** | **Não implementado** — G-08 |
| `ProxyStatus` | `pendente · validada · expirada · revogada` | **4** | `enums/assembly.go:158` |
| `HomologationStatus` (spec) | `pending · approved · rejected` | **3** | **Não implementado** — G-09 |

---

## § 9. Telas (~30) — tabela compacta

ID = `AS-##` (não há IDs estáveis em inventário; gerados aqui para catalogação). Fonte: `Inventario-Telas.md:295-304` + `Assembleia-Live-Funcional.md` + diagrama visual.

| ID | Nome | Bloco | Persona | Ações-chave |
|---|---|---|---|---|
| AS-01 | Dashboard Assembleias | 1 | Síndico/Admin | Listar por condomínio, criar nova |
| AS-02 | Criar Assembleia (wizard 3 passos) | 1-2 | Síndico | Tipo, data, convocação 1ª/2ª, modalidade, tempo fala, quórum |
| AS-03 | Cadastrar Pauta (itens) | 2 | Síndico | Add item, tipo, descrição, anexo PDF/vídeo, quórum específico, ordem |
| AS-04 | Upload Fração Ideal (planilha Excel) | 2 | Síndico/Admin | Drag Excel, validação soma=100%, exibir conflitos |
| AS-05 | Anexar/Gerar Edital | 2 | Síndico/Admin | Anexar PDF OU usar gerador (campos preenchidos), preview |
| AS-06 | Vincular Administradora | 1 | Síndico | CNPJ lookup OU convite token externo, toggles permissões |
| AS-07 | Publicar Assembleia | 2 | Síndico | Confirma lock pauta, dispara notifs email+push+SMS, gera QR |
| AS-08 | Registrar Ciência (morador) | 2 | Morador | Modal bloqueante "Li e entendi" + 5 termos (LGPD+veracidade+etc) |
| AS-09 | Confirmar Participação | 2 | Morador | Presencial/online/indefinido/não participarei |
| AS-10 | Cadastrar Procuração | 2 | Morador | Bloco+unidade+CPF representado/procurador, ciência, aviso físico |
| AS-11 | Validar Procurações (admin) | 2 | Admin/Síndico | Aprovar (replica peso) OU rejeitar com justificativa |
| AS-12 | Upload Inadimplência | 2 | Admin/Síndico | Planilha, classifica apto/inapto, corte T-1h 1ª convocação |
| AS-13 | Liberar Voto (inadimplência pós-pagamento) | 2 | Admin/Síndico | Botão "Liberar voto" + responsável + horário |
| AS-14 | Enquete Preliminar | 2 | Síndico (cria) / Morador (responde) | Sim/não, disclaimer "sem valor jurídico" |
| AS-15 | Análise Fornecedor | 2 | Morador | Perfil fornecedor + proposta (apenas durante janela) |
| AS-16 | Simulador de Quórum | 2 | Síndico/Admin | Total unidades+frações, % ciência, % presença, procurações, quórum projetado por item |
| AS-17 | Painel Inicial pré-abertura | 3 | Síndico | Cadastra presidente+secretários+admin+auditor+fornecedores |
| AS-18 | Check-in App (morador) | 3 | Morador | Clique botão OU QR scan |
| AS-19 | Check-in QR Code | 3 | Morador | Scan QR da assembleia |
| AS-20 | Check-in Terminal (kiosk) | 3 | Admin/Operador | CPF+bloco+unidade, operador presente |
| AS-21 | Painel do Presidente | 3 | Síndico | Abrir/fechar votação, fila fala, % presença tempo real, alertas quórum/tempo |
| AS-22 | Telão Área 1 — Apresentação | 3 | (display) | Slides PDF, vídeos fornecedor, comunicados tempo real |
| AS-23 | Telão Área 2 — Operacional | 3 | (display) | Pauta, quórum, fila, cronômetro, relógio, status itens |
| AS-24 | Levantar Mão (fila fala) | 3 | Morador | "Levantar mão", "abaixar mão", ver posição |
| AS-25 | Cronômetro Fala + Prorrogação | 3 | Síndico | Iniciar, +1/+2min (botão), encerrar fala |
| AS-26 | Votação Ao Vivo (morador) | 3 | Morador | Sim/Não/Abstenção (ou escala 1-5), único clique, confirmação |
| AS-27 | Voto Presencial Assistido | 3 | Admin/Síndico | Por terminal, termo de voto obrigatório, log operador |
| AS-28 | Resultado Parcial Votação | 3 | Síndico/Morador | Contagem tempo real, barra progresso |
| AS-29 | Homologar Votação | 3 | Síndico/Admin | Confirma resultado imutável; declara item resolvido/adiado/não_resolvido/prejudicado |
| AS-30 | Modo Contingência | 3 | Síndico/Admin | Ativar + selecionar razão; marca itens afetados |
| AS-31 | Encerrar Assembleia | 4 | Síndico | Validação votações fechadas, auto-gera ata |
| AS-32 | Registro Final | 4 | Síndico | Review pre-publicação (datas, presidente, votos, procurações) |
| AS-33 | Assinar Digitalmente (gov.br) | 4 | Síndico | Redireciona gov.br, recebe assinatura (Sprint 3+) |
| AS-34 | Visualizar Ata | 5 | Morador/Síndico | Leitura markdown→PDF, score transparência |
| AS-35 | Relatório Presença | 5 | Síndico | CSV/PDF, lista anonimizada |
| AS-36 | Relatório Votação | 5 | Síndico | Por item, por opção |
| AS-37 | Relatório Procurações | 5 | Síndico | Lista usadas |
| AS-38 | Relatório Trilha Auditoria | 5 | Síndico | Eventos append-only |
| AS-39 | Relatório Decisões | 5 | Síndico | Resumo executivo |
| AS-40 | Relatório Consolidado (transparência) | 5 | Síndico | Score 0-100 + componentes |
| AS-41 | Badge Transparência (perfil condo) | 5 | Público | Score + histórico 12 assembleias + benchmark |
| AS-42 | Ciência Pendentes (dashboard admin) | 2 | Síndico/Admin | Lista nominal moradores sem ciência, quem não abriu |
| AS-43 | Criar Adendo à Ata | 5 | Síndico | Retificação (Req 25), linkada à ata original |

**Total catalogado: 43 telas** (produto sinaliza ~30+, contagem aqui inclui cada forma de check-in/relatório separada).

---

## § 10. Regras (50+)

| ID | Regra | Fonte | Bloco |
|---|---|---|---|
| R-01 | 1 assembleia ativa/condomínio | `assembly.md:34` | 1 |
| R-02 | Cancelamento até 3 dias antes | `assembly.md:35` | 1 |
| R-03 | Data ≥5 dias futuro (p/ notificação) | `assembly.md:33` | 1 |
| R-04 | Pauta máx 20 itens (spec) OU ilimitado (produto) | `assembly.md:77` vs `Content/...:87` | 2 |
| R-05 | Pauta imutável após publicação (LOCK_Pauta) | `agenda_item.go:95 Lock()` + `Content/...:114-115` | 2 |
| R-06 | Único mecanismo correção = adendo (Req 25) | `assembly.md:72` + Rule 2 | 2/5 |
| R-07 | Fração ideal soma = 100% | `Content/...:172-173` | 2 |
| R-08 | Fração ideal NUMERIC(5,4) ou (6,4) nunca float | `assembly.md:71`, migration `00502:11` | 2 |
| R-09 | Ciência obrigatória → app travado | `assembly.md:143-146` + `Content/...:128-129` | 2 |
| R-10 | Tracking ciência 6 meses | `assembly.md:145` | 2 |
| R-11 | Inadimplência upload T-1h antes 1ª convocação | `Content/...:180` + `Obsidian:56 (crítico)` | 2 |
| R-12 | Liberar voto inadimplente = só síndico/admin + responsável + horário | `Content/...:185-186` | 2 |
| R-13 | Procuração cadastrável até "Iniciar assembleia" | `Content/...:155` | 2 |
| R-14 | Procuração válida até 24h pós-encerramento | `assembly.md:180` + `proxy_use_cases.go:77` | 2 |
| R-15 | Procurador replica voto exato do titular | `assembly.md:181` | 3 |
| R-16 | Outorgante ≠ outorgado | `proxy_use_cases.go:61-63` | 2 |
| R-17 | 1 procuração/outorgante/assembleia | `proxy_use_cases.go:66-69` UNIQUE | 2 |
| R-18 | Token procuração 32B hex crypto/rand | `proxy_use_cases.go:92-97` | 2 |
| R-19 | 5 termos pré-assembleia (convocação+LGPD+veracidade+identidade+gravação) | `Content/...:192-201` | 2 |
| R-20 | Check-in só em `status=em_andamento` | `attendance_use_cases.go:55-57` | 3 |
| R-21 | UNIQUE `(assembly_id, user_id)` check-in | `00503:11` | 3 |
| R-22 | Ausente momentâneo: 15 min p/ voltar, senão abstenção | `assembly.md:267-268` | 3 |
| R-23 | Voto presencial assistido exige `termo_de_voto` | `entities/vote.go:37-39 ErrVoteRequiresTermoDeVoto` | 3 |
| R-24 | Vote APPEND-ONLY (sem setters) | `entities/vote.go:78` | 3 |
| R-25 | UNIQUE `(agenda_item_id, voter_id)` — **A-025 resolvido** | `00504:14` | 3 |
| R-26 | Pre-check `HasVoted` + DB UNIQUE backstop | `vote_use_cases.go:83-89` | 3 |
| R-27 | Voto imutável pós-envio (Rule 2) | `assembly.md:277` | 3 |
| R-28 | Audit trail: timestamp+IP+device fingerprint por voto | `assembly.md:276` + Req 37 | 3 |
| R-29 | Tempo fala 2 ou 3 min (CHECK DB) | `00501:11` | 3 |
| R-30 | Prorrogação fala +2min (até 5 total) | `Content/...:84` + `Obsidian-Dominio:56` | 3 |
| R-31 | Alerta fala 30s antes | `assembly.md:352` | 3 |
| R-32 | Apenas falas microfonadas entram na ata | `Content/...:291` | 3 |
| R-33 | Corte automático mic via Livekit API (timeout) | `assembly.md:353` Req 60.5+68 | 3 |
| R-34 | Síndico não pode votar (imparcialidade) | `assembly.md:300` | 3 |
| R-35 | Telão 2 áreas (apresentação + operacional) | `assembly.md:313` Req 60.3 | 3 |
| R-36 | PowerPoint → PDF antes de upload | `Content/...:260` | 3 |
| R-37 | WebSocket latência <200ms p95 | `assembly.md:234,275,334` NFR | 3 |
| R-38 | Modo contingência manual + registra ativador | `Content/...:357-359` | 3 |
| R-39 | Ata imutável após publicação (Rule 2) | `minutes.go:114-117` + `assembly.md:585` | 4 |
| R-40 | Ata auto-publicada Timeline `assembly_result` | `assembly.md:422` + Req 11 | 4 |
| R-41 | Assinatura digital gov.br Lei 14.063/2020 | `assembly.md:431` Sprint 3+ | 4 |
| R-42 | Relatórios anonimizados (unidade+fração, sem nome) | `assembly.md:464` | 5 |
| R-43 | Visibilidade vídeo fornecedor só em janela votação | `cross-domain.md:466-477` Req 103 | 2/3/5 |
| R-44 | Auto-revogação acesso vídeo ao encerrar votação (NATS) | `cross-domain.md:475-476` | 3/5 |
| R-45 | Audit append-only retenção 5 anos | `cross-domain.md:133-142` Req 37 | trans |
| R-46 | Cada assembleia = 1 QR próprio gerado automaticamente | `Content/...:126` + `:208` | 2/3 |
| R-47 | Vídeo curto institucional Master Síndico antes da abertura | `Content/...:254-255` | 3 |
| R-48 | Escopo administradora = sandbox só módulo assembleia do condo | `Content/...:63` | 1 |
| R-49 | Conflito voto offline: último voto válido | `assembly.md:380` | 3 |
| R-50 | Dependente não vota em assembleia | `institutional.md:393` | 3 |

---

## § 11. Invariantes testáveis

```
I-01  status ∈ {rascunho, publicada, em_andamento, encerrada, cancelada}
I-02  transition(rascunho)   ⊂ {publicada, cancelada}
I-03  transition(publicada)  ⊂ {em_andamento, cancelada}  ∧ requires edital_published_at NOT NULL
I-04  transition(em_andamento) ⊂ {encerrada}
I-05  transition(encerrada) = ∅   (terminal)
I-06  transition(cancelada) = ∅   (terminal)
I-07  ∀ vote: vote.votedAt IS NOT NULL  ∧  vote imutável pós-insert
I-08  UNIQUE(agenda_item_id, voter_id)  — 1 voto/item/morador (A-025)
I-09  UNIQUE(assembly_id, user_id) em check-in
I-10  UNIQUE(assembly_id, user_id) em ciência
I-11  UNIQUE(assembly_id, grantor_id) em procuração
I-12  UNIQUE assembly_id em minutes (1 ata/assembleia)
I-13  speech_time_minutes IN (2, 3)   (CHECK DB)
I-14  transparency_score BETWEEN 0 AND 100   (CHECK DB)
I-15  fraction_ideal NUMERIC(6,4), range 0-100 (entidade valida)
I-16  v.isAssisted = true ⇒ v.termoDeVoto ≠ NULL  (entidade enforces)
I-17  proxy.grantorID ≠ proxy.granteeID
I-18  proxy.validUntil = assembly.scheduledDate + 24h
I-19  minutes.publishedAt NULL → pode publicar; NOT NULL → 2ª Publish() falha
I-20  agendaItem.isLocked = true ⇒ OpenVoting() retorna ErrAgendaItemLocked
I-21  assembly.modality = presencial ⇒ LiveSession não é iniciada (live_use_cases.go:53-55)
I-22  LiveSession requer assembly.status = em_andamento (live_use_cases.go:56-58)
I-23  ciência requer assembly.IsEditalPublished() = true
I-24  publicação pauta NÃO dispara start de assembly (status fica publicada até Start() explícito)
I-25  assembly.End() dispara criação Minutes automaticamente (não-opcional)
```

---

## § 12. State Machines

### 12.1 Assembly

```
                 ┌──────────┐
                 │ rascunho │
                 └────┬─────┘
                      │ Publish()
           ┌──────────┼────────────┐
           │          ▼            │
           │   ┌────────────┐      │ Cancel()
           │   │ publicada  │      │
           │   └──────┬─────┘      │
           │          │ Start()    │
           │          │  (requires │
           │          │   edital)  │
           │          ▼            ▼
           │   ┌─────────────┐  ┌────────────┐
           │   │em_andamento │  │ cancelada  │ (terminal)
           │   └──────┬──────┘  └────────────┘
           │          │ End()
           │          ▼
           │   ┌─────────────┐
           └──►│  encerrada  │ (terminal, gera Minutes)
               └─────────────┘
```

### 12.2 LiveSession (virtual — sem entidade persistida)

```
not-started → CreateRoom(livekit) → active → EndRoom(livekit) → ended
                  │
                  └─ Saga compensation: se GenerateToken falha, EndRoom (live_use_cases.go:70-78)
```

### 12.3 AgendaItem

```
pendente ─OpenVoting()──► votando ─CloseVoting(approved)──► aprovado
    │                         │                  └─(!approved)─► rejeitado
    │                         │
    └─Withdraw()──► retirado  │
                              └─(requires !isLocked)
```

### 12.4 Vote

```
Vote é APPEND-ONLY. Uma vez criado, imutável.
Não tem states; é simples INSERT.
```

### 12.5 Minutes

```
draft (created via NewMinutes, publishedAt=NULL)
   │
   │ Publish()
   ▼
published (publishedAt NOT NULL, imutável — 2ª Publish retorna erro)
```

### 12.6 Proxy

```
pendente ─Validate()─► validada ─(used in vote)─► (logicamente consumida)
   │                      │
   │ Expire()/now>valid   └─ Expire()/now>valid ─► expirada
   │
   └─ revoked by grantor ─► revogada
```

---

## § 13. Domain Events

**Emissão via NATS** (canvas `Obsidian/System Architecture - Assembly.md:232-262`):

### Pré-Assembleia
- `assembly.config.created`
- `assembly.config.agenda-item-added`
- `assembly.config.published` (edital)
- `assembly.config.ciencia-confirmed`
- `assembly.config.proxy-registered`
- `assembly.config.proxy-validated`
- `assembly.config.fraction-ideal-uploaded`

### Live-Day
- `assembly.live.started`
- `assembly.checkin.morador-checked-in`
- `assembly.checkin.morador-temporary-absence`
- `assembly.checkin.morador-returned`
- `assembly.live.item-opened`
- `assembly.live.item-closed`
- `assembly.vote.cast`
- `assembly.vote.homologated`
- `assembly.speech.started`
- `assembly.speech.ended`
- `assembly.speech.extended`
- `assembly.live.contingency-activated`
- `assembly.live.contingency-resolved`
- `assembly.live.paused`
- `assembly.live.resumed`
- `assembly.voting.closed` → dispara `video.visibility.revoked` (Req 103)

### Pós-Assembleia
- `assembly.live.finalized` → dispara `timeline.entry.created` tipo `assembly_result`
- `assembly.config.report-generated`
- `assembly.config.transparency-calculated`
- `assembly.config.ata-published`
- `assembly.closed` → `timeline.entry.created` → `master_plan.updated` (cadeia Req 12)

---

## § 14. WebSocket real-time (eventos + payloads)

**Namespace**: `/assembly-live/:id` (`Obsidian/Dominio-Assembleia.md:150`).

**Rooms no Fastify**: `assembly:{id}` subdivido em:
- `assembly:{id}:telao` — presença, quórum, item atual, fila
- `assembly:{id}:voting` — contagem votos, status
- `assembly:{id}:speech` — fila fala, timer
- `assembly:{id}:checkin` — presença real-time
- `assembly:{id}:control` — ações presidente

**Fluxo** (`System Architecture - Assembly.md:119-125`):
1. Ação ocorre (voto, check-in, fala)
2. Persiste PG + outbox em TX atômica
3. Outbox publisher → NATS `assembly.live.*`
4. Consumer `assembly-broadcaster` → Redis pub/sub
5. Fastify instances → push WS clients

### Eventos emitidos pelo morador
```json
emit("morador_checkin",     { moradorId, unit, block, timestamp })
emit("morador_raise_hand",  { moradorId, timestamp })
emit("morador_cast_vote",   { moradorId, itemId, payload })
```

### Broadcasts da mesa/presidente
```json
broadcast("sync_state",              { currentAgendaItem, quorumPresent, connectedUnits })
broadcast("speaker_timer_start",     { moradorId, durationSecs })
broadcast("speaker_timer_tick",      { remainingSecs })
broadcast("item_voting_opened",      { itemId })
broadcast("item_voting_closed",      { itemId })
broadcast("item_voting_homologated", { itemId, results })
broadcast("contingency_activated",   { reason, time })
broadcast("attendance.updated",      { totalPresent, totalAbsent, quorumPct })
```

**Latência-alvo**: <200ms p95 (NFR crítico R-37).

**Redis auto-save**: chave `assembly:{id}:offline_votes:{user_id}` p/ contingência.

---

## § 15. Livekit integration

### Contrato `ILiveProvider` (`providers/i_live_provider.go:15-28`)

```go
type ILiveProvider interface {
    CreateRoom(ctx, roomName) (*LiveRoom, error)
    GenerateToken(ctx, roomName, participantID, displayName, canPublish) (string, error)
    EndRoom(ctx, roomName) error
    ListParticipants(ctx, roomName) ([]string, error)
}
```

**Swap de provider**: implementar interface + atualizar wiring em `assembly/infrastructure/providers/`. Agnosticismo confirmado.

### Implementação `LivekitProvider` (`livekit_provider.go`)

**Config via env**: `LIVE_HOST`, `LIVE_API_KEY`, `LIVE_API_SECRET`.

**CreateRoom**: `EmptyTimeout=300s` (5 min sem participantes → auto-encerra), `MaxParticipants=500`.

**GenerateToken**: pura op local (JWT sign, sem I/O). Grant `RoomJoin + CanPublish ± CanSubscribe`. TTL **2 horas**.

**EndRoom**: `DeleteRoom`; `NotFound` é terminal → retorna nil (reconciliação implícita).

**ListParticipants**: `NotFound` retorna `[]` (joiner antes do moderador é legítimo).

### Retry (A-033/A-034 resolvidos)

`backoffs = [100ms, 300ms, 900ms]` (`livekit_provider.go:31-35`). Exponencial simples sem jitter (requests de assembleia agendados, poucas simultâneas, risco baixo thundering herd).

`retryLivekit[T]` função genérica (linhas 40-64):
- 4 tentativas total (1 imediata + 3 com delays)
- Ctx cancelada aborta
- `permanentErr` callback decide terminal (NotFound, validation, auth) — não retry

### Saga compensação (A-028 resolvido) (`live_use_cases.go:66-80`)

Se `CreateRoom` OK mas `GenerateToken` falha:
1. Sala órfã no Livekit
2. Compensa chamando `EndRoom`
3. Falha de compensação: logada (não esconde erro original do usuário; reconciliação background)

**Código cita**:
> "Saga: se gerar token falhar, a sala Livekit fica órfã. Compensar encerrando a sala. Falha de compensação é logada (não esconde o erro original do usuário; reconciliação é job de background)."

### Endpoints HTTP (`module.go:234-249`)

| Método | Path | Permissão |
|---|---|---|
| POST | `/condominiums/:condo/assemblies/:id/live` | `assembly.live.start` |
| POST | `/condominiums/:condo/assemblies/:id/live/join` | `assembly.live.join` |
| DELETE | `/condominiums/:condo/assemblies/:id/live` | `assembly.live.start` |
| GET | `/condominiums/:condo/assemblies/:id/live/participants` | `assembly.live.join` |

**Restrições**:
- `Modality = presencial` → `ErrValidation` "assembleia presencial não usa sessão ao vivo" (`live_use_cases.go:53-55`). MVP presencial = Livekit não é chamado.
- `Status ≠ em_andamento` → erro.

### Lives institucionais ≠ Assembleia Live (D-097 + D-133)

Lives institucionais (academy/LMS) são criadas por **admin + Empresa Pro + agência delegada** (D-097 Fase 11 revoga D-062/D-063/D-076 admin-only do legado). Aqui (assembleia-live) a sala Livekit é criada a pedido do síndico para a assembleia (conferência tipo Meet, não broadcast) — **não é admin-only**, é síndico-only + admin. ABAC: `assembly.live.start` é role síndico/admin. **D-133 separa**: Lives institucionais usam `BroadcastLive` aggregate (Mux Live ou Cloudflare Stream — ADR-0044 pending); Assembleia usa Livekit conferência.

---

## § 16. Assinatura digital gov.br

**Base legal**: Lei 14.063/2020 (níveis simples/avançada/qualificada).

**Escopo Master Síndico**: assinatura avançada via integração gov.br (nível exigido para ata de assembleia condominial pelo STJ REsp 1.831.718/SP — precedente).

**Timing**: Sprint 3+ (não-MVP).

**Fluxo**:
1. Ata gerada (markdown → PDF)
2. Síndico clica "Assinar digitalmente"
3. Redireciona gov.br OAuth
4. Recebe PDF assinado
5. Armazena MinIO com hash (campo `minutes.signed_pdf_url`, não implementado)

**Também aplicável**: edital (`assembly.md:128`), procuração (Sprint 3+ `assembly.md:181`), termos de voto assistido (`assembly.md:261`).

---

## § 17. Eleição gera avaliação (D-063)

Cita `Content/...Funcional.md:110-112 (2026-04-22)` + matriz `institutional.md:370-381 (Req 15)`:

**Regra**: ao publicar item tipo `eleicao_sindico` (código) / `eleicao` (produto):
- Sistema cria `GovernanceAssessment` (Req 15) associado à assembleia
- 3 perguntas fixas escala 1-10 (anônimas p/ síndico)
- **Escondida** até morador dar ciência do edital
- **Sem valor jurídico** (disclaimer)
- D-063 prevê **toggle para desabilitar** (síndico pode optar por não gerar avaliação)

**Integração**:
- Evento NATS: `assembly.agenda.eleicao-added` → `institutional.assessment.created`
- Bloqueio: se assessment atrasado (>2 meses — Req 15), bloqueia certas ações do síndico (export relatórios, criar atividade)

**Impact**: fortalece governance score (Req 33 componente 9 `avaliação de gestão`).

**Aviso produto**: essa regra NÃO está explícita em `requirements/assembly.md` atual (Q40 `quebras-detectadas`).

---

## § 18. Estado no código

### F-### / A-### resolvidos (Fase 7d/8)

| ID | Descrição | Status | Evidência |
|---|---|---|---|
| **A-025** | Vote UNIQUE DB concorrência | ✅ Resolvido | `00504:14 UNIQUE(agenda_item_id, voter_id)` + `HasVoted` pre-check `vote_use_cases.go:83-89` |
| **A-028** | Saga Livekit compensação (sala órfã se token falha) | ✅ Resolvido | `live_use_cases.go:66-80` EndRoom compensação + log |
| **A-033** | ListParticipants antes do moderador (NotFound) | ✅ Resolvido | `livekit_provider.go:169-186` retry + `isNotFound` terminal retorna `[]` |
| **A-034** | EndRoom de sala já encerrada (idempotência) | ✅ Resolvido | `livekit_provider.go:155-167` retry + NotFound retorna nil |
| **R-stall** | Fase 8 = retry após stall | ✅ Este doc | Catalogação completa pós-stall |

### Módulo implementado

| Componente | Status | Arquivo |
|---|---|---|
| 7 entidades | ✅ | `entities/*.go` |
| 9 enums | ✅ | `enums/assembly.go` |
| `ILiveProvider` + Livekit impl | ✅ | `providers/` |
| Repos 7 | ✅ | `infrastructure/database/*.go` |
| Use cases 24+ | ✅ | `application/use_cases/*.go` |
| Handlers 8 | ✅ | `infrastructure/http/routes/*.go` |
| 8 migrations | ✅ | `migrations/00500-00507` |
| Testes | ✅ | `*_test.go` (incl PBT `agenda_item_pbt_test.go`) |
| Módulo wired | ✅ | `module.go:34-122` |
| ABAC permissions | ✅ | 14 permissions `assembly.{resource}.{action}` |

### Endpoints HTTP (26 rotas, `module.go:128-250`)

```
POST   /condominiums/:condo/assemblies
GET    /condominiums/:condo/assemblies
GET    /condominiums/:condo/assemblies/:id
POST   /condominiums/:condo/assemblies/:id/publish
POST   /condominiums/:condo/assemblies/:id/publish-edital
POST   /condominiums/:condo/assemblies/:id/start
POST   /condominiums/:condo/assemblies/:id/end
POST   /condominiums/:condo/assemblies/:id/agenda-items
POST   /condominiums/:condo/assemblies/:id/agenda-items/:item/open-voting
POST   /condominiums/:condo/assemblies/:id/agenda-items/:item/close-voting
POST   /condominiums/:condo/assemblies/:id/agenda-items/:item/withdraw
POST   /condominiums/:condo/assemblies/:id/agenda-items/:item/votes
GET    /condominiums/:condo/assemblies/:id/agenda-items/:item/votes
POST   /condominiums/:condo/assemblies/:id/attendance
POST   /condominiums/:condo/assemblies/:id/attendance/absence
POST   /condominiums/:condo/assemblies/:id/attendance/return
POST   /condominiums/:condo/assemblies/:id/proxies
GET    /condominiums/:condo/assemblies/:id/proxies
POST   /condominiums/:condo/assemblies/:id/proxies/validate
POST   /condominiums/:condo/assemblies/:id/science
GET    /condominiums/:condo/assemblies/:id/science
GET    /condominiums/:condo/assemblies/:id/minutes
POST   /condominiums/:condo/assemblies/:id/live
POST   /condominiums/:condo/assemblies/:id/live/join
DELETE /condominiums/:condo/assemblies/:id/live
GET    /condominiums/:condo/assemblies/:id/live/participants
```

### ABAC permissions requeridas

```
assembly.assembly.{create, read, manage}
assembly.agenda_item.manage
assembly.vote.{cast, read}
assembly.attendance.checkin
assembly.proxy.{create, read, validate}
assembly.science.{acknowledge, read}
assembly.minutes.read
assembly.live.{start, join}
```

---

## § 19. Gaps

| ID | Gap | Impacto | Prioridade |
|---|---|---|---|
| **G-01** | `Unit` sem `fracao_ideal` (vive em `agenda_items.fraction_ideal` + passada no voto) | **CRÍTICO** — votação com peso é frágil; morador pode mentir fração | **P0** |
| **G-02** | Enum `AgendaItemType` por tema (contas/obras) vs produto por mecânica (informativo/simples/fração/fornecedor/eleição) | Taxonomia errada | P0 |
| **G-03** | `AgendaItemStatus` código (aprovado/rejeitado/retirado) ≠ produto (resolvido/adiado/não_resolvido/prejudicado) | Falta `adiado`; mapear | P1 |
| **G-04** | `AssemblyType` tem 3 valores vs 4-5 produto (sem `ratificacao` nem `implantação`) | Taxonomia incompleta | P1 |
| **G-05** | `VoteType` só 3 valores; produto pede escala 1-5 p/ votação escalada | Funcionalidade faltando | P1 |
| **G-06** | Sem enum `VoteOrigin` explícito (app/presencial_assistido/procuração derivados de flags) | Audit/relatório menos granular | P2 |
| **G-07** | Sem enum `CheckInMode` (app/qr/terminal/manual) — método não persistido | Relatório presença não sabe método | P2 |
| **G-08** | Sem enum `ContingencyReason` nem campo `contingency_active` em `assemblies` | Não rastreável | P1 |
| **G-09** | Sem `HomologationStatus` nem tabela `assembly_homologations` | `End()` gera Minutes direto; sem homologação separada | P1 |
| **G-10** | Nenhuma entidade `LiveSession` persistida (estado só em Livekit server) | Sem histórico start/end ao vivo | P2 |
| **G-11** | Sem tabelas `assembly_speech_queue` / `assembly_speeches` | Fila de fala não implementada | **P0** (MVP depende) |
| **G-12** | Sem tabela `assembly_legal_acceptances` (5 termos pré-assembleia) | LGPD risco | P0 |
| **G-13** | Sem tabelas `assembly_preliminary_surveys` (enquetes preliminares) | Bloco 2 incompleto | P1 |
| **G-14** | Sem tabela `assembly_fractions` (upload planilha fração ideal) — relacionado G-01 | Planilha não persistida | P0 |
| **G-15** | Sem tabela `assembly_notifications` nem scheduler lembretes (T-3d, T-1d, T-3h) | Notificações faltam | P0 |
| **G-16** | Sem gerador automático de edital (template HTML→PDF via wkhtmltopdf/Puppeteer) | Só aceita anexo; template faltando | P1 |
| **G-17** | Sem integração Timeline para auto-publicação `assembly_result` | Rule 2 não efetivada | P0 |
| **G-18** | Sem mapeamento `VoteType escala_1_a_5` | Produto esperado; código só 3 valores | P1 |
| **G-19** | Sem cálculo ponderado por fração ideal no endpoint `GetVoteResults` (soma simples por opção) | Resultado incorreto para votação com peso | **P0** |
| **G-20** | Sem endpoint nem lógica `Simulador de Quórum` | Bloco 2.12 falta | P1 |
| **G-21** | Sem fluxo `Liberar Voto` (inadimplência pós-pagamento) | Permissão especial falta | P1 |
| **G-22** | Sem vinculação administradora por token externo (onboarding 6 passos) | Bloco 1 produto falta | P1 |
| **G-23** | Sem assinatura digital gov.br (Sprint 3+) | Validade jurídica reduzida | P2 (planejado) |
| **G-24** | Sem geração PDF ata (só markdown em `minutes.content`) | Export faltando | P1 |
| **G-25** | Sem telão (componentes SolidJS `PresidentScreen`, `OperationalPanel`) | Frontend ausente | P1 |
| **G-26** | Sem WebSocket handlers (`infrastructure/ws/` listado em `module.go` está vazio) | Real-time faltando | **P0** |
| **G-27** | Sem outbox pattern implementado para eventos `assembly.*` | Eventos não publicados | P0 |
| **G-28** | Sem score de transparência por componentes produto (engagement) vs código (compliance) — 2 versões divergentes | Reconciliar ambos | P1 |
| **G-29** | `ScienceRate` hardcoded 0.8 se `scienceCount>0` (proxy por falta de total de moradores) (`assembly_use_cases.go:290-293`) | Score transparência impreciso | P1 |
| **G-30** | `QuorumReached = true` hardcoded quando encerra (`assembly_use_cases.go:299`) | Score transparência incorreto | P1 |
| **G-31** | Sem regra "1h antes da 1ª convocação → bloqueio upload inadimplência" enforcement | Compliance regra forte falta | P1 |
| **G-32** | Precisão fração ideal divergente: spec `NUMERIC(5,4)` vs migration `NUMERIC(6,4)` | Documentação | P3 |
| **G-33** | Enquetes preliminares: produto só sim/não, spec tem 3 tipos (sim/não/não sei + múltipla + escala) | Taxonomia | P3 |
| **G-34** | Procurador externo vs mesmo condomínio — spec obriga mesmo condo, produto permite externo | Regra de negócio | P2 |
| **G-35** | Votação 15 min p/ voltar de ausência momentânea NÃO implementada como timer automático | Abstenção manual | P1 |

---

## § 19.5 Telas & Endpoints

Rastreabilidade consolidada em [[../../03-subprojects/traceability]] §2.5 (Assembleia Live) + §3.4 (matriz inversa).

**Escopo UI**:
- Web: 21 telas (ASM-*) distribuídas em 5 blocos
- Mobile: 5 telas (ASM-CHKIN · ASM-CIEN · ASM-FILA-FALA · ASM-PROC-CAD · ASM-VOTO)

**Endpoints-chave** (`/api/v1/assemblies/*`):
- Criação: POST `/assemblies` · PATCH `/:id` · GET `/:id`
- Pauta: `/:id/agenda` · `/reorder` · `/:item_id`
- Edital: `/:id/edital/upload` · `/generate` · PUT
- Publicação (step-up X-Step-Up-Token): `/:id/publish` · `/notify`
- Pré-assembleia: `/:id/acknowledge` · `/terms/accept` · `/proxies` · `/proxies/:id/validate` · `/polls` · `/quorum/simulate`
- Fração: `/condominiums/:id/units/fractions/bulk` (POST mp)
- Check-in: `/:id/checkin` · `/checkin/assisted` · `/attendance`
- Votação: `/:id/agenda/:item/vote` · `/votes`
- Homologação (step-up): `/:id/agenda/:item/homologate`
- Encerramento (step-up): `/:id/close` · `/close/preflight`
- Relatórios: `/:id/reports` · `/:id/transparency-score`
- WebSocket telão: `wss://assemblies/:id/screen`
- Webhook Livekit: `/api/v1/webhooks/livekit` ⚫

**Invariantes observáveis em UI** (21 INVs): INV-021 · INV-022 · INV-023 (fração + NUMERIC) · INV-071 (voto único) · INV-072 (ata imutável) · INV-073 (quórum fracional) · INV-074 (sem edital = sem início) · INV-075 (≤1 ativa) · INV-076 (timeout=Absent) · INV-077 (proxy=titular) · INV-078/079 (proxy validade/revogação) · INV-080 (pauta read-only pós publish) · INV-081 (síndico não vota) · INV-082 (fração NUMERIC) · INV-083 (snapshot fração no voto) · INV-084 (saga Livekit) · INV-085 (homologação separada) · INV-110 (step-up MFA) · INV-118 (edital ≥8d Lei 4.591/64) · INV-119 (DB-level immutability ata).

**Fluxo crítico** (traceability §7.4): DASH → CFG → PAUTA → EDITAL → PUB (step-up) → CIEN → TERM → PROC-CAD/VAL → FRAC → INAD → SIM → CHKIN → PAINEL-SIND (step-up) → TELAO (WS) → VOTO (INV-071/073/076/083) → FILA-FALA → HOML (step-up) → ENCER (step-up) → REL.

**Sprints**: Sprint 7 ✅ backend completo (assembly + Livekit + saga) · UI Sprint 10 (M1 core — 5 mobile bloqueadores) 🟡.

**Gaps**: Híbrida/online modalidade preparada mas inativa em M1 · Transcrição automática backlog M2/M3 · Assinatura ICP-Brasil backlog M3+.

---

## § 20. Referências cruzadas

### Dentro do sub-produto
- `§6 Blocos` ↔ `§9 Telas` (ID → bloco)
- `§8 Enums` ↔ `§12 State machines`
- `§18 Estado código` ↔ `§19 Gaps`

### Sub-produtos v2 irmãos
- [[01-governanca-institucional]] — Timeline (R-40 `assembly_result`), Mandato, Governance Score (Req 33 componente 7 `assembleias regularizadas`)
- [[02-connect-me]] — Req 103 visibilidade vídeo fornecedor na votação escolha_fornecedor
- [[03-reputacao]] — Avaliação gestão (Req 15) gerada por eleição (D-063)
- [[04-conteudo-videos]] — Vídeo fornecedor unlock durante votação + vídeo institucional pré-abertura R-47
- [[06-lms-academia]] — Lives institucionais (D-097 admin + Empresa Pro + agência delegada — revoga D-062/D-063/D-076) — contexto distinto desta live (D-133 broadcast vs conferência)
- [[09-compliance]] — Audit trail append-only Req 37, Declaração anual Req 34, Score governança Req 33, Dossiê Req 40, LGPD registry Req 39
- [[11-administradoras-condominiais]] — vínculo por token externo, permissões toggles

### Specs canônicos backend
- `backend/.kiro/specs/master-sindico/requirements/assembly.md` — Req 48-60.12
- `backend/.kiro/specs/master-sindico/requirements/cross-domain.md` — Req 37 audit, Req 68 Livekit, Req 103 visibilidade
- `backend/.kiro/specs/master-sindico/requirements/institutional.md` — Req 11 Timeline, Req 12 Plano Diretor, Req 15 avaliação
- `backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` — paywalls voto+criação
- `backend/.kiro/specs/master-sindico/requirements/commercial.md` — Req 25 adendo (único mecanismo correção)

### Canvas Obsidian
- `01 - Arquitetura de Sistema/System Architecture - Assembly.md` — infra WS+NATS+Redis+outbox
- `02 - Domínios e Requisitos/Domínio - Assembleia.md` — v3.0 endpoints + eventos
- `Arquitetura Assembleia.canvas` — visão 2D
- `Fluxo Assembleia Ao Vivo.canvas` — fluxograma

### Decisions
- **D-056** Aditivo contratual: +1 sprint Live Assembly (9-10 sprints total)
- **D-062** Lives admin-only MVP (aplicável a LMS; aqui contexto diferente — assembleia é síndico+admin)
- **D-063** Eleição gera avaliação (pode desabilitar)
- **Decision 1** (legacy) — RESOLVED para Live Assembly (Option B) com WebSocket real-time (`System Architecture - Assembly.md:30-32`)

### Regras canônicas
- Rule 2 — Pós-publicação imutável, modificação só via adendo
- Rule 4 — Vídeos empresa blindados exceto janela votação
- Rule 8 — 1 dispositivo ativo/conta (aplicável ao login p/ votação)
- Rule 10 — LGPD rastreabilidade (termos pré-assembleia)

---

_Destilação completa sub-produto 5 — Fase 8 retry após stall. Dual-check pendente._
