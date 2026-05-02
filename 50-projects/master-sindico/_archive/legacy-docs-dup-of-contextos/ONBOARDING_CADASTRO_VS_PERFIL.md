# Onboarding - Cadastro vs Perfil (Explicitacao)

Data: 2026-02-06

Objetivo
- Definir explicitamente o que e CADASTRO (obrigatorio para ativacao) e o que e PERFIL (customizacao opcional).
- Garantir que dados de perfil sejam opcionais (nao bloqueiam ativacao), enquanto dados de cadastro bloqueiam.
- Alinhar com os documentos: `contextos/DADOS PARA CADASTRO.txt`, `contextos/novo escopo-7.pdf`, `contextos/Matriz Funcional.pdf`, `contextos/Manual Tecnico My sindico.pdf`.

Definicao base
- Usuario (User): identidade e acesso. Serve para login, seguranca, governanca. Campos minimos: nome, email, senha, role, emailVerified.
- Perfil (Profile): dados publicos e especificos da persona. Ex: documento, endereco profissional, categorias, certificacoes, fotos, videos, etc.

Regras gerais
- `role` e imutavel apos criacao.
- `documento` (CPF/CNPJ) e imutavel apos primeira definicao.
- Dados de PERFIL sao opcionais e nao devem deixar o perfil como incompleto.
- Status do perfil deve refletir somente ativacao e pagamento (quando aplicavel).

Estados sugeridos de perfil
- `incomplete`: faltam dados obrigatorios de CADASTRO para ativacao.
- `pending_payment`: cadastro obrigatorio ok, mas falta pagamento (quando aplicavel).
- `active`: cadastro obrigatorio ok e pagamento ok (se houver).
- `disabled`: bloqueio administrativo.

Opcional: Flag separado para visibilidade
- `visibilityReady` (boolean): true quando os dados de PERFIL minimos para aparecer em busca/visibilidade estiverem completos.
- Esse flag deve ir em users para poder ser otimizado nas buscas.
- Esse flag nao interfere no `status` de ativacao.

Cadastro minimo (primeira etapa)
- `name`, `email`, `password`, `role`.
- Dispara verificacao de email.
- Nao cria perfil ainda.

Onboarding (segunda etapa)
- Define documento (CPF/CNPJ) e dados obrigatorios de CADASTRO.
- Cria o perfil e seta status conforme regras.


## Mapeamento por persona

### Morador (Base / Morador Pagante)
Cadastro (obrigatorio)
- Nome completo (User)
- Email (User)
- Senha (User)
- CPF (Profile) - imutavel
- Data de nascimento (Profile)
- Telefone (Profile ou User, ver nota de contato abaixo)
- CEP (Profile)
- Condominio (nome ou codigo) (Profile)

Perfil (opcional / customizacao)
- Foto de perfil
- Endereco completo
- Video curriculo (quando aplicavel)

Visibilidade (minimo para aparecer)
- CPF + nascimento + condominio + endereco basico


### Sindico (N1 / N2 / N3)
Cadastro (obrigatorio)
- Razao social / Nome completo (User)
- Email profissional (User)
- Senha (User)
- Endereco profissional (Profile)
- Telefone (Profile ou User)
- Documento (CPF ou CNPJ) (Profile) - imutavel
- Data de nascimento (Profile)

Perfil (opcional / customizacao)
- Nome fantasia
- Cidade/Estado de atuacao
- Tipo de sindico (morador/profissional)
- Tempo de atuacao, qtd de condominios
- Certificacoes (ABRACS, seguro, LGPD, etc)
- Mini bio, video institucional, formacao, administradoras

Visibilidade (minimo para aparecer)
- Documento + tipo + cidade/estado + endereco profissional


### Empresas do Mercado Condominial (Emp. Plus/Pro)
Cadastro (obrigatorio)
- Razao social (User)
- Email do representante legal (User)
- Senha (User)
- CNPJ (Profile) - imutavel
- Data de aniversario da empresa (Profile)
- Representante legal (Profile)
- Email comercial (Profile)
- Telefone comercial (Profile)
- Endereco comercial com CEP (Profile)
- Financeiro (nome, telefone, email) (Profile)

Perfil (opcional / customizacao)
- Logo
- Nome fantasia
- Cidades/estados de atuacao
- Categorias e subcategorias
- Certificacoes (ISO, ESG, etc)
- Descricao institucional, videos, portfolio

Visibilidade (minimo para aparecer)
- CNPJ + endereco comercial + nome fantasia


### Parceiros da Vizinhanca (Comercio Local)
Cadastro (obrigatorio)
- Nome/razao social (User)
- Email do representante legal (User)
- Senha (User)
- CPF/CNPJ (Profile) - imutavel
- Data de aniversario da empresa (Profile)
- Representante legal (Profile)
- Financeiro (nome, telefone, email) (Profile)

Perfil (opcional / customizacao)
- Logo
- Nome fantasia
- Email comercial
- Telefone comercial
- Endereco comercial com CEP
- Ate 3 fotos

Visibilidade (minimo para aparecer)
- Documento + nome fantasia + endereco


### Empresas de Marketing
Cadastro (obrigatorio)
- Razao social (User)
- Email do representante legal (User)
- Senha (User)
- CNPJ (Profile) - imutavel
- Data de aniversario da empresa (Profile)
- Representante legal (Profile)
- Email comercial (Profile)
- Telefone comercial (Profile)
- Endereco comercial com CEP (Profile)
- Financeiro (nome, telefone, email) (Profile)
- Cidade/estado de atuacao (Profile)

Perfil (opcional / customizacao)
- Logo
- Nome fantasia
- Informacoes publicas adicionais

Visibilidade (minimo para aparecer)
- CNPJ + endereco + nome fantasia


## Alinhamento com o schema atual (Drizzle)
Observacao: hoje varias colunas nos perfis estao como NOT NULL. Para que PERFIL seja opcional, e necessario:
- ou permitir NULL nesses campos (migracao),
- ou manter NOT NULL mas nao criar o perfil ate ter os dados obrigatorios.

Sugestao pragmatica
- Criar o perfil somente quando o CADASTRO obrigatorio estiver completo.
- Campos de customizacao permanecem null/ausentes e nao bloqueiam ativacao.
- Usar `visibilityReady` para controlar busca/visibilidade.


## Logica de front-end (sem codigo)

Fluxo geral
1. Cadastro minimo (nome, email, senha, role)
2. Verificacao de email
3. Onboarding por etapa
   - Etapa A: Documento (CPF/CNPJ) e dados obrigatorios de CADASTRO
   - Etapa B: Pagamento (se aplicavel)
   - Etapa C: Customizacao de perfil (opcional)

Regra de bloqueio
- Usuario pode logar apos email verificado.
- Perfil so aparece em busca quando `visibilityReady = true`.
- Funcoes premium so liberam quando `status = active` e plano ok.

Exemplo de telas por etapa
- Tela 1: Verificar Email (bloqueante)
- Tela 2: Documento (CPF/CNPJ) e dados obrigatorios (bloqueante)
- Tela 3: Pagamento (bloqueante se plano pago)
- Tela 4: Customizar Perfil (nao bloqueante)


## Resposta sobre dados privados vs publicos
Sua decisao faz sentido:
- Dados privados (login, contato legal/administrativo) em Users.
- Dados publicos e de contato comercial em Profile.

Recomendacao adicional
- Mantenha um email de login exclusivo no User, mesmo que exista email comercial no Profile.
- Se precisar separar contato legal vs contato publico, use campos distintos no Profile para publico e no User para legal.

