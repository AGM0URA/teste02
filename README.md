# AS ORDENS 🛠️

Sistema de gerenciamento de chamados e ordens de serviço com interface web e API REST. Projeto acadêmico de Back-End Frameworks da UNINASSAU.

## Funcionalidades

- Autenticação JWT e perfis de administrador, agente e usuário.
- Abertura e acompanhamento de chamados, mensagens, anexos e avaliações.
- Gestão de usuários, equipes e categorias com SLA, relatórios e configurações.

## Tecnologias e estrutura

O front-end usa Vue 3, Vue Router, Vite e Axios. O back-end usa Java 17, Spring Boot, Spring Security, Spring Data JPA e Maven. O banco é PostgreSQL; os testes utilizam H2.

```text
asOrdensCompleto/
├── frontend/             # Interface Vue
├── backend/              # API Spring Boot e Maven Wrapper
│   └── migration/        # Migração do banco legado
└── docker-compose.yml    # Front-end, API e PostgreSQL
```

## Pré-requisitos

- Docker e Docker Compose para executar os serviços em containers ou apenas o banco.
- Para o front-end local: Node.js **^22.18.0 ou >=24.12.0** e npm.
- Para o back-end local: Java 17 ou superior e PostgreSQL. O Maven Wrapper incluído dispensa instalar o Maven.

Os comandos partem da raiz do projeto, salvo indicação contrária. Para parar um processo no terminal, use Ctrl+C.

## Executar tudo com Docker

Com o Docker em execução, na raiz:

```bash
docker compose up -d --build
```

| Serviço | Endereço |
| --- | --- |
| Interface | http://localhost:5173 |
| API | http://localhost:8080 |
| Swagger | http://localhost:8080/swagger-ui/index.html |
| Health check | http://localhost:8080/actuator/health |

Para acompanhar os logs ou parar os serviços:

```bash
docker compose logs -f
docker compose stop
```

O banco usa um volume persistente. Evite removê-lo sem um backup atualizado. A porta da interface pode ser alterada pela variável `FRONTEND_PORT`.

## Executar o front-end sozinho

Na raiz do projeto:

```bash
cd frontend
npm ci
npm run dev
```

Acesse o endereço informado pelo Vite, normalmente **http://localhost:5173**. A interface inicia sem o back-end, mas login e recursos que dependem da API precisam de uma API disponível.

Por padrão, as requisições para `/api` passam pelo proxy do Vite para `http://localhost:8080`. Para apontar o proxy para outro servidor, defina `API_PROXY_TARGET` antes de iniciar o Vite. Também é possível definir `VITE_API_URL` como URL base completa da API, por exemplo `http://localhost:8080/api`; nesse caso, confira a configuração de CORS do servidor de destino.

Para gerar e visualizar a versão de produção:

```bash
npm run build
npm run preview
```

## Executar o back-end sozinho

O back-end pode ser executado sem o front-end, mas precisa de um PostgreSQL acessível e da variável `JWT_SECRET`.

### 1. Prepare o banco

**Opção A — somente o banco no Docker:** na raiz, execute:

```bash
docker compose up -d db
```

Por padrão, isso disponibiliza o banco `chamados_db` em `localhost:5432`, com usuário `postgres` e senha `2030` em um volume novo. Se o volume já existir, as credenciais previamente inicializadas permanecem em uso.

**Opção B — PostgreSQL local:** crie um banco chamado `chamados_db` e ajuste usuário, senha e URL conforme seu ambiente. Os valores padrão da aplicação são os mesmos da opção A.

### 2. Configure e inicie a API

Defina uma chave JWT local de pelo menos 32 bytes e execute o Maven Wrapper. Os exemplos mostram como sobrescrever os dados do banco; ajuste-os se necessário. As variáveis valem para a sessão atual do terminal.

**Windows PowerShell:**

```powershell
cd backend
$env:JWT_SECRET = "chave-local-de-demonstracao-com-mais-de-32-bytes"
$env:SPRING_DATASOURCE_URL = "jdbc:postgresql://localhost:5432/chamados_db"
$env:SPRING_DATASOURCE_USERNAME = "postgres"
$env:SPRING_DATASOURCE_PASSWORD = "2030"
.\mvnw.cmd spring-boot:run
```

**Linux/macOS:**

```bash
cd backend
export JWT_SECRET='chave-local-de-demonstracao-com-mais-de-32-bytes'
export SPRING_DATASOURCE_URL='jdbc:postgresql://localhost:5432/chamados_db'
export SPRING_DATASOURCE_USERNAME='postgres'
export SPRING_DATASOURCE_PASSWORD='2030'
./mvnw spring-boot:run
```

A API estará em **http://localhost:8080**. Acesse o [Swagger](http://localhost:8080/swagger-ui/index.html) ou o [health check](http://localhost:8080/actuator/health). Para usar front e back localmente, mantenha cada um aberto em um terminal separado.

### Variáveis de ambiente

| Variável | Uso | Padrão local |
| --- | --- | --- |
| `JWT_SECRET` | Chave de assinatura dos tokens; obrigatória para a API local | Sem padrão local |
| `SPRING_DATASOURCE_URL` | URL JDBC do PostgreSQL | `jdbc:postgresql://localhost:5432/chamados_db` |
| `SPRING_DATASOURCE_USERNAME` | Usuário do banco | `postgres` |
| `SPRING_DATASOURCE_PASSWORD` | Senha do banco | `2030` |
| `API_PROXY_TARGET` | Destino do proxy do Vite | `http://localhost:8080` localmente |
| `VITE_API_URL` | URL base do cliente HTTP do front-end | `/api` |
| `POSTGRES_PASSWORD` | Senha inicial do PostgreSQL no Compose | `2030` |
| `FRONTEND_PORT` | Porta publicada pelo Compose para o front-end | `5173` |

No Compose, defina `JWT_SECRET` no ambiente antes de iniciar os serviços. Use uma chave própria e credenciais adequadas fora do ambiente local; não versione segredos reais. Variáveis em um arquivo `.env` na raiz são lidas pelo Compose, mas não são carregadas automaticamente pelo comando local do Spring Boot.

## Acesso inicial

Quando não há usuários cadastrados, a inicialização cria estas contas de demonstração:

| Perfil | E-mail | Senha |
| --- | --- | --- |
| Administrador | `admin@asordens.com` | `Admin123` |
| Agente | `agente@asordens.com` | `Agente123` |

Troque as senhas antes de usar dados reais. Se já houver usuários no banco, as contas acima não são recriadas automaticamente. A inicialização também cria uma equipe e categorias de exemplo quando as respectivas tabelas estão vazias.

## API

A documentação interativa está em [Swagger UI](http://localhost:8080/swagger-ui/index.html), com o documento OpenAPI em `http://localhost:8080/v3/api-docs`. Entre as rotas estão:

| Método | Rota | Finalidade |
| --- | --- | --- |
| `POST` | `/api/auth/login` | Autenticação e obtenção de token |
| `POST` | `/api/auth/register` | Cadastro de usuário |
| `GET`, `POST` | `/api/chamados` | Consulta e abertura de chamados |
| `PATCH` | `/api/chamados/{id}/assumir` | Assumir atendimento |
| `PATCH` | `/api/chamados/{id}/finalizar` | Finalizar atendimento |
| `POST` | `/api/chamados/{id}/interacoes` | Adicionar interação |
| `POST` | `/api/chamados/{id}/avaliacao` | Avaliar atendimento |
| `GET` | `/api/relatorios/chamados` | Consultar relatório |

Para rotas protegidas, envie `Authorization: Bearer <token>`; as permissões dependem do perfil do usuário.

## Testes do back-end

Na pasta `backend/`, execute:

```powershell
# Windows
.\mvnw.cmd test
```

```bash
# Linux/macOS
./mvnw test
```

Os testes usam H2 em memória conforme a configuração de teste do projeto.

## Banco legado

O script `backend/migration/migrar-legado.sql` registra a migração específica do esquema antigo. Nomes históricos de solicitantes e agentes foram associados a contas inativas sem senha de acesso; categorias históricas ausentes receberam SLA inicial de 24 horas, que deve ser revisado. O backup local `asordens-antes-integracao.dump`, quando presente, é ignorado pelo Git.

## Integrantes

- Jorge Vilela
- André Gomes Moura
- Lucas Matheus Pereira da Silva
- Bernardo Augusto Franco
- Kauã Nyllo

