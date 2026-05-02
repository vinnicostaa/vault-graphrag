---
title: Mobile — UI Catalog
type: spec
tags: [subproject, mobile, master-sindico, v2, ui-catalog, telas, flutter]
source:
  - ../web/ui-catalog.md (jornadas completas)
  - Clientes/Joao/MasterSindico/Development/app/lib/features/** (2026-04-21)
  - 00-product/personas.md
created: 2026-04-23
updated: 2026-04-23
aliases: [mobile-ui-catalog, app-ui-catalog]
---

# Mobile — UI Catalog

Catálogo de telas **mobile** do Master Síndico. Subconjunto das 141+ telas web, adaptadas para mobile em 3 ondas de entrega:

- **M1** (2026-05-08): morador completo + base de conta + sistema.
- **M2** (2026-06-20): síndico light (voice-first) + Connect Me morador + vídeo morador + votação fornecedor.
- **M3** (2026-07-20): assembleia live (procuração, check-in, vote, live session) + LMS thin + store submission.

> Personas mobile: **Morador** (principal) + **Síndico** (paridade parcial). Empresa, Parceiro, Agência de Marketing = **web only**. Admin (D-058) = role privilegiada **dentro** da mesma app — nunca app separada; telas gated aparecem conforme ABAC.

Referências: [[../web/ui-catalog]] (source de jornadas); tokens visuais espelhados de [[../web/design-system]] (com ressalva de reconciliação da paleta — ver [[architecture#122-alvo-reconciliar-com-manual-iv-ouronavy]]).

---

## Índice

- [§1 Sistema (bootstrap, splash, device, admin-gate)](#1-sistema)
- [§2 Auth](#2-auth)
- [§3 Home shell](#3-home-shell)
- [§4 Morador](#4-morador)
- [§5 Síndico (mobile light)](#5-síndico-mobile-light)
- [§6 Assembleia Live](#6-assembleia-live)
- [§7 Connect Me (morador)](#7-connect-me-morador)
- [§8 Vídeo (morador)](#8-vídeo-morador)
- [§9 Perfil, conta, assinatura](#9-perfil-conta-assinatura)
- [§10 Notificações](#10-notificações)
- [§11 Exceções / offline](#11-exceções-offline)
- [§12 Admin role gated (D-058)](#12-admin-role-gated-d-058)
- [§13 Paridade com web — matriz](#13-paridade-com-web--matriz)
- [§14 Pendências dual-check](#14-pendências-dual-check)

---

## 1. Sistema

### APP-SYS-001 — Splash

- **Rota**: `/` (initialLocation do GoRouter atual `AppRoutes.home`; alvo M1: splash dedicada `/splash`).
- Exibição institucional: logo + "Master Síndico" + subtítulo.
- Bootstrap: `WidgetsFlutterBinding.ensureInitialized()` → `SharedPreferences.getInstance()` → `configureDependencies()` → checa token em `flutter_secure_storage`; se válido → redirect `/home`; se inválido → `/login`.
- Timeout 1.5s — se bootstrap mais rápido, exibe ≥ 800ms pra evitar flash.
- **Estados**: loading (sempre); error (retry + "Sem conexão").

### APP-SYS-002 — Onboarding first-open (opcional)

- **Rota**: `/welcome`.
- 3 slides swipable: "Governança estruturada", "Timeline imutável", "Vote com sua fração ideal".
- **Ação**: [Começar] → `/login`.
- **Estado**: mostrado apenas no primeiro open (flag `hasSeenOnboarding` em SharedPrefs).

### APP-SYS-003 — Biometria prompt (opcional)

- Modal apresentado em re-open pós-logout curto (< 30min).
- Ações: [Usar biometria]; [Usar senha].
- Se biometria habilitada + `local_auth` success → usa refresh token secure_storage sem re-OIDC.

### APP-SYS-004 — Device mismatch

- **Rota**: `/device-mismatch`.
- Aparece quando backend retorna `403 NEW_DEVICE_DETECTED` (IR-011).
- Mensagem: "Sessão foi transferida para outro dispositivo. Por segurança, você foi desconectado aqui."
- Ação: [Voltar ao login].

### APP-SYS-005 — App outdated (force update)

- Modal bloqueante quando backend retorna `426 APP_VERSION_OUTDATED` em feature crítica.
- Ação: [Atualizar agora] → abre store; [Mais tarde] (se soft-deprecation).

### APP-SYS-006 — Integrity blocked (D-050 M3)

- Se `X-Device-Integrity = jailbroken | hooked` em feature crítica (votação, assembleia, assinatura): modal "Dispositivo não seguro — use outro para esta operação."
- M1 = report-only (sem modal); M3 = bloqueio real.

---

## 2. Auth

### APP-AUTH-001 — Login

- **Rota**: `/login`.
- **Implementado hoje**: `LoginPage` (StatelessWidget) + `LoginForm` com botão "[Entrar com Zitadel]" (mudar label → "[Entrar]") que dispara `AuthBloc.add(AuthLoginRequested())`.
- **Fluxo real**: Bloc chama `LoginUseCase` → `AuthRepositoryImpl.login()` → `AuthRemoteDatasourceImpl.login()` que cria `OidcUserManager.lazy` e dispara `loginAuthorizationCodeFlow()`:
  - iOS: `ASWebAuthenticationSession`.
  - Android: Custom Tabs.
- Zitadel autentica → callback custom scheme `com.mastersindico:/auth` → app troca code por tokens → persist em `flutter_secure_storage` + snapshot `UserModel` em `SharedPreferences` chave `CACHED_USER`.
- **Estados** (reais): `AuthInitial`, `AuthLoading` (spinner no botão), `AuthAuthenticated(user)` (navega `/home`), `AuthFailureState(message)` (snackbar vermelho).
- **Rodapé** (alvo): Termos + Política de Privacidade.

### APP-AUTH-002 — Logout confirm

- Modal: "Deseja mesmo sair?"
- Ação: [Sair] → `AuthBloc.add(AuthLogoutRequested())` → `LogoutUseCase` → `AuthRepositoryImpl.logout()` chama `manager.logout()` Zitadel + clear secure_storage + prefs + navega `/login`.

---

## 3. Home shell

### APP-HOME-001 — Shell com bottom nav (M1 alvo)

- **Rota wrapper**: `/home` (ShellRoute GoRouter).
- Bottom nav 4 abas:
  - **Início** (home feed + atalhos) → `/home`
  - **Timeline** (linha do tempo do condomínio ativo) → `/home/timeline`
  - **Ações** (Plano Diretor, Eventos, Votações, Connect Me) → `/home/acoes`
  - **Conta** (perfil, assinatura, configurações) → `/account`

Para síndico (role síndico na `membership` ativa): bottom nav **adicional** "Governança" → `/sindico`.

### APP-HOME-002 — Selector de condomínio (bottom sheet top)

- Tap no topo "Condomínio X" abre bottom sheet com lista de memberships ativas.
- Ações: [Trocar]; [Adicionar condomínio] → `/condominium/add`.
- Cada card: nome, endereço, unidade, papel (Titular/Dependente), status.

---

## 4. Morador

### APP-MOR-001 — Painel morador (Início)

- **Rota**: `/home`.
- Topo: nome condomínio + trocar.
- Cards: Próximo evento; Linha do tempo recente (3); Votação em andamento; Plano Diretor resumo; Comunicados não lidos.
- **Estados**: loading skeleton; empty "Bem-vindo!"; error.
- **Implementado hoje**: apenas stub `home_page.dart` com "Olá, Template!" e `logout` no AppBar — M1 substitui.

### APP-MOR-002 — Timeline (lista)

- **Rota**: `/home/timeline`.
- ListView infinito com cards (ver [[../web/design-system]] §8.15 Timeline entry card espelhado).
- Filtros (bottom sheet): período, natureza, risco.
- Pull-to-refresh (`RefreshIndicator` — padrão já usado em `AssemblyListPage`).
- Tap no card → APP-MOR-003.

### APP-MOR-003 — Timeline detalhe

- **Rota**: `/home/timeline/:entryId`.
- Conteúdo: tipo, título, data, importância, descrição, fotos, vídeos (player), anexos, impacto esperado (se autorizado), próxima ação (se autorizado).
- **Estados**: loading, error, permission-denied.

### APP-MOR-004 — Plano Diretor (leitura)

- **Rota**: `/home/plano-diretor`.
- Lista por status/natureza/risco.
- Por item: título, status badge, prazo (se autorizado).
- Tap → detalhe (subset campos).

### APP-MOR-005 — Eventos

- **Rota**: `/home/eventos`.
- Abas: Próximos | Em andamento | Encerrados.
- Card por evento: tipo, título, data/hora, local, taxa visualização.
- Tap → APP-MOR-006.

### APP-MOR-006 — Evento detalhe + ciência

- **Rota**: `/home/eventos/:id`.
- Detalhes: descrição, data início/fim, local, anexos.
- Ações: [Ciente] (registra ciência); [Confirmo participação] (até prazo).
- Feedback pós-ação: "Participação confirmada. Pode alterar até [data]".

### APP-MOR-007 — Votação de Fornecedor (M2)

- **Rota**: `/home/votacao/:sessionId`.
- Título: "Escolha da proposta".
- Lista cards fornecedor: nome + badge verified + resumo técnico + valor + condições pagamento + prazos + garantia.
- Ações: [Ver proposta completa] (modal bottom sheet); [Ver vídeo] (APP-VID-003 com visibility grant temporário); [Votar].
- Pós-voto: "Seu voto foi registrado" (com `bloc_concurrency.droppable()` — evita double-tap).
- **Estados**: sessão aberta (votar); encerrada (read-only resultado); proxy outorgado (badge).

### APP-MOR-008 — Votação: ver proposta completa (modal)

- Bottom sheet full-height.
- Exibe: descrição técnica, escopo, valor, condições pagamento, prazos, garantia, responsável técnico, anexos.
- Ações: [Fechar]; [Ver vídeo] (se autorizado).

### APP-MOR-009 — Avaliação obrigatória da gestão (gate)

- **Rota**: `/home/avaliacao-obrigatoria`.
- 3 critérios 1-10: Comunicação / Organização / Confiança.
- Regra: **bloqueia acesso a `/home`** até envio (redirect via `GoRouter.redirect`).

### APP-MOR-010 — Gerenciar acessos (account → acessos)

- **Rota**: `/account/acessos`.
- Lista unidades vinculadas.
- Ações por item: atualizar status; encerrar acesso; editar vínculo.

### APP-MOR-011 — Adicionar condomínio — buscar por ID

- **Rota**: `/condominium/add`.
- Campo: ID 8-char.
- Após inserção: card com nome + endereço + CEP.
- [Confirmar] → APP-MOR-012.

### APP-MOR-012 — Identificação da unidade (wizard)

- **Rota**: `/condominium/add/:condId/unit`.
- Campos: tipo (Residencial/Comercial); número; bloco; vínculo; papel.
- Regras exibidas: 1 titular/unidade; até 2 dependentes.
- [Avançar] → APP-MOR-013.

### APP-MOR-013 — Termo de ciência

- **Rota**: `/condominium/add/:condId/termo`.
- Texto completo (igual web M5).
- Checkbox + [Confirmar e solicitar acesso] → APP-MOR-014.

### APP-MOR-014 — Status da unidade

- **Rota**: `/condominium/add/:condId/status`.
- Radio 6 opções.
- [Salvar e continuar] → retorna pra `/home` com novo condomínio ativo.

---

## 5. Síndico (mobile light) — M2

Mobile M2 implementa **operações rápidas** do síndico (voice-first, campo); configurações complexas (matriz permissões, compliance completo, relatórios) ficam **web only**.

### APP-SIN-001 — Governança (entrada)

- **Rota**: `/sindico`.
- Para síndicos: bottom nav adicional "Governança" (5ª aba).
- Cards: Central (dashboard resumo); Plano Diretor; Timeline (novo registro); Eventos (criar rápido); Votações; Connect Me; Validações pendentes.

### APP-SIN-002 — Central (resumo)

- **Rota**: `/sindico/central`.
- Cards resumidos (subset S7): Plano Diretor — total/vencendo; Validações pendentes; Próxima assembleia; Avaliação média.

### APP-SIN-003 — Timeline — novo registro rápido (voice-first)

- **Rota**: `/sindico/timeline/novo`.
- Form simplificado (R2 voice-first):
  - Botão **mic grande** no topo: grava descrição por voz (`speech_to_text` plugin M2).
  - Tipo atividade (bottom sheet chooser 12 opções).
  - Importância (chip selector).
  - Fotos (câmera nativa + galeria, compressão cliente BR-MOB-CAM-003).
  - Visibilidade: Gestão / Público.
- [Publicar] → modal R1 confirmação ("Publicar para [Condomínio]?") → submit.

### APP-SIN-004 — Plano Diretor — lista + nova ação

- **Rota**: `/sindico/plano-diretor`.
- Lista resumida (filtro rápido).
- FAB [+] → `/sindico/plano-diretor/novo`.

### APP-SIN-005 — Plano Diretor — nova ação (form completo mobile)

- **Rota**: `/sindico/plano-diretor/novo`.
- Stepper vertical 4 steps: Básico (título + tipo + descrição voice); Classificação (natureza + risco + importância + áreas); Prazos (início + limite + responsável); Revisão + publicar.

### APP-SIN-006 — Eventos — criar

- **Rota**: `/sindico/eventos/novo`.
- Form adaptado mobile: tipo (bottom sheet 19 opções); título; descrição voice; data início/fim (native pickers); local; visibilidade.
- [Publicar] → R1 modal.

### APP-SIN-007 — Validações pendentes (fila)

- **Rota**: `/sindico/validacoes`.
- Lista cards: origem (empresa verificada/não), tipo (proposta/serviço/comunicado), preview.
- **Swipe left** [Rejeitar]; **swipe right** [Validar]; tap detalhes → full screen com actions.

### APP-SIN-008 — Connect Me — nova solicitação (síndico)

- **Rota**: `/sindico/connect-me/nova`.
- Form mobile otimizado (voice-first description).
- Texto LGPD + checkbox.
- [Enviar].

### APP-SIN-009 — Indicadores (resumo)

- **Rota**: `/sindico/indicadores`.
- Cards em swipe horizontal: adesão; eventos participação; votações; pendências por tipo.

### APP-SIN-010 — Modo apresentação rápida

- **Rota**: `/sindico/apresentacao`.
- Slides (swipe horizontal): indicadores principais; próximas ações; riscos críticos; eventos próximos; decisões recentes.
- Botão "Entrar em modo full-screen" → esconde status bar.

---

## 6. Assembleia Live (M3)

Feature `assembly/` **já tem infra real** (entidades, usecases, bloc, 5 páginas) — M3 completa a live session e integra com backend WebSocket + Livekit.

### APP-ASM-001 — Lista de assembleias (implementada)

- **Rota real**: `/condominiums/:condominiumId/assemblies` (morador e síndico; diferenciação via ABAC).
- Página real: `AssemblyListPage` com `BlocProvider(create: (_) => sl<AssemblyBloc>()..add(AssemblyListRequested(...)))`, `RefreshIndicator`, `ListView.separated` + `AssemblyCard`.
- Estados reais: `AssemblyLoading` (spinner), `AssemblyListLoaded(assemblies)` (lista ou empty "Nenhuma assembleia encontrada"), `AssemblyFailureState(message)` (snackbar + shrink).
- Alvo M3: abas Agendadas | Em andamento | Encerradas.

### APP-ASM-002 — Detalhe da assembleia (implementada)

- **Rota real**: `/condominiums/:condId/assemblies/:asmId`.
- Página real: `AssemblyDetailPage` → status chip (rascunho/publicada/em_andamento/encerrada), título, descrição, info rows (data, tipo, modalidade, quórum), `_ActionBar` ([Edital], [Registrar ciência]), `_AgendaSection` (lista `AgendaItemCard` com botão [Votar] se `isVotingOpen`).
- Ação AppBar: ícone article_outlined → `/minutes`.
- Estados reais: `AssemblyLoading`, `AssemblyDetailLoaded(assembly)`, `AssemblyFailureState`.

### APP-ASM-003 — Ciência da pauta / edital (implementada)

- **Rota real**: `/condominiums/:condId/assemblies/:asmId/science?v=1`.
- Página real: `AssemblySciencePage` — ícone `description_outlined`, texto "Declaração de Ciência do Edital (versão X)", `CheckboxListTile`, botão `ElevatedButton` com spinner.
- Submit real: `AssemblyScienceRegistered(condominiumId, assemblyId, editalVersion)` → `RegisterScienceUseCase` → backend `POST /science`.
- Pós-sucesso: snackbar "Ciência registrada com sucesso!" + `Navigator.pop()`.
- Alvo M3: listar itens de pauta com checkbox individual "Ciente de [item]".

### APP-ASM-004 — Outorgar procuração (M3)

- **Rota**: `/home/assembleias/:id/procuracao`.
- Página real (domain): `CreateProxyUseCase` pronto; bloc `AssemblyProxyCreated` event e `AssemblyProxySuccess(proxy)` state. UI pendente.
- Campos: para quem (search membership); tipo (específica/geral); `validUntil` duração; itens específicos se aplicável.
- Assinatura digital (biometria `local_auth` + payload signed pelo backend) — BR-MOB-BIO-003.

### APP-ASM-005 — Check-in (M3)

- **Rota**: `/home/assembleias/:id/checkin`.
- Opções: [Escanear QR] (camera + `mobile_scanner` ou `qr_code_scanner`), [Código manual], [Check-in remoto].
- Registra `AttendanceRecord` no backend.

### APP-ASM-006 — Live session (M3)

- **Rota**: `/home/assembleias/:id/live`.
- Layout split vertical mobile:
  - **Top 40%**: vídeo Livekit (`livekit_client` Flutter SDK).
  - **Bottom 60%**: painel dinâmico — pauta ativa + votação aberta + fila de fala + chat metadata.
- Ações morador: [Solicitar fala]; [Votar Aprovar/Rejeitar/Abster]; [Ausência 15min].
- Realtime WS broadcast com live region `semanticsLabel` para screen reader anunciar mudanças.
- Full-screen toggle pra vídeo maximizado.
- **`FLAG_SECURE`** ativo — evita screenshot.

### APP-ASM-007 — Votação num item de pauta (implementada)

- **Rota real**: `/condominiums/:condId/assemblies/:asmId/vote/:agendaItemId?title=...`.
- Página real: `AssemblyVotePage` — ícone `how_to_vote`, título do item, disclaimer "Seu voto é secreto e definitivo", `VoteButtonRow` widget (Aprovar/Rejeitar/Abster).
- Submit real: `AssemblyVoteCast(condominiumId, assemblyId, agendaItemId, voteValue)` → `CastVoteUseCase` → backend `POST /agenda-items/:id/votes`.
- Pós-sucesso: snackbar "Voto registrado!" + `Navigator.pop()`.
- **M1 obrigatório**: `bloc_concurrency.droppable()` no handler `_onVoteCast` — double-tap hoje ainda é problema.

### APP-ASM-008 — Ata da assembleia (implementada)

- **Rota real**: `/condominiums/:condId/assemblies/:asmId/minutes`.
- Página real: `AssemblyMinutesPage` — `_ScoreChip(score)` transparência (cores: verde ≥80, laranja ≥50, vermelho <50), badge "Gerada automaticamente" se `isAutoGenerated`, publishedAt, divider, content.
- Estado error: ícone `error_outline` + "Ata não disponível" + [Tentar novamente].
- Alvo M3: PDF viewer embed + download button.

---

## 7. Connect Me morador (M2)

### APP-CM-001 — Hub

- **Rota**: `/home/connect-me`.
- Cards: Nova solicitação (morador → síndico ou empresa); Minhas solicitações enviadas; Quota restante (respeitando matriz D-094: Morador Base = 2/ano; quando saldo = 0 mostrar card "Faça upgrade" para Pagante 4/ano).

### APP-CM-002 — Nova solicitação morador

- **Rota**: `/home/connect-me/nova`.
- Campos: categoria; descrição voice; urgência; prazo; anexos; consent LGPD (BR-MOB-LGPD-002 obrigatório).
- [Enviar] → decrementa quota backend (I-030); `bloc_concurrency.droppable()` no handler.

### APP-CM-003 — Minhas solicitações

- **Rota**: `/home/connect-me/enviadas`.
- Lista status (aberta/respondida/encerrada).

---

## 8. Vídeo morador (M2)

### APP-VID-001 — Player Mux

- Componente inline (não rota própria) — `shared/widgets/mux_player.dart`.
- Aspect ratio 16:9 default; expande full-screen.
- Lock badge "🔒 Travado até DD/MM" quando `locked_until > now` (I-031 Lock 90d).
- Controls: play/pause, seek, speed, captions (se disponível), full-screen.
- Quality auto baseado em conexão.

### APP-VID-002 — Upload vídeo-currículo morador (Pagante)

- **Rota**: `/account/video-curriculo`.
- Fluxo: [Gravar] (camera até 90s) ou [Selecionar galeria] (max 90s, max 200MB).
- Upload direto Mux via presigned (GR-019).
- Progress bar + cancel.
- Após upload: status "Em processamento" → "Em moderação" → "Aprovado" / "Rejeitado" (push notification FCM categoria `connect_me` ou `content`).
- Lock 90d pós-approved (I-031); bypass exige 4-eyes admin (D-054).

### APP-VID-003 — Visualizar vídeo institucional empresa (durante votação)

- Parte de APP-MOR-007 Votação fornecedor.
- Backend gera signed URL com TTL curto (≤ 1h) durante sessão ativa (I-034).
- Badge "Vídeo institucional da empresa [Nome]" + disclaimer LGPD.

---

## 9. Perfil, conta, assinatura

### APP-ACC-001 — Conta (home)

- **Rota**: `/account`.
- Seções: perfil; meus condomínios; assinatura + trial; segurança (senha/biometria/sessões ativas); preferências (notificações/idioma/tema); privacidade (LGPD); sobre (versão+termos).

### APP-ACC-002 — Perfil

- **Rota**: `/account/perfil`.
- Campos editáveis (dependendo persona): foto perfil; nome completo; data nascimento (read-only); email; telefone; endereço.
- Síndico N2+ (equivalente matriz): adicional mini bio, vídeo institucional (link web-only), formação, administradoras.
- **Regra**: dados obrigatórios bloqueiam acesso completo até preenchidos (DADOS PARA CADASTRO §Síndico).

### APP-ACC-003 — Assinatura

- **Rota**: `/account/assinatura`.
- Exibe plano atual (ID global D-057), trial countdown, próxima cobrança (se paid), features do plano.
- [Fazer upgrade] → abre URL Stripe Checkout em Custom Tab/ASWebAuthenticationSession (BR-MOB-BILL-002).

### APP-ACC-004 — Segurança

- **Rota**: `/account/seguranca`.
- Toggles: usar biometria na próxima abertura; exigir biometria em ações críticas; sessões ativas (lista com device + ação [Encerrar]).
- Regra 1-device: só 1 sessão mobile ativa por vez (IR-011).

### APP-ACC-005 — Preferências notificações

- **Rota**: `/account/notificacoes`.
- Toggles por categoria (6): eventos; votações; Connect Me; assembleia live; comunicados; compliance; marketing (opt-in explícito).

### APP-ACC-006 — Privacidade / LGPD

- **Rota**: `/account/privacidade`.
- Ações: [Baixar meus dados] (trigger `/me/export` → recebe link por email); [Excluir conta] (confirmação dupla + re-auth + **biometria obrigatória** BR-MOB-BIO-003); [Histórico de compartilhamentos Connect Me].

### APP-ACC-007 — Sobre

- **Rota**: `/account/sobre`.
- Versão do app, build, commit hash (de `--dart-define=GIT_SHA`), termos, política, licenças open-source (gerado via `flutter_oss_licenses`).

---

## 10. Notificações

### APP-NOT-001 — Lista de notificações (inbox)

- **Rota**: `/notifications`.
- ListView de `NotificationItem` (título, preview, timestamp, unread badge).
- Tap → deep-link pra rota correspondente (via `app_links` + GoRouter).
- Pull-to-refresh.
- Swipe action: mark read/unread; delete local.

### APP-NOT-002 — Notification push (foreground)

- In-app toast/snackbar quando recebe push em foreground.
- Layout: ícone + título + preview + ação [Ver] (se aplicável).

### APP-NOT-003 — System notification (background/terminated)

- Notificação nativa Android/iOS com deep-link no tap → abre rota.
- Channels/Categories (6 — BR-MOB-PUSH-003): events, connect_me, votes, assembly_live, compliance, marketing.

---

## 11. Exceções / offline

### APP-ERR-001 — Sem conexão (banner + tela)

- Banner topo: "Sem conexão. Algumas funções podem não funcionar".
- Em rotas de escrita: bloqueio com modal "Sem conexão. Sua ação será sincronizada quando reconectar" + queue Hive `sync_queue` (BR-MOB-OFF-003).
- Em rotas leitura: mostra cached data com label "Cached (há X min)".

### APP-ERR-002 — 401 unauthorized

- Interceptor Dio tenta refresh; falha → logout + redirect `/login` preservando `?redirect=<original>`.

### APP-ERR-003 — 403 NEW_DEVICE_DETECTED

- Redirect APP-SYS-004.

### APP-ERR-004 — 429 rate-limited

- Modal: "Muitas solicitações. Tente novamente em [Xs]" (backend retorna `Retry-After`).

### APP-ERR-005 — Upload falhou

- Tela de erro com [Retry] e [Cancelar]. Queue local se possível (não p/ vídeo — exige network estável).

### APP-ERR-006 — Low battery / low memory

- Warning em flows críticos (assembleia live): "Bateria baixa (X%). Conectar carregador recomendado".

### APP-ERR-007 — App version outdated

- APP-SYS-005 modal obrigatório apenas se feature crítica depende.

### APP-ERR-008 — Integrity (D-050 M3)

- APP-SYS-006 em features críticas quando `integrity != ok`.

---

## 12. Admin role gated (D-058)

Admin **não tem app separada**. Nos builds (`dev/staging/prod`), as telas admin aparecem **dentro da mesma app** gated por permissão ABAC retornada em `/auth/me`.

### APP-ADM-GATE — Bottom nav extra "Admin"

- Renderizada **apenas** se token contém permissão `admin.access`.
- 5ª/6ª aba "Admin" → `/admin`.

### APP-ADM-001 — Admin home

- **Rota**: `/admin`.
- Cards: Usuários (busca); Condomínios (busca); Financeiro (resumo); Moderação (fila); Auditoria (últimos eventos); Bypass Lock 90d pendentes (4-eyes — D-054).

### APP-ADM-002 — Bypass Lock 90d — solicitar (mobile entry)

- **Rota**: `/admin/lock-bypass/novo`.
- Campos: recurso (ID do vídeo/contrato); categoria (erro operacional / ordem judicial / fraude / LGPD); motivo (texto ≥ 20 chars).
- [Submeter] → solicitação vai para fila; Admin B aprova em mobile **ou** web dentro de 24h (D-054).

### APP-ADM-003 — Bypass Lock 90d — aprovar (mobile entry)

- **Rota**: `/admin/lock-bypass/pendentes`.
- Lista solicitações onde Admin A ≠ current user.
- Tap → tela detalhe com contexto + [Aprovar] (biometria obrigatória) / [Rejeitar] (exige motivo).

> Telas admin complexas (relatórios globais multi-tenant, pipeline Stripe, dashboards financeiros) são [[../web/ui-catalog|web only]]. Mobile admin é **operação rápida** de campo.

---

## 13. Paridade com web — matriz

| Feature | Web | Mobile M1 | Mobile M2 | Mobile M3 |
|---|---|---|---|---|
| Onboarding morador | ✅ | ✅ | — | — |
| Painel morador | ✅ | ✅ | — | — |
| Timeline (read) | ✅ | ✅ | — | — |
| Plano Diretor (read) | ✅ | ✅ | — | — |
| Eventos morador | ✅ | ✅ | — | — |
| Votação fornecedor (morador) | ✅ | — | ✅ | — |
| Connect Me morador | ✅ | — | ✅ | — |
| Vídeo-currículo upload | ✅ | — | ✅ | — |
| Assembleia detalhe + edital + ciência | ✅ | ⚠️ parcial no código | ✅ | ✅ |
| Assembleia voto item | ✅ | ⚠️ parcial no código | ✅ | ✅ |
| Assembleia procuração | ✅ | — | — | ✅ |
| Assembleia check-in | ✅ | — | — | ✅ |
| Assembleia live session | ✅ | — | — | ✅ |
| Central síndico | ✅ | — | ✅ (light) | ✅ (pleno) |
| Timeline novo (voice-first) | ✅ | — | ✅ | ✅ |
| Plano Diretor novo | ✅ | — | ✅ | ✅ |
| Eventos criar | ✅ | — | ✅ | ✅ |
| Votação criar | ✅ | — | — | ✅ |
| Validações pendentes | ✅ | — | ✅ | ✅ |
| Connect Me síndico | ✅ | — | ✅ | ✅ |
| Assembleia criar | ✅ | — | — | ✅ |
| Compliance completo | ✅ | — | — | — (web only) |
| Empresa dashboard | ✅ | — | — | — (web only) |
| Parceiro dashboard | ✅ | — | — | — (web only) |
| Agência Marketing dashboard | ✅ | — | — | — (web only) |
| Admin role (D-058) — operações rápidas | ✅ | — | ✅ (básico) | ✅ (4-eyes bypass) |
| Admin relatórios / pipelines complexos | ✅ | — | — | — (web only) |
| LMS morador + síndico | ✅ | — | — | ✅ thin |
| LP institucional | ✅ | — | — | — (web only) |

**Código real hoje**: `assembly/` tem página detalhe + lista + ciência + voto + minutes prontas — mas tudo sob rotas `/condominiums/:id/assemblies/*` fora do shell M1. Reorganizar em M1 para caber no `/home` shell e integrar com bottom nav + selector de condomínio.

---

## 14. Pendências dual-check

- **P-APP-001** — D-041 Hive vs sqflite vs drift (alvo: Hive).
- **P-APP-002** — Livekit Flutter SDK spec review (M3).
- **P-APP-003** — Certificate pinning implementação: custom `IOHttpClientAdapter` vs `http_certificate_pinning` package vs `ssl_pinning_plugin` — checar manutenção atual.
- **P-APP-004** — Upload fotos execução síndico mobile — M2 ou M3.
- **P-APP-005** — Push notifications channels Android — alinhar com backend payload canônico (6 categorias, BR-MOB-PUSH-003).
- **P-APP-006** — Reconciliar paleta mobile (`#6C63FF` roxo) com Manual IV do cliente (`#C69C3F` ouro + `#071A33` navy) — decidir antes de M1 ship.
- **P-APP-007** — `NoParams` mobile não estende `Equatable` — refactor inofensivo; mas `registerFallbackValue` em todos os testes fica verboso.
- **P-APP-008** — `minSdk = 19` no Android atual; BR-MOB-STORE-003 exige 26 — elevar antes de M1.
- **P-APP-009** — Bundle ID Android atual `com.mastersindico.br.mastersindico`; alvo canônico `com.mastersindico.app` — normalizar (migração breaks update path se publicado).

---

## 15. Links

- [[README]]
- [[architecture]]
- [[patterns]]
- [[security]]
- [[../web/ui-catalog]] — jornadas completas (mobile é subset)
- [[../web/design-system]] — tokens espelhados
- [[../../00-product/personas]]
- [[../../STATE#D-058|D-058 Admin = role privilegiada]]
