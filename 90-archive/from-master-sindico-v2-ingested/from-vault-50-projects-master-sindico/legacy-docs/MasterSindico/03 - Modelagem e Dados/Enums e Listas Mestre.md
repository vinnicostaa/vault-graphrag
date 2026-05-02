---
title: "Enums e Listas Mestre"
version: "3.0"
date: 2026-03-10
status: "concluido"
tags:
  - mastersindico
  - referencia
  - enums
  - banco-de-dados
---

# 📚 Enums e Listas Mestre

> Dicionário de dados fixo da plataforma. Valores aqui documentados devem ser mapeados diretamente como `enums` no Drizzle ORM do PostgreSQL.
> **Breadcrumbs:** [[00 - Índice Master Síndico|🏠 Início]] > 📊 Modelagem e Dados > 📄 Enums e Listas Mestre

---

## 🏢 Gestão de Condomínio e Mandatos

### Tipos de Condomínio
`residencial`, `comercial`, `misto`, `shopping_center`, `galeria_comercial`, `edificio_garagem`

### Tipos de Unidade
`residencial`, `comercial`, `vaga_garagem`

### Status da Unidade
`ocupada_proprietario`, `ocupada_inquilino`, `unidade_vazia`

### Tipos de Vínculo de Morador
`proprietario_residente`, `proprietario_nao_residente`, `inquilino`

### Motivo Fim de Vínculo (Morador)
`mudanca_definitiva`, `venda_unidade`, `encerramento_contrato_locacao`, `erro_cadastro`

### CNH (Carteira Nacional de Habilitação)
`nao`, `A`, `B`, `AB`

### Tipos de Mandato (Síndico)
`sindico_eleito`, `sindico_profissional`, `sindico_interino`

### Motivo Fim de Mandato
`fim_periodo`, `renuncia`, `destituicao`, `transferencia`

---

## 🛠️ Plano Diretor e Atividades Técnicas

### Áreas Impactadas (26 itens)
`estrutura_predial`, `sistema_hidraulico`, `sistema_eletrico`, `sistema_incendio`, `reservatorios`, `elevadores`, `garagem`, `fachada`, `cobertura`, `seguranca_patrimonial`, `administracao`, `portaria`, `playground`, `academia`, `salao_festas`, `corredores`, `jardins`, `casa_maquinas`, `rede_esgoto`, `rede_drenagem`, `sistema_gas`, `iluminacao`, `sistema_cameras`, `controle_acesso`, `areas_comuns`, `outros`

### Tipos de Atividade (26 itens)
`manutencao_preventiva`, `manutencao_corretiva`, `inspecao_tecnica`, `vistoria_tecnica`, `contratacao_servico`, `execucao_servico`, `reparo_emergencial`, `obra_melhoria`, `adequacao_normativa`, `atualizacao_infraestrutura`, `monitoramento_ambiental`, `revisao_tecnica`, `auditoria_tecnica`, `limpeza_tecnica`, `controle_pragas`, `limpeza_reservatorios`, `treinamento_tecnico`, `adequacao_legal`, `revisao_equipamentos`, `atualizacao_administrativa`, `contratacao_fornecedor`, `encerramento_contrato`, `avaliacao_fornecedor`, `implantacao_sistema`, `atualizacao_normas`, `acao_emergencial`

### Natureza do Serviço
`preventiva`, `corretiva`, `emergencial`, `estrategica`

### Riscos Associados
`sem_risco`, `risco_operacional`, `risco_estrutural`, `risco_sanitario`, `risco_eletrico`, `risco_hidraulico`, `risco_incendio`, `risco_juridico`, `risco_ambiental`

### Classificação de Risco (Geral)
`baixo`, `moderado`, `elevado`, `critico`

### Próxima Ação (Plano Diretor)
`monitorar_evolucao`, `nova_inspecao`, `contratar_fornecedor`, `executar_manutencao`, `atualizar_plano_diretor`, `registrar_conclusao`, `acompanhar_garantia`, `reavaliar_necessidade`

### Status do Plano Diretor
`planejada`, `em_contratacao`, `em_execucao`, `concluida`, `suspensa`, `reprogramada`, `atrasada`

---

## 🤝 Comercial e Connect Me

### Motivo de Recusa (Empresa → Connect Me)
`fora_area_atendimento`, `servico_fora_especialidade`, `agenda_indisponivel`, `conflito_servicos_agendados`, `orcamento_incompativel`, `outro_motivo`

### Critérios de Avaliação (Síndico avalia Empresa, 1 a 10)
`qualidade_servico`, `cumprimento_prazo`, `atendimento`, `custo_beneficio`, `recomendacao`

### Status do Registro de Execução (Empresa)
`rascunho`, `enviado`, `aguardando_validacao`, `solicitado_ajuste`, `aprovado`, `publicado`, `rejeitado`

### Tipos de Empresa/Agência
**Tipos de Usuários Empresa:** `administrador`, `operador_tecnico`
**Status da Agência:** `convite_enviado`, `ativa`, `encerrada`
**Tipos de Suporte Marketing:** `producao_videos_institucionais`, `gestao_conteudos_tecnicos`, `estrategia_posicionamento`, `gestao_redes_sociais`, `consultoria_marketing`, `outro`
**Marketing Link Priority:** `imediato`, `proximos_30_dias`, `sem_prazo_definido`
**Marketing Link Status:** `sent`, `viewed`, `proposal_sent`, `atendido`, `accepted`, `rejected`

---

## 📺 Conteúdo e Comunicação

### Entradas na Linha do Tempo (Timeline Types)
`atividade_da_gestao`, `registro_de_execucao`, `comunicado`, `evento`, `adendo`, `resultado_consulta_moradores`, `assembly_result`

### Tipos de Evonto
`assembleia_ordinaria`, `assembleia_extraordinaria`, `manutencao_programada`, `inspecao_tecnica`, `obra_programada`, `interrupcao_programada_servicos`, `treinamento_equipe`, `campanha_institucional`, `evento_comunitario`, `reuniao_administrativa`, `atualizacao_normativa`, `simulado_emergencia`, `outros`

### Tipos de Comunicado
`aviso_operacional`, `interrupcao_programada`, `orientacao_moradores`, `aviso_seguranca`, `comunicado_institucional`, `conclusao_servico`, `recomendacao_manutencao`, `alerta_preventivo`

### Tipos de Comunicado Pós-Negócio (Empresa)
`orientacao_tecnica`, `recomendacao_manutencao`, `alerta_preventivo`, `conclusao_servico`, `atualizacao_garantia`

### Tipos de Vídeo Institucional
`apresentacao_institucional`, `apresentacao_equipe_tecnica`, `demonstracao_servico`, `explicacao_tecnica`, `boas_praticas_condominiais`, `orientacao_preventiva`, `caso_real_atendimento`, `treinamento_tecnico`, `explicacao_legislacao_normas`, `dicas_manutencao`, `conteudo_educativo_sindicos`, `conteudo_educativo_moradores`

---

## 🏛️ Assembleia Ao Vivo

### Participantes da Assembleia (Confirmação Prévia)
`participarei_presencialmente`, `participarei_online`, `ainda_nao_defini`, `nao_participarei`

### Tipos de Assembleia
`ordinaria`, `extraordinaria`, `implantacao`

### Elementos da Pauta (Assembly Items)
`informativo`, `votacao_simples`, `votacao_fracao_ideal`, `escolha_fornecedor`, `eleicao`

### Tipos de Quórum
`simple_majority`, `qualified_majority`, `unanimous`

### Opções de Voto
`favor`, `contra`, `abstencao`

### Opções de Tempo de Fala (Timer)
`2_minutos`, `3_minutos`

---

## 🎫 Parceiro da Vizinhança (Descontos)

### Categorias de Parceiro (Por Grupo)
* **Alimentação:** `padaria`, `restaurante`, `lanchonete`, `cafeteria`, `pizzaria`, `hamburgueria`, `sorveteria`, `doceria`, `acai`
* **Mercado:** `supermercado`, `hortifruti`, `adega`
* **Saúde e Pet:** `farmacia`, `pet_shop`, `clinica_veterinaria`, `banho_tosa`
* **Academia:** `academia`, `crossfit`, `pilates`, `yoga`
* **Estética:** `salao_beleza`, `barbearia`, `spa`
* **Serviços:** `lavanderia`, `costureira`, `sapateiro`, `eletricista`, `encanador`, `chaveiro`, `marceneiro`, `pintor`, `assistencia_celular`, `assistencia_computadores`, `oficina`, `lava_jato`
* **Educação/Outros:** `cursos_idiomas`, `reforco_escolar`, `outros_servicos`

### Status do Convite de Parceiro
`convite_enviado`, `parceiro_cadastrado`, `parceiro_ativo`

### Tipos de Benefício / Promoção
`desconto_percentual`, `desconto_fixo`, `produto_gratuito`, `combo_promocional`, `avaliacao_gratuita`, `brinde`, `beneficio_exclusivo`

---

## 🎓 Academia LMS

### Categorias de Curso e Fórum
`gestao_condominial`, `saude_ambiental`, `seguranca_predial`, `manutencao_predial`, `legislacao_condominial`, `mediacao_conflitos`, `tecnologia_condominios`, `sustentabilidade`, `administracao_financeira`, `outros`, `convivencia_moradores` *(Apenas Fórum)*, `seguranca` *(Apenas Fórum)*

### Níveis de Curso
`basico`, `intermediario`, `avancado`

### Tipos de Lives e Eventos ao Vivo
`debates_condominiais`, `lancamento_solucoes`, `palestras_tecnicas`, `entrevistas_especialistas`

### Acesso a Lives
`abertas` (todos do LMS), `exclusivas` (baseado em plano/convite)

### Tipos de Conteúdo na Biblioteca
`videos`, `guias_tecnicos`, `manuais`, `livros_digitais`, `ebooks`, `materiais_tecnicos`

---

## 🔒 Acesso e Plataforma

### Planos
`trial`, `base`, `plus`, `pro`

### Roles de Usuário (ABAC Base)
`sindico`, `morador_titular`, `morador_dependente`, `empresa_administrador`, `empresa_operador`, `agencia`

### Compliance Status Global
`em_dia`, `pendente`, `atrasado`

### Modalidades de Contratação (Banco de Talentos)
`clt`, `pj`, `estagio`, `temporario`, `folguista`, `diarista`, `freelancer`, `intermitente`, `outros`
