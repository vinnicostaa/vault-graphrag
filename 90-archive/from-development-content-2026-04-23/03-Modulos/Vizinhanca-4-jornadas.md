---
title: "Módulo Vizinhança — 4 Jornadas Integradas"
updated: 2026-04-22
maintainer: agent-3 (analista)
type: module
status: stable
source: Content/contents/MÓDULO VIZINHANÇA.ini, *MÓDULO VIZINHANÇA*.md, MÓDULO VIZINHANÇA - MORADOR.pdf, Jornada do Parceiro da Vizinhança.pdf
related-spec: backend/.kiro/specs/master-sindico/requirements/commercial.md (Req 27-27.3)
---

# Módulo Vizinhança — 4 Jornadas Integradas

Conecta condomínios e comércio local. **Decision 12**: Parceiro da Vizinhança é persona com login próprio, aceita CPF (CNPJ não obrigatório).

> **Persona delegada vs persona independente**: Parceiro da Vizinhança **É** uma persona independente com login (role `local_company`). Empresa síndica delegada e Agência de Marketing são actors delegados (sem tenant próprio).

---

## Mensagem institucional (cross-jornada)

> "O condomínio faz parte do bairro. A comunidade começa no condomínio e se estende pelo bairro."
>
> "A Master Síndico conecta você diretamente com moradores, síndicos e empresas da região."

---

## 1. Jornada do Parceiro (VZ1-VZ6) — 6 telas

| ID | Nome | Conteúdo |
|---|---|---|
| VZ1 | Cadastro do Parceiro | App→Vizinhança→"Quero ser parceiro". Campos obrigatórios: nome_estabelecimento, categoria_principal, subcategoria, telefone, whatsapp, email, cep, endereco, numero, bairro, cidade, descricao. Opcionais: instagram, logo. **Não exige CNPJ** (CPF basta). |
| VZ2 | Perfil do Parceiro | Seções: Promoções + Métricas. Botões: Criar promoção · Ver métricas · Editar perfil. |
| VZ3 | Criar Promoção | Campos: titulo, descricao, tipo (7 opções), validade (data início/término), termos. **Segmentação obrigatória**: aberta_bairro OR exclusiva_condominio. |
| VZ4 | Promoções Ativas | Lista: titulo, tipo, validade, visualizações, ativações. Editar/encerrar. |
| VZ5 | Métricas do Parceiro | Indicadores: visualizações_perfil, visualizações_promoções, ativações, cliques_em_contato. Gráficos: engajamento_semanal, promoções_mais_ativadas. |
| VZ6 | Histórico de Promoções | Lista: promoção, data_publicação, data_encerramento, número_ativações. |

### Regras Operacionais
- Pode: criar, editar, encerrar promoções
- **Não pode**: publicar comunicados em condomínios, registrar execução de serviços, acessar painel empresarial

### 7 Tipos de Benefício
```
desconto_percentual | desconto_fixo | produto_gratuito | combo_promocional
avaliacao_gratuita | brinde | beneficio_exclusivo
```

### 38+ Categorias de Estabelecimento
Ver [[../04-Listas-Mestre/Enums-Consolidados]] §12.

### Visibilidade da Promoção
- **Aberta para o bairro**: visível para moradores + síndicos + empresas da região
- **Exclusiva para condomínio**: visível APENAS para moradores do condomínio selecionado (síndico não vê, empresa não vê)

---

## 2. Jornada do Síndico (VS1-VS10) — 10 telas

Acesso via Governança → Vizinhança.

| ID | Nome | Conteúdo |
|---|---|---|
| VS1 | Painel da Vizinhança | 4 seções: explorar parceiros, promoções exclusivas do condomínio, parceiros indicados, histórico. Botões: Explorar · Indicar parceiro local · Criar benefício exclusivo. |
| VS2 | Explorar Parceiros | Feed de parceiros (filtros: categoria, bairro). Card: nome, categoria, bairro, promoção ativa. |
| VS3 | Detalhe do Parceiro | nome, categoria, endereço, telefone, descrição, promoções disponíveis. Botões: Ver promoção · Entrar em contato · Indicar para moradores. |
| VS4 | Detalhe da Promoção | titulo, descricao, tipo, validade. Botão: Ativar promoção. |
| VS5 | Ativar Promoção | "Apresente esta tela no estabelecimento para ativar seu benefício." Mostra: nome_estabelecimento, nome_promocao, validade. **A ativação gera métrica para o parceiro.** |
| VS6 | Indicar Parceiro Local | Síndico convida estabelecimento. Campos: nome, categoria, telefone, endereço, observação. **Sistema envia link para cadastro do parceiro.** |
| VS7 | Criar Benefício Exclusivo para Condomínio | Síndico cria promoção em parceria com estabelecimento. Selecionar parceiro · titulo · descricao · tipo (7 opções) · validade (início/término). **Promoção aparece apenas para moradores daquele condomínio.** |
| VS8 | Benefícios do Condomínio | Lista de promoções ativas: promoção, parceiro, validade, ativações. Editar/encerrar. |
| VS9 | Parceiros Indicados pelo Síndico | Status: convite_enviado · parceiro_cadastrado · parceiro_ativo |
| VS10 | Histórico de Promoções Ativadas | Lista: promoção, estabelecimento, data_ativação. Controle de promoções utilizadas. |

### Regras Operacionais
- Pode: explorar parceiros, ativar promoções, indicar parceiros, criar benefícios exclusivos
- **Não pode**: editar promoções criadas pelo parceiro

### Métricas para o Síndico (VS10)
- Número de parceiros indicados
- Promoções exclusivas ativas
- Ativações de benefícios

---

## 3. Jornada da Empresa (VE1-VE6) — 6 telas

Acesso via Painel Empresarial → Vizinhança.

| ID | Nome | Conteúdo |
|---|---|---|
| VE1 | Painel da Vizinhança para Empresas | 3 seções: explorar parceiros, promoções disponíveis, histórico. Botões: Explorar parceiros · Ver promoções. |
| VE2 | Explorar Parceiros | Feed (filtros: categoria, bairro). |
| VE3 | Detalhe do Parceiro | nome, categoria, endereço, telefone, descrição, promoções. Botões: Ver promoção · Entrar em contato. |
| VE4 | Detalhe da Promoção | titulo, descricao, tipo, validade. Botão: Ativar promoção. |
| VE5 | Ativar Promoção | Mesmo padrão de VS5 / VM4. **Métrica gerada para parceiro, separada por tipo de usuário (empresa).** |
| VE6 | Histórico de Promoções Utilizadas | Promoção, estabelecimento, data_ativação. |

### Regras Operacionais
- Pode: explorar, visualizar, ativar promoções, entrar em contato com parceiros
- **Não pode**: criar/editar promoções, cadastrar parceiros

### Visibilidade — regra dura
Empresa **NÃO** vê promoções exclusivas (só moradores). Empresa **só** vê promoções abertas (bairro).

---

## 4. Jornada do Morador (VM1-VM7) — 7 telas

Acesso via Painel do Morador → Vizinhança.

| ID | Nome | Conteúdo |
|---|---|---|
| VM1 | Painel/Feed do Bairro | Feed de parceiros priorizado por: same_neighborhood + active_promotions + sindico_indicated_partners. Filtro: categoria. |
| VM2 | Detalhe do Parceiro | nome, categoria, endereço, telefone, descrição, promoções. Botões: Ver promoção · Entrar em contato · Ir até local. |
| VM3 | Detalhe da Promoção | titulo, descricao, tipo, validade. Mensagem: "Esta promoção faz parte da rede de vizinhança da Master Síndico." Botão: Ativar promoção. |
| VM4 | Ativar Promoção | "Apresente esta tela no estabelecimento para ativar seu benefício." |
| VM5 | Benefícios do meu Condomínio | Promoções **exclusivas** criadas pelo síndico em parceria com estabelecimentos. Mensagem: "Alguns parceiros oferecem benefícios exclusivos para moradores do seu condomínio." |
| VM6 | Parceiros Indicados pelo Síndico | Lista com badge "Indicado pelo Síndico" — parceiros recomendados pela gestão. |
| VM7 | Histórico de Ativações | Promoção, estabelecimento, data_ativação. |

### Regras Operacionais
- Pode: visualizar parceiros, visualizar promoções, ativar promoções, ver parceiros indicados
- **Não pode**: publicar promoções, cadastrar parceiros

### Visibilidade
- Promoções **abertas** (bairro): visíveis para todos (moradores + síndicos + empresas)
- Promoções **exclusivas** (condomínio): visíveis APENAS para moradores do condomínio específico
- Badge "Indicado pelo Síndico" diferencia parceiros recomendados de descobertos

---

## Métricas Administrativas (Plataforma Master Síndico)

Para Admin MS acompanhar:
- Parceiros cadastrados
- Promoções ativas
- Promoções por bairro
- Engajamento médio
- Ativações separadas por: moradores · síndicos · empresas (3 contadores)

---

## Connect Me Síndico → Parceiro (Req 19.3)

Síndico pode também usar **Connect Me** para iniciar contato com Parceiro propondo promoção exclusiva (não apenas explorar feed). Ver [[Connect-Me-4-direcoes]] §4.

### Diferença
- **Connect Me Síndico→Parceiro**: contato direto via formulário (LGPD-compliant), parceiro responde com aceite ou contra-proposta
- **VS6 Indicar Parceiro Local**: síndico envia convite para estabelecimento se cadastrar (link para o flow VZ1)

---

## Implementação backend

### Tabelas
- `companies` (com `is_neighborhood=true` para Parceiro Vizinhança)
- `company_profiles`
- `neighborhood_partnerships`
- `neighborhood_benefits` (com `scope: aberta_bairro | exclusiva_condominio`)
- `neighborhood_visibility` (controle de quem vê o quê — para promoções exclusivas)
- `neighborhood_metrics` (ativações, views, cliques)
- `connect_me_neighborhood_requests`

### Eventos NATS
- `neighborhood.partner.registered`
- `neighborhood.benefit.created`
- `neighborhood.promotion.activated` (com `actor_type: morador | sindico | empresa` para métricas separadas)
- `neighborhood.exclusive_benefit.created` (visível só para moradores do condomínio)

### ABAC
- `commercial.neighborhood_benefit.create` — apenas role=local_company
- `commercial.neighborhood_benefit.activate` — qualquer role autenticada
- `commercial.exclusive_benefit.create` — role=syndic
- `commercial.exclusive_benefit.view` — apenas se `unit.condominium_id == benefit.condominium_id`

---

## Restrição cruzada (Decision 7)

**Parceiro da Vizinhança NÃO tem acesso à Academia LMS.** Botão "Academia" oculto na navegação. Ver [[../03-Modulos/LMS-Academia]] (a criar) e Req 98.1.

---

## Referências

- [[../02-Jornadas/Inventario-Telas]] §5-§8 (VZ1-VZ6, VS1-VS10, VE1-VE6, VM1-VM7)
- [[../04-Listas-Mestre/Enums-Consolidados]] §12 (categorias 38+) §13 (7 benefícios)
- [[Connect-Me-4-direcoes]] §4 (Síndico → Parceiro)
- `backend/.kiro/specs/master-sindico/requirements/commercial.md` Req 27-27.3 (canônico)
