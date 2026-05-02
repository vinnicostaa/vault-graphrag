---
title: Vision — Master Síndico
type: project
tags: [master-sindico, product, vision]
project: master-sindico
created: 2026-04-22
---

# Vision — Master Síndico

## Propósito

Master Síndico é uma **plataforma SaaS de governança condominial** que conecta síndicos, moradores, empresas do mercado condominial, agências de marketing e comércio local de um condomínio.

> **Não é** ERP, app de comunicação interna, sistema financeiro. **É** governança aplicada.

## O que resolve

**Problema**: gestão condominial no Brasil é fragmentada, sem critério na contratação de serviços, sem memória institucional auditável, e o síndico — em sua maioria leigo — toma decisões sem apoio estruturado.

**Solução**:
1. **Critério na contratação** via sistema de reputação auditável (Bronze → Diamante)
2. **Memória institucional** via timeline imutável do condomínio
3. **Ecossistema unificado** de perfis, vídeos institucionais, cursos, contato direto entre todos os agentes
4. **Assembleia digital** com voto rastreável, ata imutável, fila de fala gerenciada

## Personas

Ver [[personas]]. Resumo:

- **Síndico** — gere o condomínio; principal decisor; quer critério, histórico, transparência
- **Morador** — contrata a plataforma via condomínio; quer saber o que acontece + acessar serviços
- **Empresa condominial** — vende serviços; quer ter vitrine e reputação construída
- **Agência de marketing** — auxilia empresas a se posicionarem
- **Parceiro de comércio local** — quer vínculo com condomínios da vizinhança

## North-star metrics (6 meses)

1. **Condomínios ativos** — pagantes, com plano diretor cadastrado, ≥ 1 assembleia realizada na plataforma
   - Meta M3 (2026-07): 30 condomínios ativos
2. **Connect Me fechados / mês** — contratos com trilha de auditoria completa
   - Meta M3: 100 / mês
3. **Assembleias realizadas / mês** — com quórum atingido, ata publicada
   - Meta M3: 20 / mês
4. **NPS síndicos** — intenção de recomendar
   - Meta M3: ≥ 50

## Invariantes do produto

1. **Identidade única** — 1 CPF = 1 usuário. Contextos de operação (síndico@X, morador@Y, empresa) são **múltiplos**.
2. **Memória imutável** — timeline do condomínio nunca edita ou apaga; corrige via entry nova.
3. **Reputação determinística** — Bronze→Diamante é função de métricas auditáveis, nunca input humano direto.
4. **Trial por persona** — não global. Síndico 15d; Empresa 7d; Parceiro 30d (pode revisar).
5. **Privacidade de métricas** — views, buscas, comportamento de um tenant **não** vazam pra outro.
6. **1 dispositivo ativo** por conta (configurável via env, mas default é 1).
7. **Governança antes de operação** — Master Síndico oferece **critério**, não **operação** (não é ERP).

## Fora de escopo (guard contra scope-creep)

❌ Cobrança condominial (taxas, boleto, inadimplência)
❌ ERP financeiro (prestação de contas detalhada, orçamento)
❌ App de chat tempo real (clientes usam WhatsApp/similar)
❌ Emissão fiscal (NF-e, NFS-e)
❌ Terceirização de portaria/segurança
❌ Internacional (foco Brasil; i18n prep em M3)

Se alguém pedir algo dessa lista: **empurrar back** ou marcar como fora de escopo.

## Stakeholders

| Papel | Pessoa / role | Envolvimento |
|---|---|---|
| Product Owner | João (cliente/dev) | Todas as decisões de produto |
| Eng Lead | João + agente | Arquitetura + execução |
| DPO LGPD | a definir | Compliance |
| Sales (futuro) | a contratar pós-M3 | Pipeline de condomínios |

## Princípios de produto

1. **Clareza > Cobertura** — é melhor fazer 5 coisas incríveis do que 50 ok
2. **Auditável por padrão** — tudo que importa deixa trilha
3. **Privacidade default ON** — ofertar dados só com opt-in
4. **Pt-BR first** — linguagem ubíqua do condomínio brasileiro
5. **Mobile-first** em telas de morador; web-first em telas de síndico/empresa

## Roadmap de alto nível

Ver [[../ROADMAP]]. M1 (2026-05-08) → M2 (2026-06-20) → M3 (2026-07-20).

## Links

- [[_moc]]
- [[personas]]
- [[business-model]]
- [[glossary]]
- [[../CLAUDE]]
- [[../ROADMAP]]
- [[../STEERING]]
- [[../client-vault/00 - Visão Geral/00 - Índice Master Síndico]]
