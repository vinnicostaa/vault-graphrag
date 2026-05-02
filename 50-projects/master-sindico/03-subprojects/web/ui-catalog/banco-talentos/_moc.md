---
title: "Banco de Talentos — MOC (BT01-BT11)"
type: moc
tags: [master-sindico, ui-catalog, web, banco-de-talentos, morador]
project: master-sindico
persona: morador
sub_produto: banco-de-talentos
status: specification
stack: web
created: 2026-04-23
updated: 2026-04-23
---

# Banco de Talentos — MOC (cadastro morador)

Map of Content do catálogo UI do **Banco de Talentos — fluxo de cadastro do morador (BT01-BT11)**. Vídeo-currículo de **90 segundos** + ficha estruturada (18 áreas condominiais).

> "Sua carreira condominial em 90 segundos. As empresas não procuram apenas experiência. Elas procuram pessoas que combinem com o ambiente de trabalho."

## Quem publica · quem busca
- **Publica**: Morador Pagante (1 currículo por morador, INV-BT-05).
- **Busca**: Empresa Plus + Empresa Pro (Pro com analytics/auto-tags/exportação PDF/relatórios).
- **Modera**: Admin Master Síndico (D-058 transversal).
- **Sem acesso**: Morador Base, Síndico, Empresa Trial, Agência Marketing, Parceiro Vizinhança.

## Telas (cadastro do morador — fluxo)
- [[BT01]] Boas-vindas
- [[BT02]] Vídeo de Apresentação (60-90s — divergência cliente, ver Q39 análoga)
- [[BT03]] Perfil Profissional (6 perguntas abertas)
- [[BT04]] Áreas de Interesse (18 condominiais + função desejada)
- [[BT05]] Dados Pessoais
- [[BT06]] Disponibilidade e Contratação
- [[BT07]] Pretensão Salarial
- [[BT08]] Experiência Profissional (**5 vínculos — D-099 Fase 11 fecha Q39 na direção do PDF original**)
- [[BT09]] Formação + CNH + NR (Autorrelato)
- [[BT10]] Consentimento LGPD (versionado)
- [[BT11]] Confirmação

## Regras críticas
1. **Vídeo-currículo até 90 segundos** — DB CHECK `duration_seconds <= 90`. Cliente PDF original "60s"; Decision do client-vault elevou para 90s.
2. **Lock 90 dias pós-upload do vídeo** — re-upload bloqueado (`locked_until = created_at + 90 days`); compromisso anti-fraude/anti-spam.
3. **Empresa NÃO vê contato direto** do candidato — só por preenchimento de formulário; plataforma envia notificação por email/SMS ao candidato (LGPD).
4. **5 vínculos profissionais máx** (D-099 Fase 11 STATE v2 — sobrescreve D-060/D-064; Q39 fechada na direção do PDF original do cliente).
5. **Consentimento LGPD obrigatório e versionado** — sem aceite, currículo fica em `draft`. Mudança de termo obriga reconsent.
6. **1 currículo por morador** — `UNIQUE(morador_id)` em `curricula`.
7. **Sem endorsements, sem likes públicos** — não existem tabelas; anti-fraude estrutural.
8. **Toda busca empresa logada** em `curriculum_search_logs` (LGPD append-only).
9. **Privacidade cross-tenant** — favoritos/tags privados por empresa; A não vê o que B fez.
10. **Tags só de campos estruturados** (Fase 1) — sem NLP em respostas abertas.

## Estados do Curriculum
`draft → submitted → published → archived` (transições controladas; admin aprova `submitted → published`).

## Fontes canônicas
- `vault/90-archive/master-sindico-pdfs-cliente-originais/ESTRUTURA DE CADASTRO DE CURRÍCULO.md`
- `vault/90-archive/master-sindico-pdfs-cliente-originais/COMO A EMPRESA VÊ O CURRÍCULO DO MORADOR (1).md`
- `vault/50-projects/master-sindico/00-product/sub-produtos/07-banco-de-talentos.md`
- `master-sindico-v2/STATE.md` **D-099 Fase 11** (5 vínculos — sobrescreve D-060/D-064 que diziam 3).

## Ligações
- [[../../../../00-product/sub-produtos/07-banco-de-talentos]]
- [[../../../../00-product/sub-produtos/02-connect-me]] (fluxo especial Empresa→Morador BT)
- [[../../../../00-product/sub-produtos/04-conteudo-videos]] (infra Mux + lock 90d)
- [[../morador/_moc]]

## Gaps consolidados
- ~~Q39: 3 vs 5 vínculos~~ **FECHADA 2026-04-24 via D-099 Fase 11 → 5 vínculos** (alinha PDF original do cliente; BT08 atualizada).
- Vídeo 60s (PDF) vs 90s (sub-produto destilado) — nesta task adoto 90s do canônico v2.
- 6 perguntas abertas BT03 — catálogo definitivo é seed (G-BT-02).
- 7 auto-tags Pro — lista literal pendente (G-BT-03).
- Quota Connect Me Empresa→Morador BT (G-BT-04).
- Política retenção LGPD pós-archived (G-BT-06).
- Telas Admin de moderação BT — não catalogadas (G-BT-12).
