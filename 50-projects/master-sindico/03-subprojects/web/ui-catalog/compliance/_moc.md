---
title: "Compliance — MOC (C1-C11)"
type: moc
tags: [master-sindico, ui-catalog, web, compliance, sindico]
project: master-sindico
persona: sindico
sub_produto: compliance
status: specification
stack: web
created: 2026-04-23
updated: 2026-04-23
---

# Compliance (Síndico) — MOC

Map of Content do catálogo UI do módulo **Compliance** (11 telas C1-C11). Camada de blindagem documental, rastreabilidade e mitigação de risco jurídico, acessível via botão dedicado na Central de Governança.

## Princípio canônico
> "Compliance é a camada de proteção da sua gestão. Aqui ficam registradas declarações formais, responsabilidades assumidas, trilhas de auditoria e evidências institucionais."

O módulo **NÃO bloqueia** o cadastro inicial do condomínio, mas determinados marcos exigem compliance ativo (4 gatilhos canônicos abaixo).

## 4 Gatilhos obrigatórios (trava)
1. **Encerrar mandato** — exige declaração + assinaturas + dossiê.
2. **Gerar relatório final** — não emite sem compliance "Em dia".
3. **Marcar negócio fechado** — opcional, conforme política.
4. **Exportar dossiê** — obriga finalizar todas as pendências.

> Mensagem do bloqueio: "Para prosseguir, finalize as pendências de compliance."

## Telas
- [[C1]] — Painel de Compliance
- [[C2]] — Declaração Anual do Síndico (texto jurídico versionado, 1x/ano ou novo mandato)
- [[C3]] — Assinaturas Obrigatórias dos Usuários (até 3: principal, preposto, administradora, conselho)
- [[C4]] — Matriz de Conflito de Interesse
- [[C5]] — Trilha de Auditoria Visível (LGPD retenção 5 anos)
- [[C6]] — Imutabilidade e Adendos (único mecanismo de correção)
- [[C7]] — Mapa de Risco Consolidado
- [[C8]] — Conformidade Contratual
- [[C9]] — Registro LGPD (`lgpd_logs` central, retenção 5 anos)
- [[C10]] — Score Interno de Governança (privado)
- [[C11]] — Dossiê de Governança Exportável (PDF estruturado)

## Regras transversais
- **LGPD retenção 5 anos** — toda evidência (auditoria + lgpd_logs).
- **Adendo é único mecanismo de correção** — registros bloqueados (votações, negócios, declarações, eventos) não se editam; criam-se adendos formais com justificativa + descrição (voz/texto) + anexo opcional.
- **Imutabilidade**: declarações, assinaturas, matriz de conflitos, registros LGPD são imutáveis pós-confirmação.
- **Continuidade entre mandatos**: novo síndico assina nova declaração + atualiza conflito de interesse; histórico anterior preservado em modo leitura.

## Fontes canônicas
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Compliance-Detalhado.md` (C1-C11 com textos jurídicos literais e 4 gatilhos)
- `vault/90-archive/from-development-content-2026-04-23/contents/####################TELA DO MORADOR#####(1).md` (Bloco 18 + telas C1-C11)
- `vault/50-projects/master-sindico/00-product/sub-produtos/09-compliance.md`
- `vault/50-projects/master-sindico/03-subprojects/backend/.kiro/specs/master-sindico/requirements/cross-domain.md` (Req 33-42)
- `vault/50-projects/master-sindico/03-subprojects/backend/.kiro/STATE.md` DT-010 (`IAuditLogger` cross-módulo)

## Ligações
- [[../sindico/_moc]] — MOC catálogo síndico (governança principal)
- [[../../../../00-product/sub-produtos/09-compliance]]
- [[../../../../00-product/sub-produtos/01-governanca-institucional]]

## Gaps consolidados
- Score Governança 1-10 vs Score Compliance 0-100 (Q25 — divergência ####TELA####(1).ini × requirements.md Req 41).
- Endpoints inferidos para todos os fluxos (cliente não documentou rotas).
- Provedor de assinatura digital ICP-Brasil (MP 2.200-2) sem decisão (Sprint 3+, M4+).
- Política de archive Glacier após 1 ano sem ADR formal.
