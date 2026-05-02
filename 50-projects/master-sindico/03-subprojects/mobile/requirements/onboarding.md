---
title: Onboarding — Mobile Flutter
type: requirement
tags: [master-sindico, mobile, flutter, identity, institutional, feature-req]
project: master-sindico
stack: mobile
priority: m1
status: pending
bounded_context: identity
created: 2026-04-24
---

# Onboarding — Mobile

Wizard pós-cadastro que coleta dados estruturais para vincular o usuário ao bounded context correto. **4 fluxos** por persona com **auto-save por step** e **retomada** após fechar o app.

## Escopo M1/M2/M3

- **M1**: fluxo morador completo (busca condomínio → identificação unidade → termo → status ativo).
- **M1**: fluxo síndico completo (condomínio + documentos + cadastro base).
- **M2**: fluxo empresa parceira (4-7 telas: dados + docs + banco talentos).
- **M2**: fluxo parceiro comercial (3-5 telas).
- **M3**: upsell in-app para upgrade de plano pós-onboarding.

## Telas envolvidas

Morador: `ONB-M2 ONB-M3 ONB-M4 ONB-M5 ONB-MFIN`
Síndico: `ONB-S2 ONB-S3 ONB-S4 ONB-S5 ONB-S6`
Empresa: `ONB-E2 ONB-E3 ONB-E4 ONB-E5 ONB-E6 ONB-E7`
Parceiro: `ONB-P2 ONB-P3 ONB-P4`

Web equivalente: `[[../../web/ui-catalog/onboarding/_moc]]` (quando criado).

## Funcionais (FR-MOB-ONB-N)

- **FR-MOB-ONB-1** Primeira abertura pós-login sem vínculo → navegar para `ONB-<persona>2` baseado em claim `role` do ID token.
- **FR-MOB-ONB-2** Cada step salva parcial via `PATCH /api/v1/onboarding/sessions/me` (auto-save debounce 2s) → retomada fiel mesmo após kill app.
- **FR-MOB-ONB-3** Step "Buscar Condomínio" (morador): input CEP + número + autocomplete via `GET /api/v1/institutional/condominiums/search?cep=...&number=...`; resultado selecionado → cria `Membership` pendente.
- **FR-MOB-ONB-4** Step "Identificação Unidade" (morador): bloco + apto + fração ideal (read-only, vem do condomínio selecionado).
- **FR-MOB-ONB-5** Step "Termo de Ciência" (morador + síndico): texto versionado (C2 compliance) + checkbox + timestamp de aceite persistido.
- **FR-MOB-ONB-6** Step final: tela institucional "Tudo pronto!" + CTA "Ir para o painel" → `/home`.
- **FR-MOB-ONB-7** Síndico upload de documentos (ata de posse, CPF, selfie com doc) via presigned URL; compressão cliente-side pre-upload (img max 2MB JPEG Q85).
- **FR-MOB-ONB-8** Progress bar não-linear (3/5 etc) em header; botão voltar preserva dados.
- **FR-MOB-ONB-9** Offline: step form fica editável; submit fica enfileirado até conectar (banner "Envio pendente").

## Não-funcionais (NFR-MOB)

- **NFR-MOB-A11Y-1** Touch targets ≥ 48dp; labels em `Semantics`.
- **NFR-MOB-A11Y-2** `textScaler` respeitado; nada de texto com altura hard-coded.
- **NFR-MOB-PERF-1** Cada step < 200ms para renderizar (skeleton otimista).
- **NFR-MOB-I18N-1** pt-BR first; termos condominiais preservados (CPF, CNPJ, bloco, unidade, fração ideal).

## Dados locais

- **Hive box `onboarding_draft`**: step atual + payload acumulado + hash do último sync.
- **SharedPreferences**: `onboarding_v1_completed_at` (timestamp após FIN).
- **Secure Storage**: nenhum (onboarding não toca segredos).
- **Invalidação**: após `POST /sessions/me/complete` 200 → limpar draft local.

## Integração backend

- `GET /api/v1/onboarding/sessions/me` — retoma sessão.
- `PATCH /api/v1/onboarding/sessions/me` — auto-save.
- `POST /api/v1/onboarding/sessions/me/complete` — finaliza.
- `GET /api/v1/institutional/condominiums/search` — busca.
- `POST /api/v1/storage/presign` — upload docs síndico.

Sem WebSocket.

## Padrões Flutter aplicáveis

- `OnboardingBloc` com state freezed: `OnboardingStep(current, total, payload, persona)`.
- Sub-blocs por step (`StepCondominioBloc`, `StepUnidadeBloc`) composição via `MultiBlocProvider`.
- `bloc_concurrency.sequential()` em auto-save (ordem importa, evita race na PATCH).
- `hydrated_bloc` persiste `OnboardingStep` entre kills.
- `freezed` em `OnboardingPayload` com `union` por persona (sealed class + copyWith gerado).
- `go_router` aninhado `/onboarding/:persona/:step`.

## Gaps/decisões abertas

- **Q-MOB-ONB-01** Upload de foto da unidade (morador) é opcional? Assumido opcional em M1.
- **Q-MOB-ONB-02** Retomada cross-device (logar em novo device e continuar onboarding) depende de servidor guardar estado — ver `[[../../backend/requirements]]`.
- **Q-MOB-ONB-03** Fluxo parceiro só tem 3 telas confirmadas; cliente pode adicionar "confirmação bancária" tardiamente.

## Links

- [[../../../_moc]]
- [[../../../00-product/sub-produtos/01-governanca-institucional]]
- [[../../../00-product/sub-produtos/11-administradoras-condominiais]]
- [[../../web/requirements/_moc]]
- [[onboarding-flow]] diagrama mermaid
