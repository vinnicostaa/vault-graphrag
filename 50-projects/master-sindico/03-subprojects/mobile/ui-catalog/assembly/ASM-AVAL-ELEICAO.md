---
title: ASM-AVAL-ELEICAO — Avaliação Escondida (mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, flutter, assembleia, eleicao, lgpd]
project: master-sindico
persona: morador
category: ASM
screen_id: ASM-AVAL-ELEICAO
sub_produto: assembleia-live
plan_requirement: any
status: specification
stack: mobile
milestone: m3
created: 2026-04-24
---

# ASM-AVAL-ELEICAO — Avaliação Escondida (mobile)

## Finalidade

Espelho mobile da tela web [[../../../web/ui-catalog/assembleia/ASM-AVAL-ELEICAO]]. Morador na assembleia ao vivo via app é forçado a avaliar gestão do síndico atual antes de votar no candidato a eleição. Confidencial.

## Persona e ABAC

Ver web. Mobile-específico:
- Morador presencial via app (check-in por QR ou localização)
- Notificação push aparece quando presidente abre pauta `eleicao_sindico` — deep-link para esta tela

## Fluxo (Flutter)

1. Push notification `ElectionPauta.Opened` chega
2. User tap → deep-link `master-sindico://assembly/:id/hidden-assessment`
3. AuthGuard valida token + membership
4. `HiddenAssessmentBloc` verifica status via `/hidden-assessments/me/status`
5. Se não submeteu → exibe tela; se já submeteu → redirect para tela de voto (ASM-VOTO mobile)
6. Tela mostra:
   - AppBar sem back button (NavGuard) + texto "Avaliação da gestão — mandato X"
   - Card amarelo com aviso LGPD "Esta avaliação é confidencial..."
   - 5 questions em PageView (uma por página) OU lista scroll (decidir em UX)
   - Cada question: slider 1-10 ou RadioGroup vertical
   - Bottom CTA "Submeter" disabled até completar
7. Tap Submit → `POST /hidden-assessments` → loading → success dialog (sem score) → redirect ASM-VOTO mobile

## Widgets Flutter

- `HiddenAssessmentPage` (StatefulWidget + BLoC)
- `ConfidentialBannerCard` (novo) — card com ícone lock + texto destacado
- `QuestionSlider` (novo) — slider 1-10 com labels semânticos
- `SubmitButton` (existente — design-system mobile)
- `NavGuardWrapper` — intercepta `WillPopScope` bloqueando back button
- `ConfirmationDialog` (existente) — pós-submit

## Platform-specific notes

- **iOS**: WillPopScope substituído por `PopScope(canPop: false)` (Flutter 3.12+) — interceptar swipe-back
- **Android**: hardware back button interceptado pelo NavGuardWrapper
- **SafeArea**: respeitar notch/ilha dinâmica em iPhone 14+
- **Keyboard**: se tiver campo texto opcional, `resizeToAvoidBottomInset: true`
- **Accessibility**: labels semânticos em slider; screen reader diz "pergunta 1 de 5: transparência; valor atual 7"

## Offline behavior

- **Não permite offline** — assembleia ao vivo requer conexão (votação real-time). Se conexão cair durante preenchimento:
  - Banner "Sem conexão — sua avaliação será submetida quando restabelecer"
  - BLoC mantém respostas em estado local (não persiste em Hive — é confidencial, volatile apenas)
  - Retry automático a cada 10s
- Se morador submete e perde conexão antes de 201 → avaliação já gravada no servidor (idempotência protege duplicata pelo UNIQUE)

## Push notification

- **Trigger**: backend detecta `AssemblyPautaOpened(id, type=eleicao_sindico)` → envia push para todos os check-in'd
- **Payload**:
  ```json
  {
    "type": "election_pauta_opened",
    "assembly_id": "...",
    "mandate_id": "...",
    "deep_link": "master-sindico://assembly/<id>/hidden-assessment",
    "title": "Eleição de síndico — avaliação obrigatória",
    "body": "Antes de votar, avalie a gestão atual (confidencial)."
  }
  ```
- **Canal**: `assembly_urgent` (canal dedicado Android O+; crítico para não silenciar)

## Integração com backend

Mesma do web — ver [[../../../web/ui-catalog/assembleia/ASM-AVAL-ELEICAO#integração-com-backend]].

## Regras de negócio

Mesmas do web. Mobile-específicas:
- App NUNCA guarda respostas em SharedPreferences/Hive — só estado BLoC volátil
- Se app crash durante preenchimento: respostas perdidas, morador refaz (aceitável dada a criticidade)
- Biometria (`flutter_local_auth`) opcional antes de submeter — configurável por condomínio (decisão aberta)

## Ligações

- [[../../../web/ui-catalog/assembleia/ASM-AVAL-ELEICAO]] — versão web
- [[../../../../01-domain/invariants#INV-ASM-023]]
- [[../../../../04-requirements/functional/assembly#ASM-ELE-AVAL]]
- [[../../../../STATE#D-104]]
- [[../assembly/ASM-VOTO]] — tela destino
- [[../../requirements/assembly-vote]]

## Gaps/ressalvas

- **Push permission**: iOS requer user grant — se morador bloqueou, cai para polling 10s da UI aberta
- **Biometria antes de submeter**: opcional, decisão produto
- **Accessibility VoiceOver/TalkBack**: labels completos pendentes (specifista a11y)
- **Strings literais**: a confirmar com produto (banner LGPD, títulos, mensagens erro)

## Links

- [[_moc]]
- [[../../CLAUDE]] *(via config mobile)*
- [[../../requirements/_moc]]
