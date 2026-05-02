---
title: source-07 — Building and maintaining the LinkedIn Skills Graph taxonomy
type: source
tags: [research, linkedin, skills-graph, taxonomy, banco-talentos]
source: https://www.engineering.fyi/article/building-and-maintaining-the-skills-taxonomy-that-powers-linkedin-s-skills-graph
created: 2026-04-23
updated: 2026-04-23
aliases: [linkedin-skills-taxonomy, skills-graph]
---

# source-07 — LinkedIn Skills Graph taxonomy

**URL (mirror)**: https://www.engineering.fyi/article/building-and-maintaining-the-skills-taxonomy-that-powers-linkedin-s-skills-graph
**Complementar**: https://engineering.linkedin.com/blog/2023/extracting-skills-from-content-to-fuel-the-linkedin-skills-graph
**Data**: 2023
**Fonte primária**: sim (mirror autenticado do engineering blog)

## Por que essa fonte importa

Referência direta pra construção do **Banco de Talentos morador** — como estruturar universo de skills com aliases, hierarquia, e manutenção sustentável.

## Pontos extraídos

### Escala do universo
- ~39.000 skills.
- ~374.000 aliases.
- ~200.000 conexões entre skills (skill↔skill semelhança, hierarquia, co-ocorrência).
- Crescimento ~35% desde 2021 (só 2 anos).

### Manutenção
- **Dual approach**: taxonomistas humanos + ML.
- Humanos curam, ML sugere candidatos e valida relevância.
- Mantém taxonomy "relevant and high-quality".

### Papel no produto
- Foundational vocabulary pro Skills Graph.
- Alimenta: Recruiter (busca), LinkedIn Learning (recomendação), Jobs (matching).

### Skills de content (artigo complementar)
- ML extrai skills de posts, articles, CVs, job descriptions.
- NLP + embeddings pra mapear texto livre → skill_id canônico da taxonomy.

## Aplicabilidade master-sindico

Detalhada em [[_destilado#6-skills-taxonomy--aplicável-ao-banco-de-talentos-morador]].

- **Taxonomy curada** (sem NLP auto-extract em M1/M2) — universo estimado 200-500 skills BR pra Banco de Talentos morador.
- **Aliases obrigatórios**: "eletricista" ↔ "eletricidade residencial" ↔ "instalação elétrica" = 1 skill_id. Evita fragmentação do índice de busca (que o síndico usaria pra filtrar).
- **Hierarquia rasa em M1**: categoria → skill. Ex: "Manutenção" → {"elétrica residencial", "hidráulica", "pintura"}. Não reinventar ontologia profunda.
- **Curadoria humana** — time de produto revisa sugestões vindas de moradores que queiram adicionar skill ausente. Fluxo "sugerir → revisão semanal → aprovar" basta.
- **ML extraction de skills**: **fora de escopo** até provar necessidade. Morador marca skills explicitamente via UI; suficiente.

## Lições transferíveis

1. **Aliases evitam fragmentação** — universal, aplica mesmo em taxonomies pequenas.
2. **Human+ML não é "ML everywhere"** — é delegação: humano decide what, ML faz scale. Em master-sindico, escala pequena → humano faz tudo por enquanto.
3. **Taxonomy serve múltiplos produtos** — Recruiter, Learning, Jobs no LinkedIn. Banco de Talentos, Connect Me (busca de serviço) e Reputação (selos de skill verificada) no master-sindico.

## Quando releer

- Quando Banco de Talentos entrar em roadmap ativo.
- Quando universo de skills passar de 500 (provável ≥ 2027).
- Quando precisarmos conectar skills de morador com skills de empresa (Connect Me) pra cross-recommendation.

## Vizinhos

- [[_destilado]]
- [[source-08]] — Skill Assessments (sobre mesma taxonomy)
- [[../../03-subprojects/banco-talentos]] (quando existir)
