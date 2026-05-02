---
title: "Inventário de Telas — Master Síndico (extraído dos PDFs/MDs originais)"
updated: 2026-04-22
maintainer: agent-3 (analista)
type: catalog
status: stable
source: Content/contents/JORNADA*.pdf, ONBOARDING DOS PERFIS.pdf, MÓDULO ACADEMIA.ini, MÓDULO VIZINHANÇA.*, ####TELA DO MORADOR####(1).ini
---

# Inventário de Telas — Master Síndico

Consolidação fiel dos PDFs e MDs originais em `contents/`. **Total: 114 telas mapeadas** distribuídas por 9 jornadas. Cada tela tem ID estável (M1-M15, S1-S31, E1-E16, MK1-MK8, VS1-VS10, VE1-VE6, VM1-VM7, VZ1-VZ6, LMS1-LMS15) que será usado pelo agente principal e pelos times web/app na implementação.

> **Nota:** Esta é a fonte de verdade de PRODUTO. A arquitetura técnica (handlers, rotas, componentes) deriva — mas não substitui — esta lista. Quando faltar tela, abrir D-0XX em `backend/.kiro/STATE.md`.

---

## 1. Jornada do Morador (M1–M15) — 15 telas

| ID | Nome | Módulo | PDF/source |
|---|---|---|---|
| M1 | Painel do Morador | Home | JORNADA DO MORADOR.pdf |
| M2 | Selecionar Condomínio | Navegação | M2 |
| M3 | Buscar Condomínio (por ID) | Onboarding | M3 |
| M4 | Cadastro da Unidade | Onboarding | M4 |
| M5 | Home do Condomínio | Home | M5 |
| M6 | Linha do Tempo | Governança (leitura) | M6 |
| M7 | Detalhe da Publicação | Governança (leitura) | M7 |
| M8 | Plano Diretor | Governança (leitura) | M8 |
| M9 | Detalhe da Ação | Governança (leitura) | M9 |
| M10 | Eventos | Eventos | M10 |
| M11 | Comunicados | Comunicados | M11 |
| M12 | Avaliar Gestão | Avaliação (bimestral, 3 perguntas escala 1-10) | M12 |
| M13 | Meu Vínculo | Membership | (####TELA####.ini Tela 2) |
| M14 | Gerenciar Acessos (dependentes) | Membership | (####TELA####.ini Tela 14) |
| M15 | Encerrar Vínculo | Membership | (####TELA####.ini Tela 15) |

### Regras estruturais (PDF Morador)
- R1: Linha do Tempo é principal fonte de info
- R2: Morador NÃO publica conteúdo (só visualiza, responde perguntas, participa de eventos, avalia gestão)
- R3: Vínculo com unidade é obrigatório (sem vínculo, sem acesso ao condomínio)
- R4: Dependentes vinculados ao titular (NÃO podem alterar status, remover titular ou incluir novos dependentes)
- R5: Unidade vazia → dependentes removidos automaticamente

---

## 2. Jornada do Síndico (S1–S31) — 31 telas

| ID | Nome | Módulo |
|---|---|---|
| S1 | Entrada da Governança | Onboarding |
| S2 | Meus Condomínios | Navegação (até 15, ordenar ativos primeiro) |
| S3 | Cadastrar Condomínio (Endereço) | Registry |
| S4 | Dados da Gestão (Mandato) | Registry |
| S5 | Cadastro / Vínculo Empresa Síndica | Registry |
| S6 | Home da Governança | Home (10 cards: PD, LT, Eventos, Connect Me, Registros execução, Comunicados, Validações Pendentes, Dashboard, Compliance, Modo Assembleia) |
| S7 | Plano Diretor | PD |
| S8 | Cadastrar Ação PD | PD |
| S9 | Lista Ações PD | PD |
| S10 | Detalhe Ação PD | PD |
| S11 | Linha do Tempo | LT |
| S12 | Criar Atividade | LT |
| S13 | Editar Atividade (gera nova versão) | LT |
| S14 | Histórico LT | LT |
| S15 | Eventos | Eventos |
| S16 | Criar Evento | Eventos |
| S17 | Lista Eventos | Eventos |
| S18 | Connect Me (hub) | Connect Me |
| S19 | Criar Connect Me | Connect Me |
| S20 | Connect Me Recebido (M→S) | Connect Me |
| S21 | Registros de Execução | Validações |
| S22 | Detalhe Registro Execução | Validações |
| S23 | Comunicados | Comunicados |
| S24 | Criar Comunicado | Comunicados |
| S25 | Lista Comunicados | Comunicados |
| S26 | Validações Pendentes (hub) | Validações |
| S27 | Dashboard | Dashboard |
| S28 | Compliance (entrada) | Compliance |
| S29 | Termo de Responsabilidade | Compliance |
| S30 | Declaração Anual | Compliance |
| S31 | Auditoria | Compliance |

### Regras estruturais (PDF Síndico)
- R1: Linha do Tempo é memória oficial; tudo passa por lá
- R2: Plano Diretor não atualiza manualmente (só via publicação na LT)
- R3: Empresa não publica direto — passa por validação do síndico
- R4: Edição preserva histórico (versão anterior + trilha)
- R5: Síndico pode consultar até **12 condomínios** (PDF Mar/2026) — **mas Decision 11 diz 15** (atual). Ver [[../regras-de-negocio/quebras-detectadas]] Q30.
- R6: Empresa síndica vinculada pode: visualizar publicações, editar publicações do síndico, ocultar publicações, validar registros de empresas, validar comunicados de empresas

### Regras adicionais do ####TELA####(1).ini (telas-detalhe sind.)
- R1 (anti-erro): Confirmação de contexto antes de publicar ("Publicar para Condomínio X?")
- R2: Voz em todas descrições (comando de voz + texto)
- R3: Nada é deletado (apenas editar/ocultar com trilha)
- R4: Origem externa sempre pendente
- R5: Pós-fechamento é imutável (alteração só via Adendo Formal)
- R6: Continuidade institucional (novo síndico herda histórico, antigo mantém leitura)
- R7: Compliance NÃO trava onboarding (pode ficar pendente até gatilho)
- R8: Compliance só fica "em dia" quando todos usuários assinarem
- R9: Notificações internas em toda ação importante
- R10: LGPD e rastreabilidade (todo compartilhamento registrado)

### Bloco 4 (####TELA####(1).ini) — Usuários e Permissões
Até 3 usuários por condomínio. Perfis no convite: Síndico Preposto / Administradora / Conselho (leitura ampliada).

Permissões configuráveis (toggles em S5/S8): criar PD, registrar atividade LT, tornar registro público, criar/encerrar evento, criar/encerrar votação, cadastrar/validar proposta, gerar link externo, marcar/confirmar negócio fechado, criar adendo formal, exportar relatório, acessar/assinar compliance.

---

## 3. Jornada da Empresa (E1–E16) — 16 telas

| ID | Nome | Módulo |
|---|---|---|
| E1 | Painel Empresarial | Home (cards: Oportunidades, Registrar execução, Atividades técnicas, Comunicados técnicos, Status de envios, Dashboard, Perfil, Usuários, Gestão de Agência, Marketing Link) |
| E2 | Oportunidades (Connect Me) | Connect Me |
| E3 | Recusar Oportunidade (motivo estruturado: 6 opções + justificativa) | Connect Me |
| E4 | Detalhe da Oportunidade (após "Tenho interesse" — revela contato) | Connect Me |
| E5 | Registrar Execução | Service |
| E6 | Atividade Técnica | Service |
| E7 | Comunicados Técnicos | Service |
| E8 | Status de Envios (hub) | Service |
| E9 | Detalhe do Registro | Service |
| E10 | Dashboard Empresarial | Dashboard |
| E11 | Perfil Institucional | Profile |
| E12 | Usuários da Empresa (Administrador / Operador Técnico) | Profile |
| E13 | Gestão de Agência (delegação) | Marketing |
| E14 | Convidar Agência (token) | Marketing |
| E15 | Gerenciar Acessos Agência | Marketing |
| E16 | Formulário Marketing Link (E→Agência, NÃO cria vínculo automático) | Marketing |

### Regras estruturais (PDF Empresa)
- 3 pilares: relacionamento com síndicos (Connect Me) + registro técnico das atividades + construção de reputação institucional
- Toda interação empresa↔condomínio passa por **validação do síndico**
- Marketing Link apenas registra intenção de contato — vínculo oficial só com convite via "Gestão de Agência de Marketing"

### Indicadores E1 (PDF Empresa Painel)
Novas oportunidades, Propostas aguardando validação, Votações em andamento, Negócios ativos, Execuções pendentes de validação, Garantias próximas do vencimento, Avaliações recebidas.

### Estados de execução (E9 — ####TELA####(1).ini)
`rascunho → enviado para validação → aprovado → publicado → bloqueado para edição` (alternativos: solicitado_ajuste, rejeitado).

### Termos versionados (E8)
"Termo de Execução" institucional com checkbox + LGPD log (usuário, IP, data, hora, versão do termo).

---

## 4. Jornada da Agência de Marketing (MK1–MK8) — 8 telas

| ID | Nome | Módulo |
|---|---|---|
| MK1 | Painel da Agência (cards: Empresas atendidas, Publicar vídeos, Biblioteca, Dashboard, Marketing Link) | Home |
| MK2 | Empresas Atendidas (lista após aceitar convite) | Gestão |
| MK3 | Gerenciar Empresa (cards: Publicar vídeo, Biblioteca, Dashboard) | Gestão |
| MK4 | Publicar Vídeo (12 tipos) | Content |
| MK5 | Biblioteca de Vídeos (editar/ocultar — não remove histórico) | Content |
| MK6 | Dashboard da Empresa (views, retenção, vídeo mais visto) | Dashboard |
| MK7 | Dashboard Geral (empresas, total vídeos, total views, vídeo mais acessado, empresa com maior engajamento) | Dashboard |
| MK8 | Marketing Link Recebido (lista de solicitações Empresa→Agência) | Marketing |

### Limitações (PDF Marketing)
Agência **NÃO pode acessar**: Connect Me · Registros de execução · Comunicados técnicos · Linha do Tempo do condomínio · Dados de síndicos · Dados de moradores · Dashboard de governança.

### Visibilidade vídeos (regra crítica)
Moradores **NÃO** acessam vídeos institucionais normalmente. Só visualizam quando empresa participa de processo de **escolha de fornecedor no módulo assembleia**. Após votação, acesso bloqueado.

### Checkbox no upload (MK4)
"Autorizar exibição em processos de escolha de fornecedor" — se desmarcado, vídeo fica restrito ao perfil da empresa.

---

## 5. Jornada do Parceiro da Vizinhança (VZ1–VZ6) — 6 telas

| ID | Nome | Módulo |
|---|---|---|
| VZ1 | Cadastro do Parceiro (App→Vizinhança→"Quero ser parceiro") | Onboarding |
| VZ2 | Perfil do Parceiro (seções Promoções + Métricas) | Profile |
| VZ3 | Criar Promoção (7 tipos + segmentação aberta/exclusiva) | Promotion |
| VZ4 | Promoções Ativas (editar/encerrar) | Promotion |
| VZ5 | Métricas do Parceiro (visualizações perfil/promoção, ativações, cliques contato, gráficos de engajamento) | Dashboard |
| VZ6 | Histórico de Promoções | History |

### Regras (PDF Parceiro)
- Não exige CNPJ (CPF basta)
- Pode: criar/editar/encerrar promoções
- NÃO pode: publicar comunicados em condomínios, registrar execução de serviços, acessar painel empresarial
- Tipos de promoção: aberta_bairro (visível a moradores+síndicos+empresas) OU exclusiva_condomínio (só moradores do condomínio selecionado)

### 7 tipos de benefício (Decision 6 + VZ3)
`desconto_percentual · desconto_fixo · produto_gratuito · combo_promocional · avaliacao_gratuita · brinde · beneficio_exclusivo`

---

## 6. Jornada Vizinhança — Síndico (VS1–VS10) — 10 telas

| ID | Nome |
|---|---|
| VS1 | Painel da Vizinhança (4 seções: explorar parceiros, promoções exclusivas do condomínio, parceiros indicados, histórico) |
| VS2 | Explorar Parceiros (filtros: categoria, bairro) |
| VS3 | Detalhe do Parceiro |
| VS4 | Detalhe da Promoção |
| VS5 | Ativar Promoção (gera métrica para parceiro) |
| VS6 | Indicar Parceiro Local (envia link para cadastro) |
| VS7 | Criar Benefício Exclusivo para Condomínio (visível só para moradores daquele condo) |
| VS8 | Benefícios do Condomínio (editar/encerrar) |
| VS9 | Parceiros Indicados pelo Síndico (status: convite enviado, parceiro cadastrado, parceiro ativo) |
| VS10 | Histórico de Promoções Ativadas |

---

## 7. Jornada Vizinhança — Empresa (VE1–VE6) — 6 telas

| ID | Nome |
|---|---|
| VE1 | Painel da Vizinhança para Empresas (3 seções: explorar, promoções, histórico) |
| VE2 | Explorar Parceiros |
| VE3 | Detalhe do Parceiro |
| VE4 | Detalhe da Promoção |
| VE5 | Ativar Promoção |
| VE6 | Histórico de Promoções Utilizadas |

### Regra crítica (PDF Vizinhança Empresa)
Empresa **NÃO** vê promoções exclusivas (só moradores do condomínio). Empresa **só** vê promoções abertas (bairro).

---

## 8. Jornada Vizinhança — Morador (VM1–VM7) — 7 telas

| ID | Nome |
|---|---|
| VM1 | Painel/Feed do bairro |
| VM2 | Detalhe do Parceiro |
| VM3 | Detalhe da Promoção |
| VM4 | Ativar Promoção |
| VM5 | Benefícios do meu Condomínio (exclusivos) |
| VM6 | Parceiros Indicados pelo Síndico (badge "Indicado") |
| VM7 | Histórico de Ativações |

---

## 9. Jornada LMS / Academia (LMS1–LMS15) — 15 telas

| ID | Nome |
|---|---|
| LMS1 | Entrada na Academia (5 seções: Cursos, Treinamentos, Lives, Fórum, Biblioteca) |
| LMS2 | Explorar Cursos (filtros: categoria, nível, tipo) |
| LMS3 | Página do Curso |
| LMS4 | Aula do Curso (sistema registra progresso) |
| LMS5 | Conclusão do Curso (baixar/compartilhar certificado PDF) |
| LMS6 | Treinamentos (lista) |
| LMS7 | Página do Treinamento |
| LMS8 | Agenda de Lives |
| LMS9 | Sala da Live (vídeo + chat + perguntas) |
| LMS10 | Replay das Lives |
| LMS11 | Fórum (7 categorias) |
| LMS12 | Criar Tópico (empresas também podem criar) |
| LMS13 | Discussão do Fórum |
| LMS14 | Biblioteca (filtros: tipo, categoria) |
| LMS15 | Página do Material (baixar / ler online) |

### Acesso LMS (regra dura — Decision 7)
✅ Síndicos · Empresas · Moradores
❌ **Parceiro da Vizinhança** (botão "Academia" oculto)

### Cursos pagos
Direcionam para módulo de pagamento (Stripe — não Mercado Pago).

---

## 10. Módulo Compliance (C1–C11) — 11 telas (do ####TELA####(1).ini)

Ver [[../03-Modulos/Compliance-Detalhado]] para detalhe.

| ID | Nome |
|---|---|
| C1 | Painel de Compliance (status: Em dia / Parcial / Pendente) |
| C2 | Declaração Anual do Síndico (1x/ano ou novo mandato) |
| C3 | Assinatura Obrigatória dos Usuários |
| C4 | Matriz de Conflito de Interesse |
| C5 | Trilha de Auditoria Visível (imutável) |
| C6 | Imutabilidade e Adendos |
| C7 | Mapa de Risco Consolidado |
| C8 | Conformidade Contratual (alertas garantia/prazo) |
| C9 | Registro LGPD (Connect Me, finalidade, dados) |
| C10 | Score Interno de Governança (privado) |
| C11 | Dossiê Exportável (PDF) |

### Gatilhos (compliance vira obrigatório para)
- Encerrar mandato
- Gerar relatório final
- Exportar dossiê
- (Opcional) Marcar negócio fechado se quiser endurecer

---

## 11. Módulo Assembleia (sem ID estável; estruturado em 5 etapas)

Ver [[../03-Modulos/Assembleia-Live-Funcional]] para detalhe completo extraído do PDF "ARQUITETURA FUNCIONAL DO MÓDULO ASSEMBLEIA".

5 blocos do módulo:
1. Configuração estrutural do condomínio
2. Pré-assembleia
3. Assembleia ao vivo
4. Encerramento
5. Pós-assembleia e histórico

---

## 12. Banco de Talentos / Vídeo-Currículo (TELA 01–11) — 11 telas

Detalhe pendente leitura completa dos PDFs "COMO A EMPRESA VÊ O CURRÍCULO DO MORADOR" e "ESTRUTURA DE CADASTRO DE CURRÍCULO". Resumo do `requirements.md` original Decision 6:

| TELA | Nome |
|---|---|
| 01 | Boas-vindas |
| 02 | Vídeo de Apresentação (90s, lock 90 dias) |
| 03 | Perfil Profissional (6 perguntas abertas) |
| 04 | Áreas de Interesse (18 áreas) |
| 05 | Dados Pessoais (CEP só para identificação de bairro) |
| 06 | Disponibilidade e Contratação (modalidade, horários, início, deslocamento) |
| 07 | Pretensão Salarial (mínimo + ideal) |
| 08 | Experiência Profissional (⚠️ INCONSISTÊNCIA: 3 vs 5 vínculos — ver [[../regras-de-negocio/quebras-detectadas]]) |
| 09 | Formação e Autorrelato |
| 10 | Consentimento LGPD |
| 11 | Confirmação |

7 funcionalidades para Empresa visualizar currículos: Favoritos · Exportar PDF · Relatório de Buscas (Histórico) · Relatório de Buscas (Funil) · Tags Automáticas (7 categorias) · Tags Manuais (curadoria empresa) · Busca Avançada com Filtros.

---

## 13. Telas-pivô do ecossistema (críticas para arquitetura)

| Tela | Por que crítica |
|---|---|
| S6 — Home da Governança | Único dashboard cross-módulo do síndico (10 cards + indicadores em tempo real) |
| S26 — Validações Pendentes | Hub centralizado de tudo que veio de empresa e precisa de aprovação |
| E1 — Painel Empresarial | Mesma função para empresa (10 cards + indicadores) |
| Telão da Assembleia (2 áreas) | UI real-time com latência crítica < 200ms; 2 monitores |
| Modo Assembleia (apresentação) | Modo "slides" para conduzir reunião |

---

## Métricas do inventário

| Persona | Telas |
|---|---|
| Morador | 15 |
| Síndico | 31 |
| Empresa | 16 |
| Agência Marketing | 8 |
| Parceiro Vizinhança | 6 (VZ1-VZ6) |
| Vizinhança Síndico | 10 (VS1-VS10) |
| Vizinhança Empresa | 6 (VE1-VE6) |
| Vizinhança Morador | 7 (VM1-VM7) |
| LMS Academia | 15 (LMS1-LMS15) |
| Compliance Síndico | 11 (C1-C11) |
| Banco Talentos Morador | 11 (TELA 01-11) |
| Assembleia | 5 etapas, sem IDs (~30+ telas) |
| **Total catalogado** | **~136+ telas** |

> Inventário oficial requirements.md menciona "114 screens mapped". Diferença vem de telas como Compliance (C1-C11) e Banco Talentos (TELA 01-11) catalogadas separadamente neste vault.

---

## Referências cruzadas

- [[Onboarding-por-Persona]] — fluxo de cadastro inicial (4 personas)
- [[../03-Modulos/Assembleia-Live-Funcional]] — módulo assembleia detalhado
- [[../03-Modulos/Compliance-Detalhado]] — C1-C11
- [[../regras-de-negocio/regras-canonicas]] — 10 regras de ouro + 12 decisões
- [[../regras-de-negocio/quebras-detectadas]] — inconsistências entre PDFs e specs vivas
- `backend/.kiro/specs/master-sindico/requirements/*.md` — fonte canônica viva
