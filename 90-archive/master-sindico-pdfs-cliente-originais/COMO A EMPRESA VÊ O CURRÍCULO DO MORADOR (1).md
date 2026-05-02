---
title: COMO A EMPRESA VÊ O CURRÍCULO DO MORADOR (1) (PDF do cliente — texto extraído)
type: note
tags: [archive, master-sindico, client-pdf, extracted-text]
source: Clients/MasterSindico/contents/COMO A EMPRESA VÊ O CURRÍCULO DO MORADOR (1).pdf
created: 2026-04-24
---

# COMO A EMPRESA VÊ O CURRÍCULO DO MORADOR (1)

Texto extraído do PDF original do cliente João via `pdftotext -layout`. **Fidelidade limitada** — se PDF é majoritariamente imagem/diagrama visual, o texto extraído pode ser incompleto.

- **PDF original**: 3961859 bytes
- **Texto extraído**: 4487 caracteres
- **Origem**: `~/Documentos/Obsidian Vault/Clients/MasterSindico/contents/COMO A EMPRESA VÊ O CURRÍCULO DO MORADOR (1).pdf`

## Texto extraído

```text
               COMO A EMPRESA VÊ O CURRÍCULO DO MORADOR

 Consideração importante: a empresa não terá acesso aos dados de contato
   do dono do currículo, apenas por preenchimento de formulário, onde
           enviaremos notificação via e-mail e se possível sms

1) Exportar em PDF (obrigatório)

Onde aparece: botão “Exportar PDF” na tela do candidato (e opcionalmente no
card da lista).

Conteúdo do PDF (1–2 páginas, layout simples):

   •   Cabeçalho: Master Síndico + data/hora da exportação + empresa que
       exportou

   •   Dados do candidato: Nome, bairro,

   •   Áreas de interesse + subárea/cargo desejado (texto do candidato)

   •   Disponibilidade + modalidades + início + deslocamento

   •   Pretensão salarial (mín/ideal)

   •   Formação e cursos

   •   Experiência (últimos 3 vínculos)

   •   Perguntas abertas (6) – em blocos

   •   Link/QR do vídeo (em vez de embutir vídeo)

Implementação leve: HTML → PDF (template único). Nada de design complexo.



2) Favoritos

   •      no card e no perfil

   •   Tela “Meus Favoritos” com os mesmos filtros da lista

Tabela simples: favorites(company_id, candidate_id, created_at)



3) Relatórios de busca (sem BI pesado)

A ideia é “analytics útil”, não dashboard complexo.

Relatório 1: Histórico de buscas

   •   Data/hora
   •   Filtros usados (json)

   •   Total de resultados retornados

   •   Quantos perfis completos no resultado

   •   Botão “Reaplicar busca”

Relatório 2: Funil simples da empresa (ações)

   •   Visualizações de perfis (quantas vezes abriu um candidato)

   •   Favoritados

   •   PDFs exportados

   Tudo isso é só “log”.

Tabelas:

   •   search_logs(company_id, created_at, filters_json, results_count,
       complete_count)

   •   candidate_views(company_id, candidate_id, created_at)

   •   pdf_exports(company_id, candidate_id, created_at)



Tags classificatórias na Fase 1 (quais tags?)

   Regra: tags só podem vir de campos estruturados
(checkbox/dropdown/número)
   Nada de “ler as respostas abertas” (isso é IA/NLP e atrasa)

Vou te dar 2 tipos de tags:

A) Tags automáticas (100% por regra) — recomendadas

Essas tags aparecem na lista e no perfil, e ajudam a filtrar.

1) Status do perfil

   •   Perfil completo (tem vídeo + 6 respostas + área selecionada)

   •   Faltando vídeo

   •   Faltando respostas

   •   Faltando área de interesse

2) Disponibilidade

   •   Diurno
   •   Noturno

   •   Escala

   •   Finais de semana

   •   Início imediato

   •   Início em até 15 dias

3) Contratação

   •   Aceita CLT

   •   Aceita Temporário

   •   Aceita Folguista

   •   Aceita Diarista

   •   Aceita Freelancer

   •   ...

4) Deslocamento

   •   Deslocamento até 30 min

   •   Deslocamento até 1h

   •   Deslocamento até 1h30

5) Pretensão salarial
Pretensão informada

   •   Pretensão não informada

   •   Faixa até R$ X / Faixa R$ X–Y / Acima de R$ Y
       (X/Y definidos pela empresa no filtro, não pela plataforma)

6) Formação

   •   Ensino médio completo

   •   Técnico

   •   Superior

   •   Cursos NR informados

7) Experiência

   •   Experiência informada

   •   Sem experiência informada
   •   Mais de 2 anos no último vínculo (regra baseada no tempo preenchido)

Essas tags já resolvem 80% do filtro real do síndico/empresa.



B) Tags manuais (curadoria da empresa) — MUITO valiosas e fáceis

A empresa marca na própria triagem (sem algoritmo).

Sugestões de tags manuais (curtas e úteis):

   •   Boa comunicação

   •   Postura excelente

   •   Pontualidade (referida)

   •   Perfil calmo

   •   Perfil proativo

   •   Precisa supervisão

   •   Atenção a detalhes

   •   Bom para rotina

   •   Bom com público

   •   A avaliar em entrevista

   •   CNH

   •   NR

   Implementação leve: checklist de tags + “criar tag” (opcional).
Tabela: company_tags(id, company_id, name) e
candidate_company_tags(company_id, candidate_id, tag_id)



Como aparece na tela (bem simples)

No card do candidato, mostrar no máximo 4 tags:

   •   1 tag de status (Perfil completo / Faltando vídeo)

   •   1–2 de disponibilidade (Diurno / Escala)

   •   1 de contratação (Aceita CLT)

   •   1 de deslocamento (até 1h)

O resto fica no perfil completo.

```

## Links

- [[_moc]]
