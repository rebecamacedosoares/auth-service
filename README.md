# Microsserviço de Autenticação - Auth Service

Este é um microsserviço centralizado, responsável unicamente por cadastro, login e gerenciamento de tokens de autenticação (JWT) para um ecossistema de aplicações.

## Stack de Tecnologias

* **Framework:** Flask
* **Banco de Dados:** PostgreSQL
* **Autenticação:** Flask-JWT-Extended (Access Tokens & Refresh Tokens)
* **ORM:** Flask-SQLAlchemy
* **Migrações de DB:** Flask-Migrate (Alembic)
* **Containerização:** Docker & Docker Compose

## Configuração e Execução do Ambiente

O projeto containerizado. Certifique-se de ter o **Docker** instalado.

**1. Clone o Repositório**
```bash
git clone <URL_DO_SEU_REPOSITORIO>
cd auth-service
```

**2. Configure as Variáveis de Ambiente**
Crie um arquivo chamado `.env` na raiz do projeto. Você pode copiar o arquivo de exemplo `.env.example` (se existir) ou usar o template abaixo.

```env
# /auth-service/.env

FLASK_APP="run.py"
SECRET_KEY="gere_uma_chave_segura_aqui"
JWT_SECRET_KEY="gere_outra_chave_segura_aqui"

POSTGRES_USER=user
POSTGRES_PASSWORD=password
POSTGRES_DB=auth_service_db

DATABASE_URL="postgresql://user:password@postgres:5432/auth_service_db"
```

**3. Construa e Inicie os Contêineres**
Este comando irá construir a imagem da aplicação Flask e iniciar os contêineres da API e do banco de dados.

```bash
docker-compose up -d --build
```

**4. Execute as Migrações do Banco de Dados (Primeira Vez)**
Com os contêineres rodando, execute os seguintes comandos em ordem para inicializar o banco de dados e criar as tabelas pela primeira vez.


**# 1. Inicializa o sistema de migrações (só precisa ser rodado uma vez)**
```bash
docker-compose exec auth-api flask db init
```

**# 2. Cria o primeiro script de migração baseado nos seus modelos - Criação da estrutura inicial do banco de dados**
```bash
docker-compose exec auth-api flask db migrate -m
```

**# 3. Aplica a migração ao banco de dados, criando as tabelas**
```bash
docker-compose exec auth-api flask db upgrade
```

O serviço estará disponível em `http://localhost:5000`.

## Documentação da API

### `POST /auth/register`
Registra um novo usuário.

* **Body (JSON):**
    ```json
    {
        "username": "meu-usuario",
        "email": "teste@email.com",
        "password": "senha123"
    }
    ```
* **Resposta de Sucesso (201):**
    ```json
    {
        "msg": "Usuário criado com sucesso"
    }
    ```

### `POST /auth/login`
Autentica um usuário e retorna um par de tokens.

* **Body (JSON):**
    ```json
    {
        "email": "teste@email.com",
        "password": "senha123"
    }
    ```
* **Resposta de Sucesso (200):**
    ```json
    {
        "access_token": "eyJ...",
        "refresh_token": "eyJ..."
    }
    ```

### `GET /auth/profile`
Retorna as informações do usuário autenticado.

* **Autenticação:** Requer um `access_token` válido no header `Authorization: Bearer <token>`.
* **Resposta de Sucesso (200):**
    ```json
    {
        "id": 1,
        "username": "meu-usuario",
        "email": "teste@email.com"
    }
    ```

### `POST /auth/refresh`
Gera um novo `access_token` usando um `refresh_token` válido.

* **Autenticação:** Requer um `refresh_token` válido no header `Authorization: Bearer <token>`.
* **Resposta de Sucesso (200):**
    ```json
    {
        "access_token": "eyJ_um_novo_token_de_acesso"
    }
    ```