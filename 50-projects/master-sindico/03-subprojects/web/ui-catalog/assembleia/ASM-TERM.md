---
title: ASM-TERM — Termos Obrigatórios Pré-Assembleia
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: morador
category: ASM
screen_id: ASM-TERM
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-TERM — Termos Obrigatórios da Pré-Assembleia

## Finalidade

Componente/tela (normalmente modal concatenado a [[ASM-CIEN]]) com os **5 termos obrigatórios** que o morador deve aceitar antes de concluir ciência e/ou confirmar participação. Isenta parcialmente a Master Síndico quanto à verificação material da identidade do votante.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §4.17 (Termos obrigatórios na pré-assembleia)
- [[../../../../04-requirements/compliance-lgpd]]
- [[../../../../04-requirements/functional/assembly]] Req 58

## Persona e ABAC

- **Primária**: `resident`.
- ABAC: `assembly.terms.accept` (pós-login, antes de qualquer ação).

## Fluxo da tela

1. Modal full-width com 5 blocos expansíveis:
   - **Termo de ciência da convocação**
   - **Termo LGPD** (referencia política geral da plataforma)
   - **Declaração de veracidade das informações prestadas**
   - **Declaração de identidade e legitimidade de uso do acesso da unidade**
   - **Autorização de gravação** (checkbox condicional — só se assembleia for gravada por terceiros)
2. Cada bloco tem texto oficial + checkbox de aceite.
3. Botão final **[Aceito todos]** habilita quando todos checkboxes estão marcados.
4. Registro gera entradas em `audit_trail` (`terms.accepted`) e `lgpd_logs` (LGPD Art. 8º consentimento).

## Componentes

- Accordion expansível (shadcn/solid-ui).
- Textos oficiais em `<article>` com tipografia legal.
- Versão de termo (hash + data) armazenada para trilha.

## Estados (loading/empty/error/success)

- **Error**: algum termo não aceito → botão permanece desabilitado + tooltip.
- **Success**: fecha modal + prossegue fluxo.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Aceitar | `/api/v1/assemblies/:id/terms/accept` | POST | `{versions:{cience,lgpd,veracity,identity,recording}}` | `{accepted_at}` |
| Ler versões atuais | `/api/v1/assemblies/:id/terms` | GET | — | `{texts, versions}` |

## Regras de negócio críticas

- **Texto literal** da declaração de veracidade (§4.17): _"Declaro, sob minha responsabilidade, que participo desta assembleia na condição legítima vinculada à unidade informada e que as informações prestadas são verdadeiras."_
- Versões de termo versionadas (hash) — mudança de texto requer re-aceite (LGPD-### consent versioning).
- Retenção `lgpd_logs`: 5 anos + purga pós-vencimento (INV-091).
- Aceite é pré-requisito para [[ASM-CIEN]] e [[ASM-CHKIN]].

## Ligações

- Origem: [[ASM-CIEN]].
- Destino: [[ASM-CHKIN]], [[ASM-VOTO]].
- Relacionados: [[../../../../04-requirements/compliance-lgpd]], [[../../../admin-privilegios]] §7.4.

## Gaps/ressalvas

- Texto completo de cada termo (não apenas nome) precisa ser finalizado com jurídico Master Síndico — stubs em M1.
- Checkbox de gravação condicional: UX para casos mistos (assembleia híbrida sem gravação) ainda definir.

## Links

- [[_moc]]
- [[ASM-CIEN]]
- [[ASM-CHKIN]]
