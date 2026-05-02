---
title: Lei 14.063/2020 — Assinaturas eletrônicas detalhada (níveis + gov.br)
type: source
tags: [lei-14063, assinatura-digital, govbr, icp-brasil, validacao-juridica, br]
source: https://www.planalto.gov.br/ccivil_03/_ato2019-2022/2020/lei/l14063.htm
created: 2026-04-23
updated: 2026-04-23
aliases: [source-16-lei-14063, lei-14063-2020]
fontes_consultadas:
  - https://www.planalto.gov.br/ccivil_03/_ato2019-2022/2020/lei/l14063.htm
  - https://www.jusbrasil.com.br/topicos/336997151/artigo-4-da-lei-n-14063-de-23-de-setembro-de-2020
  - https://www.jusbrasil.com.br/topicos/336997161/artigo-3-da-lei-n-14063-de-23-de-setembro-de-2020
  - https://www.jusbrasil.com.br/artigos/assinaturas-eletronicas-e-digitais-no-brasil-plataforma-govbr-lei-14063-2020-e-validade-judicial/5827838853
  - https://blog.qualisign.com.br/assinatura-eletronica-e-a-lei-14063/
  - https://www.gov.br/governodigital/pt-br/identidade/assinatura-eletronica/saiba-mais-sobre-a-assinatura-eletronica
relevancia: alta
categoria: regulamentacao
---

# Lei 14.063/2020 — Assinaturas eletrônicas no Brasil

## Escopo original
- Regula uso de assinaturas eletrônicas em **interações com o Poder Público** e entre entes públicos.
- **Aplicação se estendeu** (jurisprudência + prática) para contratos privados em geral.

## Classificação de assinaturas (Art. 4º)

### I — Assinatura eletrônica simples
- Permite identificar o signatário (ex.: e-mail + senha, código SMS).
- Anexa ou associa dados a outros dados em formato eletrônico.
- **Nível de presunção mais baixo** — admite contestação fácil.

### II — Assinatura eletrônica avançada
- Utiliza certificados não emitidos pela ICP-Brasil OU outro meio de comprovação.
- Requer:
  - Estar **associada de modo unívoco ao signatário**.
  - Usar dados sob controle **exclusivo** do signatário.
  - Estar relacionada aos dados assinados de tal modo que qualquer alteração posterior seja **detectável**.
- Exemplo: **assinatura gov.br** (nível avançado, gratuita).

### III — Assinatura eletrônica qualificada
- Certificado digital ICP-Brasil.
- **Máximo nível de presunção legal** — equivalente a firma reconhecida.

## Níveis obrigatórios (Art. 5º)
- Atos perante Poder Público:
  - **Rotineiros**: simples.
  - **Risco moderado**: avançada (ou qualificada).
  - **Risco alto / transferência de imóvel / assinatura de grandes contratos públicos**: qualificada.

## Caso de uso em condomínio

### Nível simples (suficiente)
- Login de morador na plataforma.
- Confirmação de leitura de comunicado.
- Aprovação de reserva de salão.

### Nível avançado (recomendado)
- Voto em assembleia virtual.
- Assinatura da ata pelos participantes online (via gov.br ou provedor próprio).
- Aprovação de despesa extraordinária em assembleia.

### Nível qualificado (ICP-Brasil obrigatório)
- Registro de ata em **Cartório de Títulos e Documentos** ou **Registro de Imóveis**.
- Assinatura da **Convenção de Condomínio** para averbação.
- Contratos de grande porte (acima de certo valor, conforme convenção interna).

## Interação com gov.br

### Plataforma gov.br
- Nível de confiança: **prata** ou **ouro** (ouro = via biometria ou bancos conveniados).
- **Assinatura gov.br = avançada** — válida para maioria dos atos não-registrais.
- Gratuita para cidadão BR.

## Jurisprudência STJ (dez/2024)
- **Falta de credenciamento** da entidade certificadora na ICP-Brasil, por si só, **NÃO invalida** assinatura.
- O que conta: comprovação técnica de **autoria e integridade**.
- Reforça validade da assinatura avançada não-ICP.

## Relevância para Master Síndico

### Arquitetura de assinatura no Master Síndico
```
Nível simples → login, leitura, reservas simples
Nível avançado → voto em assembleia, ata da AGO
  ↳ integração gov.br (preferencial, gratuita, BR-nativo)
  ↳ alternativa: provider próprio com timestamping
Nível qualificado → registro cartorial de ata, convenção
  ↳ integração com provedores ICP-Brasil (Certisign, Soluti, ValidCertificadora)
```

### Diferencial competitivo
- TownSq, Superlógica, uCondo **não** oferecem integração gov.br documentada.
- Master Síndico pode ser **"gov.br-native"** = UX mais simples p/ morador BR que já tem conta no portal.
- **Carimbo do tempo** (RFC 3161 via provedor credenciado) + hash SHA-256 + armazenamento PDF/A = trilha auditável robusta.

### Compliance
- Cada ata deve salvar:
  - Nível de assinatura usado (simples/avançada/qualificada).
  - Provedor.
  - Timestamp.
  - Hash.
  - Log de auditoria (IP, user-agent, localização aproximada).
