---
title: MOC — Onboarding (ui-catalog web)
type: moc
tags: [master-sindico, ui-catalog, web, onboarding, moc]
project: master-sindico
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# MOC — Onboarding por Persona (Web)

**25 telas** cobrindo os 4 fluxos de cadastro: Morador (4 telas) · Síndico (6) · Empresa (7) · Parceiro (4) + **Tela 0** (boas-vindas/seleção de perfil) e **Tela 1** (identificação inicial) comuns a todas.

Fonte canônica: [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] + [[../../../../../../90-archive/master-sindico-pdfs-cliente-originais/ONBOARDING DOS PERFIS]].

---

## Telas comuns (2)

- [[ONB-00-boas-vindas]] — Seleção de perfil (Morador/Síndico/Empresa/Parceiro)
- [[ONB-01-identificacao-inicial]] — Nome + E-mail (auto-save Redis TTL 30d)

## Fluxo Morador (4 telas + confirmação)

- [[ONB-M2]] — Identidade (foto, nome, nascimento, tel, CPF)
- [[ONB-M3]] — Vínculo com o Condomínio (CEP, endereço, código 8 chars)
- [[ONB-M4]] — Participação e Convivência (3 aceites)
- [[ONB-MFIN]] — Confirmação Final

## Fluxo Síndico (6 telas + confirmação)

- [[ONB-S2]] — Identidade de Gestão (foto profissional, CPF, e-mail profissional)
- [[ONB-S3]] — Sua Atuação (endereço, tipo síndico, tempo, qtd condomínios)
- [[ONB-S4]] — Governança e Estrutura (15 marcadores ABRACS/RC/LGPD/NR-1/compliance)
- [[ONB-S5]] — Presença Profissional (só N2/N3, mini bio + vídeo institucional trava 90d)
- [[ONB-S6]] — Registros e Responsabilidade (3 aceites)
- [[ONB-SFIN]] — Confirmação Final

## Fluxo Empresa (7 telas + confirmação)

- [[ONB-E2]] — Identidade Institucional (CNPJ, razão social, representante)
- [[ONB-E3]] — Contatos e Organização (endereço, financeiro)
- [[ONB-E4]] — Atuação no Mercado (cidades, categoria, até 5 subcategorias)
- [[ONB-E5]] — Conformidade (25 marcadores: ISOs, ESG, selos, NRs, LGPD)
- [[ONB-E6]] — Conteúdo Institucional (descrição + vídeos + portfólio; sem conteúdo comercial)
- [[ONB-E7]] — Uso da Plataforma (2 aceites: Conteúdo Técnico + Connect Me)
- [[ONB-EFIN]] — Confirmação Final

## Fluxo Parceiro da Vizinhança (4 telas + confirmação)

- [[ONB-P2]] — Identidade do Estabelecimento (CNPJ OU CPF — D-064)
- [[ONB-P3]] — Contato e Visual (endereço, financeiro, até 3 fotos)
- [[ONB-P4]] — Aceites (Promoções + Código de Conduta)
- [[ONB-PFIN]] — Confirmação Final

---

## Fluxo macro

```
ONB-00 (seleção) → ONB-01 (comum)
                ↓
  ┌─────────────┼──────────────┐
  ↓             ↓              ↓             ↓
Morador     Síndico        Empresa       Parceiro
M2→M3→M4→   S2→S3→S4→S5→   E2→E3→E4→E5→  P2→P3→P4→
MFIN        S6→SFIN        E6→E7→EFIN    PFIN
```

Total contabilizado: 2 comuns + 5 morador + 7 síndico + 8 empresa + 5 parceiro = **27 entradas** (incluindo FIN). Contagem de telas "puras" por persona segue o PDF: 4/6/7/4.

## Regras transversais

- **Auto-save Redis**: chave `onboarding:{user_id}:{step}`, TTL 30 dias.
- **URL parametrizada (§2.5 excopo)**: `?role=X&plan=Y` pula [[ONB-00-boas-vindas]] e pré-seleciona.
- **Validação imutável** (IDN-029): CPF/CNPJ imutáveis após ativação.
- **Termos LGPD** versionados por hash + data (LGPD-### consent versioning).
- **Migração draft → persistência**: após [[ONB-MFIN]]/[[ONB-SFIN]]/[[ONB-EFIN]]/[[ONB-PFIN]], dados vão para `identity`, `institutional`, `commercial` BCs.
- **Zitadel** é IdP agnóstico (D-056); Zitadel envia e-mail de validação.

## Matriz de aceites

| Persona | Qtd aceites | Telas |
|---|---|---|
| Morador | 3 | [[ONB-M4]] |
| Síndico | 3 | [[ONB-S6]] |
| Empresa | 2 | [[ONB-E7]] |
| Parceiro | 2 | [[ONB-P4]] |

## Marcadores autodeclarados

| Persona | Qtd | Tela |
|---|---|---|
| Síndico | 15 | [[ONB-S4]] |
| Empresa | 25 | [[ONB-E5]] |

## Ligações externas

- [[../../../../00-product/personas]]
- [[../../../../04-requirements/registration-data]]
- [[../../../../04-requirements/compliance-lgpd]]
- [[../../../../client-canvas/Fluxo Registro]]
- [[../../../../_moc]]

## Gaps & pendências

- **Texto literal** dos termos por persona pendente de revisão jurídica Master Síndico.
- Linter de conteúdo comercial em [[ONB-E6]] é regex simples M1; ML/NLP M3+.
- N1 pode pular [[ONB-S5]] (presença profissional) — UX específica ainda a detalhar.
- Upload de evidências (certificados ISO, RG/CNH, CCMEI) para anti-fraude: backlog M3+.
- Dashboard pós-cadastro (empresa, parceiro) não especificado neste MOC.
- Admin **não** auto-registra (IDN-001) — sem fluxo aqui.
