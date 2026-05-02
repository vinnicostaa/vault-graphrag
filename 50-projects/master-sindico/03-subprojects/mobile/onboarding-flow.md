---
title: Onboarding Flow — Mobile Flutter
type: spec
tags: [master-sindico, mobile, flutter, onboarding, flow]
project: master-sindico
stack: mobile
status: specification
created: 2026-04-24
---

# Onboarding Flow — Mobile

Diagrama end-to-end dos **4 fluxos** de onboarding mobile (morador / síndico / empresa / parceiro). Cada fluxo começa após login OIDC bem-sucedido e termina quando a persona está apta a entrar no shell principal.

## Premissas

- Zitadel ID token carrega claim `role` (`resident | syndic | company | partner`) — roteamento inicial decide persona.
- Cada step chama `PATCH /api/v1/onboarding/sessions/me` com auto-save debounce 2s.
- Retomada: ao abrir app com sessão ativa e onboarding não completo → `GET /sessions/me` retorna step corrente.
- `PATCH` com status `complete` dispara binding final + redirect home.

## Diagrama Mermaid

```mermaid
flowchart TD
  LOGIN[[AUTH-M1 Login OIDC]] -->|sucesso + claim role| ROUTE{Router decide<br/>persona}

  ROUTE -->|role=resident| M_START[ONB-M2<br/>Buscar Condomínio]
  ROUTE -->|role=syndic| S_START[ONB-S2<br/>Selecionar Condomínio]
  ROUTE -->|role=company| E_START[ONB-E2<br/>Dados Empresa]
  ROUTE -->|role=partner| P_START[ONB-P2<br/>Tipo Parceiro]

  %% Morador
  M_START --> M3[ONB-M3<br/>Identificação Unidade]
  M3 --> M4[ONB-M4<br/>Termo de Ciência]
  M4 --> M5[ONB-M5<br/>Status Unidade - proprietário/inquilino]
  M5 --> MFIN[ONB-MFIN<br/>Tudo Pronto!]
  MFIN --> HOME_M[[Home Morador M1]]

  %% Síndico
  S_START --> S3[ONB-S3<br/>Dados Pessoais]
  S3 --> S4[ONB-S4<br/>Upload Ata de Posse]
  S4 --> S5[ONB-S5<br/>Selfie com Documento]
  S5 --> S6[ONB-S6<br/>Termo + Assinatura]
  S6 --> SFIN[ONB-SFIN<br/>Análise em Curso]
  SFIN --> HOME_S[[Home Síndico S6]]

  %% Empresa
  E_START --> E3[ONB-E3<br/>CNPJ + Razão Social]
  E3 --> E4[ONB-E4<br/>Endereço + Contato]
  E4 --> E5[ONB-E5<br/>Área de Atuação]
  E5 --> E6[ONB-E6<br/>Docs - Contrato Social + CND]
  E6 --> E7[ONB-E7<br/>Banco Talentos Opt-in]
  E7 --> EFIN[ONB-EFIN<br/>Análise KYB]
  EFIN --> HOME_E[[Home Empresa - web primário]]

  %% Parceiro
  P_START --> P3[ONB-P3<br/>Dados Básicos]
  P3 --> P4[ONB-P4<br/>Termo Comissão]
  P4 --> PFIN[ONB-PFIN<br/>Aguardando Ativação]
  PFIN --> HOME_P[[Home Parceiro - web primário]]

  %% Comum
  M4 -.->|recusa termo| ABORT[LOGOUT + tela final]
  S6 -.->|recusa| ABORT
  E6 -.->|docs inválidos| E6_RETRY[Re-upload]
  E6_RETRY --> E6

  classDef resident fill:#e3f2fd,stroke:#1976d2
  classDef syndic fill:#fff3e0,stroke:#f57c00
  classDef company fill:#f3e5f5,stroke:#7b1fa2
  classDef partner fill:#e8f5e9,stroke:#388e3c
  class M_START,M3,M4,M5,MFIN resident
  class S_START,S3,S4,S5,S6,SFIN syndic
  class E_START,E3,E4,E5,E6,E7,EFIN company
  class P_START,P3,P4,PFIN partner
```

## Telas por persona

### Morador (5 telas efetivas)
1. `ONB-M2` Buscar condomínio (CEP + número)
2. `ONB-M3` Identificação unidade (bloco/apto/fração)
3. `ONB-M4` Termo de ciência
4. `ONB-M5` Status unidade (proprietário/inquilino/dependente)
5. `ONB-MFIN` Tudo pronto! → home

### Síndico (5 telas + final)
1. `ONB-S2` Selecionar condomínio onde foi eleito
2. `ONB-S3` Dados pessoais (CPF, RG, contato)
3. `ONB-S4` Upload ata de posse
4. `ONB-S5` Selfie com documento
5. `ONB-S6` Termo de uso síndico + assinatura digital
6. `ONB-SFIN` Análise em curso (async — síndico pode usar módulos read-only enquanto aguarda)

### Empresa parceira (6 telas + final)
1. `ONB-E2` Dados empresa (trade name)
2. `ONB-E3` CNPJ + razão social (via consulta serasa/CNPJ lookup)
3. `ONB-E4` Endereço + contato
4. `ONB-E5` Área de atuação (multi-select categorias)
5. `ONB-E6` Docs: contrato social + CND federal/estadual
6. `ONB-E7` Opt-in banco de talentos (D-074)
7. `ONB-EFIN` Análise KYB (async)

### Parceiro comercial (3 telas + final)
1. `ONB-P2` Tipo parceiro (agência / indicador)
2. `ONB-P3` Dados básicos + CNPJ/CPF
3. `ONB-P4` Termo de comissão
4. `ONB-PFIN` Aguardando ativação

## Decisões e padrões

- **Auto-save**: debounce 2s em cada step; se offline, enfileira em `onboarding_queue` (Hive).
- **Retomada**: `hydrated_bloc` em `OnboardingBloc` persiste `currentStep` + `payload`.
- **Abort**: recusa de termo mata sessão + logout Zitadel.
- **Async finalize**: síndico/empresa/parceiro entram em estado `pending_review`; podem acessar módulos restritos (leitura) enquanto aguardam.
- **Morador** tem `auto_approve`: binding imediato pós-ONB-M5.

## Gaps/questões abertas

- **Q-FLOW-01** Parceiro no app: UX essencial é web; mobile = fluxo redundante; avaliar descomissionar em M1.
- **Q-FLOW-02** Empresa no app similar — priorizar web M1. Mobile empresa pode ficar M3.
- **Q-FLOW-03** Onboarding cross-device (começa web, termina mobile) depende de server manter estado — ver `[[../backend/requirements]]`.

## Links

- [[requirements/onboarding]]
- [[ui-catalog/onboarding/_moc]]
- [[../web/ui-catalog/onboarding/_moc]] (equivalente web, a popular)
- [[_moc]]
