---
title: "Módulo Compliance — Detalhado (C1-C11)"
updated: 2026-04-22
maintainer: agent-3 (analista)
type: module
status: stable
source: Content/contents/####################TELA DO MORADOR#####(1).ini (Bloco 18 + telas C1-C11)
related-spec: backend/.kiro/specs/master-sindico/requirements/cross-domain.md (Req 33-42)
---

# Módulo Compliance — Síndico

11 telas dedicadas a blindagem documental, rastreabilidade e mitigação de risco jurídico. Acessível via botão dedicado na Central de Governança.

> **Princípio**: o módulo NÃO bloqueia o cadastro inicial do condomínio. Mas determinados marcos exigem compliance ativo (gatilhos no §X).

---

## Mensagem de entrada
> "Compliance é a camada de proteção da sua gestão. Aqui ficam registradas declarações formais, responsabilidades assumidas, trilhas de auditoria e evidências institucionais."

---

## C1 — Painel de Compliance

### Indicadores
- **Status geral**: Em dia / Parcial / Pendente
- **Pendências**:
  - Declaração anual
  - Usuários sem assinatura
  - Negócios fechados sem termo completo
  - Ações críticas sem classificação de risco
  - Mandato próximo do vencimento

### Botões
Declaração anual · Assinaturas pendentes · Auditoria · Imutabilidade · Mapa de risco · Conformidade contratual · LGPD · Score · Exportar Dossiê

---

## C2 — Declaração Anual do Síndico

**Frequência**: 1x por ano OU a cada novo mandato.

### Texto jurídico (versionado)
> "Declaro que exerço a função de síndico com diligência, observando a legislação aplicável, a convenção condominial, o regimento interno e as deliberações assembleares. Declaro que os registros realizados nesta plataforma refletem, dentro do meu conhecimento, informações verdadeiras e que busco atuar com transparência, responsabilidade administrativa e mitigação de riscos institucionais.
>
> Declaro ainda que compreendo que os registros aqui efetuados possuem caráter documental e poderão ser utilizados como histórico formal da gestão."

**Checkbox obrigatório** + Botão "Confirmar declaração".

### Registro automático
- Data, hora, IP, usuário
- Versão dos termos vigente

### Publicação automática
Linha do Tempo · Tipo: `administrativo → conformidade_e_governanca` · Visibilidade: Gestão · **Imutável**.

---

## C3 — Assinatura Obrigatória dos Usuários

Lista de usuários cadastrados (até 3): Síndico principal · Síndico preposto · Administradora · Conselho.

**Status individual**: Assinado · Pendente

### Texto para usuários convidados
> "Declaro ciência de que minha atuação nesta plataforma é rastreável e deve refletir informações verdadeiras e diligentes. Estou ciente de que minhas ações poderão compor histórico documental da gestão condominial."

### Regra técnica
**Compliance só fica "Em dia" quando TODOS assinarem.** Cada assinatura gera publicação automática na Linha do Tempo.

---

## C4 — Matriz de Conflito de Interesse

Síndico declara: "Possuo ou não possuo relação direta ou indireta com fornecedores cadastrados."

### Campos
- Possui vínculo com fornecedor? Sim/Não
- Se Sim: descrever empresa, natureza da relação, condomínio impactado

**Registro imutável.**

> Justificativa: protege contra alegação futura de favorecimento.

---

## C5 — Trilha de Auditoria Visível

### Lista estruturada
- Usuário, Perfil, Ação realizada, Campo alterado, Versão anterior, Versão atual, Data, Hora

### Filtros
Usuário · Data · Tipo de ação

**Imutável**. Base probatória.

### Eventos rastreados (do Bloco 18)
- Criação de atividades
- Validação de registros
- Publicação de comunicados
- Alterações cadastrais
- Atualização do plano diretor
- (do requirements.md Req 37): Login/logout, Criação/modificação de pauta, Votação, Mudança de permissões, Exclusão de dados (soft delete)

### Retenção
**5 anos** (LGPD). Após 1 ano, archive em MinIO/S3 Glacier.

### Acesso
Apenas admin + síndico principal podem acessar.

---

## C6 — Imutabilidade e Adendos

### Registros bloqueados
- Votações encerradas
- Negócios fechados
- Declarações
- Eventos encerrados

### Botão "Criar adendo formal"
**Adendo exige**:
- Justificativa
- Descrição (voz ou texto)
- Anexo opcional

**Comportamento**: registro novo, sem alterar original. Permite correção sem apagar histórico.

---

## C7 — Mapa de Risco Consolidado

### Consolida
- Ações classificadas por risco
- Distribuição por categoria
- Quantidade alto/crítico
- Ações vencidas

### Indicador visual
Baixo · Médio · Alto · Crítico

> Justificativa: transforma dados em visão estratégica.

---

## C8 — Conformidade Contratual

Lista de todos os negócios fechados.

### Por item
Valor · Prazo início · Prazo conclusão · Garantia · Contrato anexado (sim/não) · Termos assinados (sim/não) · Comunicação pós-fechamento validada (sim/não)

### Alertas
- Prazo de garantia próximo do fim
- Prazo de execução vencido

> Justificativa: evita passivo por esquecimento.

---

## C9 — Registro LGPD

### Exibe
Histórico Connect Me: Empresa · Data · Finalidade · Dados compartilhados · Consentimento registrado

### Texto fixo
> "Os dados foram compartilhados mediante autorização expressa para finalidade específica, conforme Lei 13.709/2018."

**Imutável**.

> Justificativa: proteção contra questionamento de vazamento.

### Implementação atual
Tabela `lgpd_logs` central, integrada com todos os módulos. Retenção 5 anos. Direito ao esquecimento: purga após vencimento.

---

## C10 — Score Interno de Governança

**Privado**.

### Cálculo baseado em
- Percentual de ações preventivas
- Cumprimento de prazos
- Classificação de risco adequada
- Participação dos moradores
- Compliance ativo
- Validações em dia
- Trilha consistente

### Exibe
- Score Governança
- Score Compliance

### Recomendações automáticas
- Regularizar assinaturas
- Classificar risco pendente
- Atualizar ação vencida
- Encerrar votação aberta

> Justificativa: gamificação profissional e orientação.

> ⚠️ **Inconsistência**: ####TELA####(1).ini fala em "Score Governança 1-10 privado" e "Score Compliance 0-100 privado" (dois scores). `requirements.md` Req 41 fala em "Governance Score 0-100" (um só, escala 0-100 público). Ver Q25 em [[../regras-de-negocio/quebras-detectadas]].

---

## C11 — Dossiê de Governança Exportável

**Botão**: Gerar Dossiê

### Compila
- Declarações anuais
- Assinaturas dos usuários
- Matriz de conflito
- Mapa de risco
- Trilha de auditoria (resumo)
- Contratações
- Termos assinados
- LGPD (resumo)
- Indicadores

### Formato
PDF estruturado.

### Uso
- Assembleia
- Processo judicial
- Auditoria
- Seguro

---

## Regras de Gatilho

Compliance passa a ser obrigatório para:
1. **Encerrar mandato**
2. **Gerar relatório final**
3. **Marcar negócio fechado** (opcional, se quiser endurecer)
4. **Exportar dossiê**

### Sistema exibe quando bloqueia
> "Para prosseguir, finalize as pendências de compliance."

---

## Continuidade entre Mandatos

Quando novo síndico assume, ele deve:
- Assinar nova declaração
- Atualizar conflito de interesse
- Mas mantém histórico anterior visível (somente leitura)

---

## Mapa: Compliance → Spec viva

| Tela | Spec |
|---|---|
| C2 (Declaração) | `requirements/cross-domain.md` Req 34 |
| C3 (Assinaturas) | `requirements/cross-domain.md` Req 35 |
| C4 (Conflitos) | `requirements/cross-domain.md` Req 36 |
| C5 (Auditoria) | `requirements/cross-domain.md` Req 37 — **append-only** |
| C6 (Adendos) | `requirements/commercial.md` Req 25 + `requirements/institutional.md` Req 11 |
| C7 (Risco) | `requirements/cross-domain.md` Req 38 |
| C9 (LGPD) | `requirements/cross-domain.md` Req 39 — `lgpd_logs` |
| C10 (Score) | `requirements/cross-domain.md` Req 33 + Req 41 |
| C11 (Dossiê) | `requirements/cross-domain.md` Req 40 + Req 42 |

---

## Implementação backend (DT-010 — STATE.md)

**Promovido a dívida técnica DT-010**: `IAuditLogger` cross-módulo. Hoje cada módulo loga isoladamente; precisa de adapter centralizado + DI wiring em 3 módulos. Sprint 10.

---

## Referências

- [[../02-Jornadas/Inventario-Telas]] — telas C1-C11
- [[../regras-de-negocio/regras-canonicas]] — Regra de Ouro #10 (LGPD)
- [[../regras-de-negocio/quebras-detectadas]] Q25 — Score 1-10 vs 0-100
- `backend/.kiro/specs/master-sindico/requirements/cross-domain.md` Req 33-42 (canônico)
- `backend/.kiro/STATE.md` DT-010 (IAuditLogger)
