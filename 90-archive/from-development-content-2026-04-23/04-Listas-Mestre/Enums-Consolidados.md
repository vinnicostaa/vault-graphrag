---
title: "Enums e Listas Mestre — Consolidação Completa"
updated: 2026-04-22
maintainer: agent-3 (analista)
type: catalog
status: stable
source: requirements.md (monolítico) seção "Enums and Master Lists" + ####TELA####(1).ini Bloco 5
---

# Enums e Listas Mestre — Master Síndico

Consolidação fiel de **todos os enums** definidos no `requirements.md` (chunks 2900-3800) + Bloco 5 do `####TELA####(1).ini`. Esta é a fonte de domínio para `domain/enums/` em cada bounded context Go.

> **Convenção do projeto**: enums no banco são `lowercase_snake_case`; em pt-BR quando referem ao domínio brasileiro (categorias, tipos), em inglês quando referem ao tecnológico (status, system).

---

## 1. Identidade e Acesso

### User Roles (T7 — atual: inglês; original: pt-BR — Q19/Q20)

**Atual (Zitadel + PG)**:
```
syndic | resident | enterprise | marketing | local_company | admin
```

**Role Types (sub-contexto operacional, só no banco — T8)**:
```
principal | auxiliary | assistant     # síndico
titular   | dependent                 # morador
administrator | operator              # empresa
```

### User Status
```
pending_verification | active | suspended | deleted
```

### Plans
```
trial | base | plus | pro
```
- Trial: 15 dias Síndico, 7 dias Empresa, 30 dias Parceiro

---

## 2. Condomínio

### Tipos de Condomínio (6)
```
residencial | comercial | misto | shopping_center | galeria_comercial | edificio_garagem
```

### Tipos de Mandato (3)
```
sindico_eleito | sindico_profissional | sindico_interino
```

### Motivos de Encerramento de Mandato (4)
```
fim_periodo | renuncia | destituicao | transferencia
```

### Status do Mandato
```
ativo | encerrado | transferido
```

---

## 3. Unidade e Vínculo

### Tipos de Unidade (3)
```
residencial | comercial | vaga_garagem
```

### Tipos de Vínculo do Morador (3)
```
proprietario_residente | proprietario_nao_residente | inquilino
```

### Status da Unidade (3)
```
ocupada_proprietario | ocupada_inquilino | unidade_vazia
```

### Roles do Morador
```
titular | dependente
```

### Motivos de Encerramento de Vínculo (4)
```
mudanca_definitiva | venda_unidade | encerramento_contrato_locacao | erro_cadastro
```

---

## 4. Plano Diretor / Atividades

### Áreas Impactadas (26)
```
estrutura_predial · sistema_hidraulico · sistema_eletrico · sistema_incendio
reservatorios · elevadores · garagem · fachada · cobertura · seguranca_patrimonial
administracao · portaria · playground · academia · salao_festas · corredores
jardins · casa_maquinas · rede_esgoto · rede_drenagem · sistema_gas · iluminacao
sistema_cameras · controle_acesso · areas_comuns · outros
```

### Tipos de Atividade (26)
```
manutencao_preventiva · manutencao_corretiva · inspecao_tecnica · vistoria_tecnica
contratacao_servico · execucao_servico · reparo_emergencial · obra_melhoria
adequacao_normativa · atualizacao_infraestrutura · monitoramento_ambiental
revisao_tecnica · auditoria_tecnica · limpeza_tecnica · controle_pragas
limpeza_reservatorios · treinamento_tecnico · adequacao_legal · revisao_equipamentos
atualizacao_administrativa · contratacao_fornecedor · encerramento_contrato
avaliacao_fornecedor · implantacao_sistema · atualizacao_normas · acao_emergencial
```

### Risco Associado (9)
```
sem_risco | risco_operacional | risco_estrutural | risco_sanitario
risco_eletrico | risco_hidraulico | risco_incendio | risco_juridico | risco_ambiental
```

### Próxima Ação (8)
```
monitorar_evolucao | nova_inspecao | contratar_fornecedor | executar_manutencao
atualizar_plano_diretor | registrar_conclusao | acompanhar_garantia | reavaliar_necessidade
```

### Status do Plano Diretor (7)
```
planejada | em_contratacao | em_execucao | concluida | suspensa | reprogramada | atrasada
```

### Nível de Importância (5)
```
baixa | moderada | alta | critica | emergencial
```

> ⚠️ ####TELA####(1).ini Bloco 5.5 lista alternativa: `Baixa, Moderada, Alta, Crítica, Estratégica` — diferente da matriz Mar/2026 (`emergencial`). Adotar `requirements.md` (canônico): `baixa | moderada | alta | critica | emergencial`.

### Impacto Esperado
```
reducao_risco · adequacao_legal · melhoria_infraestrutura · preservacao_patrimonial
aumento_seguranca · melhoria_operacional · economia_financeira · melhoria_qualidade_vida
```

> ####TELA####(1).ini lista 11 itens, requirements.md lista 8. Cardinalidade divergente. Adotar requirements.md (canônico).

---

## 5. Timeline

### Tipos de Entrada (7)
```
atividade_da_gestao
registro_de_execucao
comunicado
evento
adendo
resultado_consulta_moradores
assembly_result
```

### Visibilidade
```
gestao | publico_moradores
```

---

## 6. Eventos do Condomínio

### Tipos de Evento (13)
```
assembleia_ordinaria · assembleia_extraordinaria · manutencao_programada
inspecao_tecnica · obra_programada · interrupcao_programada_servicos
treinamento_equipe · campanha_institucional · evento_comunitario
reuniao_administrativa · atualizacao_normativa · simulado_emergencia · outros
```

### Status do Evento
```
scheduled | in_progress | completed | cancelled
```

### Confirmação de Participação (Morador)
```
ciente | confirmado | nao_confirmado
```

---

## 7. Comunicados

### Tipos (8 — canônico requirements.md)
```
aviso_operacional · interrupcao_programada · orientacao_moradores · aviso_seguranca
comunicado_institucional · conclusao_servico · recomendacao_manutencao · alerta_preventivo
```

> ⚠️ ####TELA####(1).ini lista tipos adicionais (Aviso de início de serviço, Comunicado de intervenção/obra, Circular informativa, Comunicado de interdição temporária, Comunicado de conclusão). Ver Q24 em [[../regras-de-negocio/quebras-detectadas]]. Adotar 8 canônicos.

### Tipos pós-deal (5)
```
orientacao_tecnica · recomendacao_manutencao · alerta_preventivo
conclusao_servico · atualizacao_garantia
```

### Prioridade
```
low | medium | high | urgent
```

---

## 8. Consultas aos Moradores / Enquetes

### Tipos (3)
```
sim_nao_nao_sei | multiple_choice | scale_1_to_5
```

> Para enquetes preliminares de assembleia: **apenas sim_nao** (Req 51).

---

## 9. Assembleia

### Tipos (3 + 1 implantação)
```
ordinaria | extraordinaria | implantacao
```

### Status (5)
```
em_preparacao | publicada | em_andamento | encerrada | cancelada
```

> requirements.md tem alternativo: `scheduled | published | in_progress | closed | homologated` — adoção em pt-BR (em_preparacao etc) é mais próxima dos PDFs e UI.

### Modalidade
```
presencial | hibrida | online
```
> No MVP: apenas `presencial`. `hibrida`/`online` preparadas mas inativas.

### Tipos de Item da Pauta (5)
```
informativo | votacao_simples | votacao_fracao_ideal | escolha_fornecedor | eleicao
```

### Status do Item
```
resolvido | adiado | nao_resolvido | prejudicado
```

### Tipos de Quórum (3)
```
simple_majority | qualified_majority | unanimous
```

### Vote Options
```
favor | contra | abstencao         # binário
escala_1_a_5 + abstencao            # escalada
```

### Vote Origin
```
app | presencial_assistido | proxy
```

### Check-in Method (3)
```
app | qr_code | terminal_presencial
```

### Attendance Status
```
presente | ausente_momentaneo | desconectado_saiu
```

### Confirmação de Participação (Morador, pré-assembleia)
```
participarei_presencialmente | participarei_online | ainda_nao_defini | nao_participarei
```

### Speech Time (2)
```
2_minutos | 3_minutos
```

### Inadimplência
```
apto_a_votar | inapto_a_votar
```

### Modo Contingência — Razões
```
internet_failure | device_failure | digital_voting_unavailable
```

---

## 10. Commercial — Connect Me

### Tipos (4 direções)
```
sindico_to_company | morador_to_sindico | company_to_company | sindico_to_partner
```

### Status do Connect Me
```
open | interested | proposal_sent | em_negociacao | closed | expired
```

### Motivos de Recusa (Empresa, 6)
```
fora_area_atendimento | servico_fora_especialidade | agenda_indisponivel
conflito_servicos_agendados | orcamento_incompativel | outro_motivo
```

### Categorias de Connect Me Morador→Síndico (5)
```
duvida_gestao | solicitacao_manutencao | reclamacao | sugestao | outro
```

### Urgência (4)
```
baixa | media | alta | urgente
```

### Tipos de Parceria E→E (5)
```
subcontratacao | parceria_tecnica | fornecimento_materiais | indicacao_servicos | outro
```

### Tipos de Promoção S→Parceiro (5)
```
desconto_exclusivo | promocao_dia | campanha_sazonal | parceria_continua | outro
```

---

## 11. Commercial — Propostas / Deals / Execução

### Status da Proposta
```
draft | sent | awaiting_validation | validated | rejected | accepted
```

### Status do Deal
```
draft | awaiting_company_signature | awaiting_sindico_signature | confirmed | cancelled
```

### Status do Execution Record
```
draft | sent | awaiting_validation | validated | rejected
```

### Status genérico de Submissão da Empresa (E8 hub)
```
rascunho | enviado | aguardando_validacao | solicitado_ajuste
aprovado | publicado | rejeitado
```

### Risk Classification (Service)
```
baixo | moderado | elevado | critico
```

### Service Nature (4)
```
preventiva | corretiva | emergencial | estrategica
```

### Service Status (3)
```
concluido | em_andamento | parcialmente_concluido
```

### Company User Types (2)
```
administrador | operador_tecnico
```

### Critérios de Avaliação Empresa (Síndico → Empresa, 5)
```
qualidade_servico | cumprimento_prazo | atendimento | custo_beneficio | recomendacao
```
> Alternativa do PDF: `qualidade_tecnica · cumprimento_prazo · comunicacao · organizacao_documental · confiabilidade`

### Critérios de Avaliação da Gestão (Morador → Síndico, 3)
```
satisfacao_gestao | transparencia_comunicacao | percepcao_evolucao
```
Escala 1-10. Bimestral.

---

## 12. Vizinhança

### Categorias de Parceiros (38 organizadas em grupos)

**Alimentação (9)**: padaria · restaurante · lanchonete · cafeteria · pizzaria · hamburgueria · sorveteria · doceria · acai

**Mercado (3)**: supermercado · hortifruti · adega

**Saúde (1)**: farmacia

**Pet (3)**: pet_shop · clinica_veterinaria · banho_tosa

**Academia (4)**: academia · crossfit · pilates · yoga

**Estética (3)**: salao_beleza · barbearia · spa

**Serviços Domésticos (3)**: lavanderia · costureira · sapateiro

**Profissionais (5)**: eletricista · encanador · chaveiro · marceneiro · pintor

**Assistência Técnica (2)**: assistencia_celular · assistencia_computadores

**Automotivo (2)**: oficina · lava_jato

**Cursos (2)**: cursos_idiomas · reforco_escolar

**Outros (1)**: outros_servicos

### Tipos de Benefício (7)
```
desconto_percentual · desconto_fixo · produto_gratuito · combo_promocional
avaliacao_gratuita · brinde · beneficio_exclusivo
```

### Escopo de Promoção
```
aberta_bairro | exclusiva_condominio
```

### Status de Convite ao Parceiro
```
convite_enviado | parceiro_cadastrado | parceiro_ativo
```

---

## 13. Content / Vídeo

### Tipos de Vídeo Institucional (12)
```
apresentacao_institucional · apresentacao_equipe_tecnica · demonstracao_servico
explicacao_tecnica · boas_praticas_condominiais · orientacao_preventiva
caso_real_atendimento · treinamento_tecnico · explicacao_legislacao_normas
dicas_manutencao · conteudo_educativo_sindicos · conteudo_educativo_moradores
```

> Alternativa requirements.md (mesmos 12, rótulos parecidos). Adotar canônico.

### Status do Vídeo
```
processing | ready | failed | deleted | hidden
```

### Token Scope (Agência de Marketing)
```
videos:upload | videos:edit | analytics:read
```

---

## 14. LMS / Academia

### Categorias de Curso (10)
```
gestao_condominial · saude_ambiental · seguranca_predial · manutencao_predial
legislacao_condominial · mediacao_conflitos · tecnologia_condominios
sustentabilidade · administracao_financeira · outros
```

### Níveis de Curso (3)
```
basico | intermediario | avancado
```

### Tipos de Curso
```
gratuito | pago
```

### Categorias de Fórum (7)
```
gestao_condominial · manutencao_predial · seguranca · saude_ambiental
tecnologia · convivencia_moradores · legislacao_condominial
```

### Tipos de Material da Biblioteca (6)
```
videos | guias_tecnicos | manuais | livros_digitais | ebooks | materiais_tecnicos
```

### Tipos de Live (4)
```
debates_condominiais | lancamento_solucoes | palestras_tecnicas | entrevistas_especialistas
```

### Acesso a Lives (2)
```
abertas | exclusivas
```

### Trilhas de Aprendizado (3)
```
trilha_sindico | trilha_empresa | trilha_morador
```

---

## 15. Banco de Talentos / Vídeo-Currículo

### Áreas de Interesse (18)
```
operacoes_condominiais · limpeza_conservacao_higienizacao · manutencao_predial
seguranca_patrimonial · obras_reformas_adequacoes · administracao_apoio_administrativo
atendimento_relacionamento_suporte · logistica_almoxarifado_suprimentos
gestao_coordenacao_lideranca · treinamento_qualidade_processos_compliance
tecnologia_sistemas_suporte_digital · comercial_vendas_parcerias
comunicacao_marketing_conteudo · financeiro_contabil_controladoria
juridico_contratos_compliance_legal · recursos_humanos_gestao_pessoas
apoio_operacional_externo · outras_areas
```

### Modalidades de Contratação (9)
```
clt | pj | estagio | temporario | folguista | diarista | freelancer | intermitente | outros
```

### Horários Disponíveis
```
diurno | noturno | escala | finais_semana
```

### Início Disponível
```
imediato | ate_15_dias | a_combinar
```

### Tempo de Deslocamento
```
ate_30min | ate_1h | ate_1h30
```

### CNH Types (4)
```
nao | A | B | AB
```

### Cursos NR (4 + outros)
```
NR-10 | NR-33 | NR-35 | outros
```

### Motivo de Saída de Emprego (4)
```
termino_contrato | nova_oportunidade | ajuste_horario | outros
```

---

## 16. Marketing Link

### Tipos de Apoio (6)
```
producao_videos_institucionais · gestao_conteudos_tecnicos · estrategia_posicionamento
gestao_redes_sociais · consultoria_marketing · outro
```

### Prioridade (3)
```
imediato | proximos_30_dias | sem_prazo_definido
```

### Status do Marketing Link
```
sent | viewed | proposal_sent | atendido | accepted | rejected
```

---

## 17. Compliance

### Status Geral de Compliance
```
em_dia | parcial | pendente
```

### Tipos de Risco (Compliance)
```
operacional | financeiro | juridico | estrutural | reputacional
sanitario | trabalhista | tecnologia_seguranca_informacao
```

### Nível de Risco
```
baixo | medio | alto | critico
```

### Compliance Status (resumido — 3)
```
em_dia | pendente | atrasado
```

### Status de Assinatura
```
assinado | pendente
```

### Gatilhos que tornam Compliance obrigatório
- Encerrar mandato
- Gerar relatório final
- Marcar negócio fechado (opcional)
- Exportar dossiê

---

## 18. Sistema / Operacional

### Tipos de Tenant (2)
```
condominium | company
```

### Token Scope (cross-tenant)
```
connect_me | validation | empresa_sindica | marketing_agency
```

### Audit Log Event Types (exemplos — não exaustivo)
```
user_registered · user_authenticated · subscription_created · subscription_expired
condominium_created · timeline_published · master_plan_action_created
connect_me_request_created · connect_me_interest_shown
deal_confirmed · execution_validated · adendo_created
assembly_started · vote_cast · vote_homologated · assembly_finalized
declaration_submitted · signature_completed · conflict_declared
data_revealed · cross_tenant_access
```

### Levels de Log
```
debug | info | warn | error | fatal
```

---

## 19. Mensagens / Padrões UI

### Mensagem de bloqueio de plano (T15)
> "Essa funcionalidade está disponível nos planos ativos da plataforma Master Síndico. Ative seu plano para desbloquear esta função."

### Mensagem de bloqueio de compliance
> "Para prosseguir, finalize as pendências de compliance."

### Disclaimer de marcadores autodeclarados
> "Esses marcadores são informativos, baseados em autodeclaração, e não representam certificação oficial da plataforma."

### Disclaimer de promoções de parceiro
> "Condições divulgadas são de responsabilidade exclusiva do parceiro."

---

## Referências

- `backend/.kiro/specs/master-sindico/requirements/identity.md` — roles + status user
- `backend/.kiro/specs/master-sindico/requirements/billing.md` — plans + trial
- `backend/.kiro/specs/master-sindico/requirements/institutional.md` — condomínio + unidade + timeline
- `backend/.kiro/specs/master-sindico/requirements/commercial.md` — Connect Me + propostas + vizinhança
- `backend/.kiro/specs/master-sindico/requirements/content.md` — vídeo + LMS
- `backend/.kiro/specs/master-sindico/requirements/assembly.md` — assembleia
- [[../02-Jornadas/Inventario-Telas]] — onde cada enum aparece
- [[../regras-de-negocio/quebras-detectadas]] — Q24 (cardinalidade comunicados), Q26 (sprints)
