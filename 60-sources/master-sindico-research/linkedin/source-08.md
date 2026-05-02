---
title: source-08 — Building blocks of LinkedIn Skill Assessments
type: source
tags: [research, linkedin, skill-assessments, rasch-model, banco-talentos, adaptive-testing]
source: https://www.linkedin.com/blog/engineering/skills-graph/the-building-blocks-of-linkedin-skill-assessments
created: 2026-04-23
updated: 2026-04-23
aliases: [linkedin-skill-assessments, assessment-architecture]
---

# source-08 — LinkedIn Skill Assessments

**URL**: https://www.linkedin.com/blog/engineering/skills-graph/the-building-blocks-of-linkedin-skill-assessments
**Fonte primária**: sim (LinkedIn Engineering — Skills Graph)

## Por que essa fonte importa

Arquitetura completa de sistema de certificação de habilidade por assessment. Referência **quando** (M3+?) o Banco de Talentos morador for verificar habilidades.

## Pontos extraídos

### Conteúdo
- Questões criadas por "leading industry subject matter experts".
- Armazenadas em **Espresso database**.
- Conteúdo management platform original do LinkedIn Learning, reutilizado.
- Exposição via REST (rest.li framework).

### Calibração (Rasch Model)
- Modelo Rasch determina dificuldade de questão a partir de dados de resposta.
- 3 estágios:
  1. **Draft** — testes internos only.
  2. **Limited Ramp** — aparece pra members mas **não conta pontuação**; coleta dados de calibração.
  3. **Full Ramp** — totalmente calibrada, conta pro score.
- 3 workflows Hadoop: setup inicial, on-the-fly pra questões novas, recalibração periódica.

### Adaptive testing
- Dificuldade ajusta dinamicamente.
- Acertou → próxima mais difícil (vale mais).
- Errou → próxima mais fácil.

### Scoring
- Badge concedido se percentil ≥ 70.
- Badge exibido em perfil, alimenta **Galene search index via Kafka** → Recruiter search prioriza.

### Anti-cheat
- Retakes limitados.
- Copy (selecionar texto) desabilitado.
- Time limit por questão.
- Adaptive testing **também protege**: usuário que chuta errado fica preso em questões fáceis, nunca acessa conteúdo avançado (anti-scraping).

### Impacto medido
- ~30% improvement na probabilidade de receber resposta de recruiter se o candidato tem assessment passado.

## Aplicabilidade master-sindico

Detalhada em [[_destilado#6-skills-taxonomy--aplicável-ao-banco-de-talentos-morador]].

- **Skill Assessments provavelmente M3+** (não é M1).
- Quando entrar:
  - **NÃO implementar Rasch** inicialmente. Banco estático de questões + limite de tempo + percentil simples resolve 80%.
  - **Anti-cheat barato**:
    - Timer por questão ✅
    - Copy disabled ✅
    - Retake com cooldown (ex: 1 tentativa/mês por skill)
  - **Adaptive testing é M4+** — depende de calibração agregada que não existe no dia 1.
  - **Badge visível em busca**: quando morador passa em assessment de "elétrica residencial", síndico consegue filtrar Banco de Talentos por `verified:elétrica-residencial`. Outbox → OpenSearch update (mesmo padrão Kafka→Galene).
- **Verificação documental > Assessment**: em contexto BR, diploma Senac / CREA / certificação SINDUSCON vale muito mais que nosso quiz. Combinar: assessment como barreira de entrada + documento uploaded como prova forte.
- **Avaliação pós-contrato é prova mais forte ainda**: síndico marca "empresa fez bom trabalho de elétrica" depois de serviço real. Vale mais que qualquer assessment teórico.

## Quando releer

- Se Banco de Talentos for priorizado no roadmap.
- Quando decidir formato de "selo verificado" de morador.
- Se Connect Me abrir "verificação de skill" pra empresas (ao invés de só moradores).

## Vizinhos

- [[_destilado]]
- [[source-07]] — Skills taxonomy (fundação da taxonomy sobre a qual assessments rodam)
- [[source-10]] — Endorsements (mecanismo alternativo de "validação de skill")
