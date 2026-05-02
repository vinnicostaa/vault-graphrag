---
title: "Módulo Assembleia (Live) — Arquitetura Funcional"
updated: 2026-04-22
maintainer: agent-3 (analista)
type: module
status: stable
source: Content/contents/ARQUITETURA FUNCIONAL DO MÓDULO ASSEMBLEIA.pdf, DIAGRAMA VISUAL DA ASSEMBLEIA.pdf
related-spec: backend/.kiro/specs/master-sindico/requirements/assembly.md (Req 48-60.12)
---

# Módulo Assembleia — Arquitetura Funcional (Live)

> ⚠️ **MUDANÇA DE ESCOPO**: O escopo técnico antigo (excopo Sprint 2.1) declarava "É assíncrono. Não é votação em blockchain. Não é assembleia virtual ao vivo (Zoom/Meet)". A **Decision 1 atual** confirma **Live Assembly** com WebSocket real-time. Mudança contratual relevante. Ver [[../regras-de-negocio/quebras-detectadas]] Q27.

## 1. Objetivo (do PDF)

O módulo Assembleia é a estrutura digital de:
- Organização prévia da assembleia
- Convocação e rastreabilidade de ciência
- Preparação de quórum, procurações, fração ideal e elegibilidade de voto
- Condução da assembleia (presencial, **híbrida ou online preparadas mas inativas no MVP**)
- Apoio à fala, votação e deliberação
- Transparência operacional em tempo real
- Geração de histórico, trilha de auditoria e base documental pós-assembleia

### O que a plataforma NÃO substitui
- A convenção do condomínio
- A administradora
- O presidente da mesa
- A assessoria jurídica
- A formalização externa de assinatura da ata

### O que a plataforma é
- Meio tecnológico de organização e registro
- Ambiente de transparência e controle
- Apoio operacional à assembleia

---

## 2. Estrutura macro — 5 blocos

1. **Configuração estrutural do condomínio**
2. **Pré-assembleia**
3. **Assembleia ao vivo**
4. **Encerramento**
5. **Pós-assembleia e histórico**

---

## 3. Configuração Estrutural do Condomínio

### 3.1 Cadastro-base
Campos: nome, endereço, blocos, unidades, **síndico principal** (confirmar), subsíndico (preencher), conselho (preencher), administradora (preencher), auditor (preencher).

### 3.3 Habilitação da Administradora
**Caso seja empresa Master Síndico**: vincular por busca por CNPJ ou seleção no diretório Master.
**Caso não seja empresa Master Síndico**: criação de acesso externo controlado:
1. Síndico informa empresa, CNPJ, responsável, CPF, e-mail e telefone
2. Sistema envia convite
3. Administradora valida o acesso por token
4. Confirma CPF do responsável
5. Aceita termo de responsabilidade
6. Passa a ter acesso apenas ao módulo assembleia daquele condomínio

### 3.4 Permissões da administradora
Pode: visualizar assembleia em construção, adicionar observações, gerar edital automático ou anexar edital pronto, subir planilha de fração ideal, subir planilha de inadimplência, validar procurações, liberar voto de unidade regularizada, registrar voto presencial assistido, homologar votação, encerrar assembleia.

> Observações da administradora poderão exigir validação do síndico, conforme configuração.

---

## 4. Pré-Assembleia

### 4.1 Criação da assembleia
**Campos**:
- Tipo: ordinária / extraordinária / implantação
- Data
- Horário da 1ª convocação
- Horário da 2ª convocação
- Modalidade: presencial (MVP), híbrida ou online (preparadas, inativas)
- Local físico (se houver)
- Link digital (se houver)
- Descrição geral
- Tempo máximo de fala por morador: **2 ou 3 minutos com botão de prorrogação de mais 2 min**

### 4.2 Cadastro dos itens da pauta
Itens são **ilimitados**. Cada item deve conter:
- Título
- Descrição
- Item informativo OU deliberativo
- Tipo do item
- Impacto financeiro: sim ou não
- Valor estimado, se houver
- Quórum exigido para aprovação daquele item
- Anexo de documento
- Anexo de vídeo explicativo (caso queira)

### 4.3 Tipos de item (5)
1. `informativo`
2. `votacao_simples`
3. `votacao_fracao_ideal`
4. `escolha_fornecedor`
5. `eleicao`

### 4.4 Regras por tipo de item
- **Informativo** — sem votação oficial
- **Votação simples** — gera enquete preliminar (sem valor jurídico) + votação oficial na assembleia
- **Votação por fração ideal** — gera enquete preliminar + votação oficial com validação por peso da fração ideal
- **Escolha de fornecedor** — gera enquete preliminar + comparação de propostas + votação oficial
- **Eleição** — gera **avaliação/percepção da gestão (sem caráter jurídico, escondida — só aparece após registro de ciência do morador)** + votação oficial de eleição

> ⚠️ Esta regra de "eleição gera avaliação escondida" não está explícita em `requirements/assembly.md` atual. Ver Q40 em [[../regras-de-negocio/quebras-detectadas]].

### 4.5 Travamento jurídico da pauta (LOCK_Pauta)
Após "Publicar assembleia": não pode adicionar item, alterar título, alterar descrição, alterar quórum, alterar anexos estruturantes.

### 4.6 Edital de convocação
**Forma 1 — Anexo livre**: administradora ou síndico anexa edital pronto.

**Forma 2 — Gerador automático da plataforma** (`tirar` no PDF — nota de revisão):
Campos: identificação do condomínio, tipo da assembleia, data, horários de 1ª e 2ª convocação, modalidade, local ou link, ordem do dia (importada da pauta), regras de participação, regras de procuração, regras de votação, logo da administradora, campo de assinatura.

**Quem pode gerar**: síndico OU administradora com permissão.

### 4.7 Publicação da assembleia
Ao publicar, sistema dispara: **e-mail + notificação no app + SMS**. Cada assembleia gera um **QR code próprio**.

### 4.8 Ciência obrigatória
A ciência é obrigatória. **Até registrar ciência: o app fica bloqueado para outras funções.**

Sistema registra e armazena por **6 meses**: quem deu ciência, quando deu ciência, quem ainda não deu ciência, quem não abriu a comunicação.

Síndico e administradora visualizam: percentual de ciência, lista nominal de pendentes, identificação literal de quem não abriu.

### 4.9 Confirmação de participação (opcional)
Morador pode marcar: participarei presencialmente / participarei online / ainda não defini / Não participarei.

### 4.10 Lembretes automáticos
- 3 dias antes
- 1 dia antes
- 3 horas antes

### 4.11 Enquetes preliminares
Permitidas para: votação simples + votação por fração ideal. Opções de resposta: **apenas sim ou não**.

Regras: sem valor jurídico, servem para engajamento e preparação, não substituem deliberação.

### 4.12 Fornecedores
Moradores podem acessar perfil previamente (apenas durante o período da assembleia): proposta, escopo, diferenciais.

**Fornecedor Master**: mostra selo + vídeo institucional.
**Fornecedor externo**: mostra dados básicos + proposta + escopo + indicação de "empresa não verificada pela Master Síndico".

### 4.13 Procurações
Pode ser apresentada até **o momento anterior ao síndico clicar "Iniciar assembleia"**.

**Fluxo do morador**:
- Após registrar ciência, pode confirmar presença + cadastrar procurações
- Campos: bloco e unidade do representado, nome e CPF do representado, bloco e unidade do procurador, nome e CPF do procurador, checkbox de ciência e responsabilidade
- **Confirmação final com mensagem informando que ele precisa apresentar a procuração fisicamente para validação**

**Fluxo da administradora**:
- Visualiza dados do procurador + dados do representado
- Ações: aprovar (ao aprovar, número da unidade representante recebe o mesmo voto que a unidade representada — respeitando os pesos individuais) OU rejeitar com justificativa

**Efeito**: voto adicional computado automaticamente. Funciona para voto simples e fração ideal.

**Validade**: vínculo da procuração após votação é de **apenas 24h** após encerramento — depois perde efeito.

### 4.14 Fração ideal
**Para o MVP**: melhor arquitetura é **upload por planilha**.

Modelo da planilha: bloco · unidade · fração ideal · CPF titular.

**Regras de validação**: soma total das frações = 100% ou parâmetro equivalente · ausência de duplicidade de bloco+unidade · ausência de campos obrigatórios em branco.

**Persistência**: importação única, permanece salva no condomínio, nova importação apenas por correção ou mudança estrutural.

### 4.15 Inadimplência
Upload até **1 hora antes da 1ª convocação**.

Pode ser feito por: administradora ou síndico.

Sistema classifica: apto a votar / inapto a votar.

**Caso especial — comprovante de pagamento de última hora**: permissão especial e manual via botão **"Liberar voto"**. Usuários com permissão para liberar: síndico ou administradora. Campos obrigatórios: checkbox de aprovação para votação, responsável, horário.

### 4.16 Simulador de quórum
Síndico e administradora visualizam: total de unidades, total de frações, percentual de ciência, percentual de confirmação de presença, número de procurações registradas a serem aprovadas, número de procurações aprovadas, **quórum projetado por item**.

### 4.17 Termos obrigatórios na pré-assembleia
Antes de concluir ciência ou participação, morador deve aceitar:
- Termo de ciência da convocação
- Termo LGPD
- Declaração de veracidade das informações prestadas
- Declaração de identidade e legitimidade de uso do acesso da unidade
- Checkbox de autorização de gravação, caso a assembleia seja gravada por terceiros

**Texto da declaração**: "Declaro, sob minha responsabilidade, que participo desta assembleia na condição legítima vinculada à unidade informada e que as informações prestadas são verdadeiras."

> Isso ajuda a isentar a Master Síndico quanto à verificação material da identidade do votante.

---

## 5. Assembleia ao Vivo

### 5.1 Formas de check-in (3)
Cada assembleia possui QR code próprio.

1. **Pelo app** — morador faz check-in pelo próprio celular
2. **Por QR code** — escaneia o QR da assembleia e entra
3. **Por terminal de apoio na entrada** — tablet ou notebook de apoio. Morador informa: CPF, bloco, unidade. Sistema registra a presença digitalmente.

### 5.2 Voto presencial assistido
Quem entrou pelo terminal vota por **voto presencial assistido** — o voto será lançado no painel por síndico ou administradora.

Sistema registra: unidade, horário, operador responsável, tipo de voto (`presencial assistido`).

### 5.3 Tratamento do abandono da assembleia
**Regra de presença**: se morador faz check-in e sai, status:
- presente
- ausente momentâneo
- desconectado/saiu

**Regra de voto**:
- Votos já computados continuam válidos
- Itens ainda não votados ficam indisponíveis enquanto ele estiver ausente, **entra como abstenção**
- Se ele retornar, volta a poder votar nos itens ainda abertos

**Transparência**: telão e painel mostram: presentes, online ativos, ausentes/desconectados — evita fraude e mostra claramente a base votante atual.

### 5.4 Painel inicial do síndico
Antes do início, sistema exige cadastro de:
- Presidente da mesa
- Até 3 secretários
- Administradora presente: nome da administradora, nome, função, CPF
- Auditor (opcional): nome, função, CPF
- Fornecedores presentes (opcional): nome da empresa, representante

### 5.5 Papéis do presidente da mesa
- Abrir formalmente a assembleia
- Conduzir a ordem dos trabalhos
- Chamar itens da pauta
- Autorizar e ordenar falas
- Abrir e encerrar votação
- Declarar item resolvido, adiado ou prejudicado
- Encerrar discussão de item
- Autorizar reabertura de item, quando necessário
- Declarar resultado
- Autorizar encerramento formal da assembleia

### 5.6 Início da assembleia
Síndico/administradora clica em **Iniciar assembleia**.

Antes da abertura, **roda vídeo curto da Master Síndico** explicando regras de participação, fala, votação e deliberação.

### 5.7 Apresentação dentro da plataforma
Síndico, administradora ou auditor podem apresentar material sem sair da plataforma. **Disponibilizar opção do marco que o material deverá ser inserido.**

Formatos aceitos: PDF, imagem, vídeo. **PowerPoint deve ser convertido em PDF antes do upload.**

### 5.8 Estrutura do telão (2 áreas)
**Área 1 — Apresentação**: documentos de apoio (pptx, word, pdf, vídeos do síndico/condomínio/administradora/auditor).

**Área 2 — Painel operacional**:
- Pauta toda (com marcação do item em destaque)
- Unidades presentes
- ~~Unidades online (habilitar depois)~~
- Unidades ausentes
- Fila de mãos levantadas
- Cronômetro da fala
- Status dos itens
- Quórum presente
- Quórum necessário
- Relógio da assembleia

### 5.9 Status dos itens
Status possíveis: **resolvido / adiado / não resolvido / prejudicado**.

Cada item terá: hora de abertura, hora de entrada em votação, hora de encerramento, duração total da discussão.

### 5.10 Controle de fala (fila)
Tempo máximo de fala definido na criação da assembleia.

**Fluxo**:
1. Morador aperta "levantar mão"
2. Entra na fila
3. Pode abaixar a mão
4. Síndico ou presidente chama a unidade pelo microfone (microfone deve ser habilitado de forma alternada para melhor captação do áudio e transcrição da gravação)
5. Cronômetro inicia
6. **Apenas as falas microfonadas serão registradas em ata.** Orientar o síndico a falar isso antes da assembleia.

Painel do síndico mostra: fila de espera, tempo restante.

---

## 6. Votação (Req 57 spec viva)

### Tipos de voto
- Sim / Não / Abstenção (votação simples)
- Escala 1-5 + Abstenção (votação escalada)

### Regra de procuração
Procurador vota → voto replicado para unidade representada (considerando fração ideal).

### Telão durante votação
Pauta completa, item atual, unidades presentes, unidades ausentes, fila de fala, cronômetro, relógio assembleia, quórum presente, quórum necessário, **votos por unidade**.

### Resultado
Encerrar votação → **Homologação** (síndico OU administradora homologa) → finalização do item (resolvido / adiado / não resolvido / prejudicado).

---

## 7. Encerramento

Encerrar assembleia → **Registro final** com campos: data início, data término, presidente, secretários, administradora, auditor, fornecedores presentes, votos, procurações, inadimplentes, liberações de voto.

---

## 8. Pós-Assembleia e Histórico

### Relatórios (6 tipos)
- Presença
- Votos
- Procurações
- Trilha de auditoria
- Decisões
- Consolidado

### Linha do Tempo
Histórico inclui: assembleia · edital · pauta · resultados · relatórios.

---

## 9. Score de Transparência (Req 60.7)

10 componentes calculados após homologação:
1. Percentual de ciência
2. Percentual de presença
3. Percentual de votantes
4. Percentual de leitura da pauta
5. Percentual de visualização de documentos
6. Completude da trilha de auditoria
7. Regularidade de homologação
8. Cumprimento integral da pauta
9. Tempo médio por item
10. Percentual de itens com documentação prévia

Indicadores: convocação, participação, votação, documentação. Comparado com média da plataforma.

---

## 10. Modo Contingência (Req 60.10)

Ativação manual (síndico ou administradora). Razões: falha de internet, falha de dispositivo, votação digital indisponível.

Quando ativado: continuar assembleia com voto presencial assistido. Itens afetados marcados como "entrou em modo contingência".

Registra: hora de ativação, hora de normalização (se aplicável), ID do ativador.

---

## 11. NFR Críticos

- WebSocket latência **< 200ms (P95)** — obrigatório para check-in e votação real-time
- Fração ideal: **NUMERIC(5,4)** no PostgreSQL, nunca float
- Append-only para todos os eventos (voto, check-in, homologação)
- Auto-save em Redis: estado de assembly recuperável < 1min após crash

---

## 12. Diagrama Visual (resumo do PDF DIAGRAMA VISUAL)

```
DASHBOARD
  ↓
CRIAR ASSEMBLEIA
  ↓
CONFIGURAR (tipo, data, convocação, modalidade)
  ↓
VINCULAR ADMINISTRADORA (token de acesso)
  ↓
CADASTRAR PAUTA (itens, quórum, votação, anexos)
  ↓
ANEXAR EDITAL
  ↓
PUBLICAR ASSEMBLEIA
  ↓
[ETAPA 2 — PREPARAÇÃO]
  → NOTIFICAÇÃO MORADORES → REGISTRAR CIÊNCIA → CONFIRMAR PARTICIPAÇÃO
  → VISUALIZAÇÃO DA PAUTA
  → ENQUETES PRELIMINARES
  → CADASTRO DE PROCURAÇÃO
  → ANÁLISE DE FORNECEDORES
  ↓
[ETAPA 3 — PREPARAÇÃO OPERACIONAL]
  PAINEL ADMINISTRADORA → IMPORTAR FRAÇÃO IDEAL → REGISTRAR INADIMPLÊNCIA → VALIDAR PROCURAÇÕES
  SIMULADOR DE QUÓRUM
  ↓
[ETAPA 4 — DIA DA ASSEMBLEIA]
  CHECK-IN (App / QR Code / Terminal)
  PAINEL DO SÍNDICO (cadastrar presidente, secretários, administradora, auditor, fornecedores)
  INICIAR ASSEMBLEIA → VÍDEO INSTITUCIONAL → PAINEL OPERACIONAL
  CONDUÇÃO DA PAUTA: Item Atual → Discussão (Fila de Fala) → Votação
  CONTROLE DE FALA: Levantar Mão → Fila → Síndico Autoriza → Cronômetro
  VOTAÇÃO: voto APP / voto presencial assistido (Sim/Não/Abstenção OU Escala 1-5+Abstenção)
  PROCURAÇÃO: voto replicado considerando fração ideal
  TELÃO: pauta, item atual, unidades presentes/ausentes, fila, cronômetro, relógio, quórum, votos
  ENCERRAR VOTAÇÃO → HOMOLOGAÇÃO (síndico ou administradora)
  FINALIZAÇÃO DO ITEM: resolvido / adiado / não resolvido / prejudicado
  ENCERRAR ASSEMBLEIA → REGISTRO FINAL
  ↓
[PÓS-ASSEMBLEIA]
  RELATÓRIOS: presença, votos, procurações, trilha auditoria, decisões
  HISTÓRICO LINHA DO TEMPO: assembleia, edital, pauta, resultados, relatórios
```

---

## Referências

- [[../02-Jornadas/Inventario-Telas]] — outras telas do produto
- [[Compliance-Detalhado]] — auditoria também grava ações de assembleia
- [[../regras-de-negocio/quebras-detectadas]] Q27, Q40 — mudanças de escopo e regras não migradas
- `backend/.kiro/specs/master-sindico/requirements/assembly.md` — Req 48–60.12 (canônico)
