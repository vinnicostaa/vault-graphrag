---
title: source-06 — Keeping LinkedIn professional, detecting inappropriate profiles
type: source
tags: [research, linkedin, trust-safety, fake-profiles, cnn, content-classification]
source: https://engineering.linkedin.com/blog/2020/keeping-linkedin-professional
created: 2026-04-23
updated: 2026-04-23
aliases: [linkedin-inappropriate-profiles, profile-classifier]
---

# source-06 — Keeping LinkedIn professional: inappropriate profile detection

**URL (atual, pós-redirect)**: https://www.linkedin.com/blog/engineering/trust-and-safety/keeping-linkedin-professional
**Data**: 2020
**Fonte primária**: sim (LinkedIn Engineering — Trust & Safety)

## Por que essa fonte importa

Caso concreto de **ML classifier pra conteúdo**, com discussão honesta de viés de training set. Contraponto útil pra decisão "não-ML" em M1/M2.

## Pontos extraídos

### Modelo
- **CNN text classifier** — boa em image **e** text por entender contexto.
- Exemplo do texto: palavra "escort" tem significado inofensivo em "security escort" vs inapropriado em outros contextos; CNN aprende o contexto circundante.

### Training set
- Features: conteúdo público de perfil (nome, headline, summary, etc).
- Labels binárias: inappropriate (contas removidas) vs appropriate (members ativos).
- Downsampling do universo de 660M+ members porque violações são raras (class imbalance).

### Problema crítico: viés do training set
- Labels "inappropriate" vieram de blocklist anterior (baseada em palavras-chave).
- Modelo corre risco de "apenas mimetizar o blocklist" → não aprende contexto, só memoriza palavras.
- Mitigação: **rotulagem manual de usos legítimos** de palavras problemáticas, adicionados ao training como contra-exemplos.

### Operação
- Model scora contas novas **diariamente** em produção.
- Roda retrospectivamente sobre member base existente.
- Roadmap: Microsoft translation pra suporte multilíngue.

## Aplicabilidade master-sindico

- **NÃO replicar ML classifier em M1/M2**. Universo de perfis é minúsculo (dezenas de milhares no máximo), revisão manual sustentável.
- **Aproveitar**: se houver blocklist de palavras proibidas, **documentar viés** — palavra banida não é mesma coisa que intenção maliciosa. "Eletricista" numa denúncia de assembleia não é spam ainda que apareça em 90% de mensagens automatizadas.
- **Padrão metodológico transferível**: "usar histórico de ações manuais como training é arriscado" — aplicável a qualquer automação futura que queiramos colocar em cima de decisões humanas (ex: aprovação automática de cadastro baseada em "padrões" de aprovação anterior).

## Anti-padrão observado (e a evitar)

- Automatizar com ML o que já está sendo decidido por humanos → perpetua viés humano em escala industrial. Sem auditoria robusta, compliance LGPD quebra.

## Quando releer

- Se volume de denúncias manuais ultrapassar capacidade (> 100/semana não resolvidas em 24h).
- Antes de qualquer automação de moderação de conteúdo.

## Vizinhos

- [[_destilado]]
- [[source-05]] — Defending Against Abuse (contexto infra)
