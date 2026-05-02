---
title: Assinatura eletrônica — MP 2.200-2/2001 + Lei 14.063/2020 (visão geral)
type: source
tags: [assinatura-digital, icp-brasil, mp-2200-2, lei-14063, ata, br]
source: https://www.planalto.gov.br/ccivil_03/mpv/antigas_2001/2200-2.htm
created: 2026-04-23
updated: 2026-04-23
aliases: [source-11-assinatura-eletronica, mp-2200-2, icp-brasil]
fontes_consultadas:
  - https://www.planalto.gov.br/ccivil_03/mpv/antigas_2001/2200-2.htm
  - https://www.planalto.gov.br/ccivil_03/_ato2019-2022/2020/lei/l14063.htm
  - https://www.stj.jus.br/sites/portalp/Paginas/Comunicacao/Noticias/2024/03122024-Falta-de-credenciamento-da-entidade-certificadora-na-ICP-Brasil--por-si-so--nao-invalida-assinatura-eletronica-.aspx
  - https://www.gov.br/governodigital/pt-br/identidade/assinatura-eletronica/saiba-mais-sobre-a-assinatura-eletronica
  - https://certifica.com.br/blog/validade-juridica-do-certificado-digital/
  - https://certifica.com.br/blog/assinatura-digital/
relevancia: critica
categoria: regulamentacao
---

# Assinatura eletrônica — MP 2.200-2/2001 + Lei 14.063/2020

## Hierarquia legal
### MP 2.200-2/2001 (vigente por EC 32)
- Instituiu **ICP-Brasil** (Infraestrutura de Chaves Públicas).
- AC Raiz + ACs + ARs encadeados.
- **Art. 10**: documento eletrônico com certificado ICP-Brasil tem **presunção de veracidade equivalente** à assinatura manuscrita reconhecida em cartório.

### Lei 14.063/2020
- Classifica assinaturas em **3 níveis**:
  - **Simples**: identifica o signatário (nível mais baixo — ex.: e-mail + senha).
  - **Avançada**: usa certificados não-ICP-Brasil, mas com controles técnicos (chave privada sob controle exclusivo, detecção de alteração).
  - **Qualificada**: certificado ICP-Brasil (máxima presunção legal).
- Validade **não depende** de ICP-Brasil desde que autoria + integridade estejam garantidas (art. 4º II).

### Jurisprudência STJ 2024
- **Decisão relevante**: falta de credenciamento da entidade certificadora na ICP-Brasil **NÃO invalida** automaticamente a assinatura eletrônica. Conta a comprovação técnica de autoria e integridade.

## Aplicação em condomínio
### Convenção de condomínio
- Pode ser averbada em cartório. Cartório tipicamente exige **assinatura qualificada ICP-Brasil** para registro imobiliário.

### Ata de assembleia
- Lei 14.309/2022 + Código Civil + MP 2.200-2 dão base legal para **ata eletrônica assinada digitalmente** ter mesma validade da física.
- **Na prática**:
  - Síndico + participantes online: assinatura avançada ou qualificada (ICP-Brasil preferencial p/ registrar em cartório).
  - Moradores ausentes com procuração: procuração eletrônica válida.
  - **Carimbo do tempo** (time-stamping) recomendado p/ provar momento da assinatura.

### Edital de convocação
- Pode ser totalmente digital (Lei 14.309/2022).
- Assinatura simples geralmente suficiente (autoridade do síndico na convocação é presumida pela convenção).

## Relevância para Master Síndico
- **Módulo assembleia live** deve oferecer:
  - Assinatura **avançada** por padrão (mais simples, custo baixo).
  - **Opcional ICP-Brasil** p/ condomínios que vão registrar ata em cartório.
  - **Carimbo do tempo** (timestamping via provedor credenciado).
  - Trilha de auditoria: quem assinou, quando, qual IP, com qual certificado.
- Integrar com **gov.br** (assinatura gov.br é nível avançado, gratuita, amplamente usada no BR) = diferencial de UX para moradores.
- Armazenar ata em formato PDF/A (arquivamento de longo prazo).
- Permitir exportação em formato aceito pelo cartório (PDF + p7s / PAdES).
