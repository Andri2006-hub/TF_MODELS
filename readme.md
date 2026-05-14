# Projeto Desenvolvimento Web - Bimestre 2

## Instalação e Execução

Siga os passos abaixo para rodar o projeto localmente:

---

## ⚙️ Configuração da Aplicação

1. Clonar o repositório:

```sh
git clone https://github.com/luan-tavares/unifaat-2026-dw-project
```

2. Entrar na pasta do projeto:

```sh
cd unifaat-2026-dw-project
```

3. Instalar as dependências:

```sh
npm install
```

4. Copiar o arquivo `.env` (**escolha apenas um, dependendo do seu sistema**):

Linux / Mac:
```sh
cp .env.example .env
```

Windows (CMD):
```sh
copy .env.example .env
```

5. Editar o arquivo `.env` e definir a senha do banco (**ALTERE AQUI**):

```env
POSTGRES_HOST=localhost
POSTGRES_DB=unifaat_dw
POSTGRES_PORT=6789
POSTGRES_USER=unifaat_user
POSTGRES_PASSWORD=**COLOQUE_SUA_SENHA_AQUI**

NODE_WEB_PORT=3000
```

---

## 🚀 Servidor Backend Node

6. Iniciar o servidor:

```sh
node _web.js
```

O servidor estará disponível em: http://localhost:3000

---

## 🐳 Docker

Após configurar o `.env`, basta subir os containers:

```sh
docker compose up
```

O servidor web estará disponível em: http://localhost:8080

---

## 🔄 Nodemon (Opcional)

Para desenvolvimento com reload automático:

Global:
```sh
npm install -g nodemon
nodemon _web.js
```

Local:
```sh
npm install --save-dev nodemon
./node_modules/.bin/nodemon _web.js
```

---

## 🧭 Estrutura do Projeto

- `app/`
  - Regras de negócio da aplicação.
  - `app/Controllers/`: controllers que tratam as rotas.

- `bootstrap/`
  - Inicialização da aplicação.
  - `app.js` e `config.js`.

- `config/`
  - Arquivos de configuração.

- `database/`
  - Conexões com banco de dados.
  - `database/connections/`: conexão com Postgres.

- `docker/`
  - Configurações de containers.
  - `docker/postgres/init`: scripts de inicialização do banco.

- `public/`
  - Arquivos estáticos.

- `routes/`
  - Definição das rotas HTTP.

- `storage/`
  - Armazenamento de arquivos/dados.

- `_web.js`
  - Entrypoint da aplicação.

- `.env`
  - Variáveis de ambiente.

- `.env.example`
  - Exemplo de variáveis.

- `docker-compose.yml`
  - Orquestração dos containers.

- `package.json`
  - Dependências e scripts.

---

## ✅ Exercício TF 11 — CRUD de Cursos Implementado

A solução implementa o CRUD completo da tabela `courses` usando **ORM Sequelize** e seguindo o padrão **REST**.

### 📋 Especificação

- **Objetivo**: Praticar a criação de uma API REST completa usando model ORM
- **Requisito obrigatório**: Usar Sequelize ORM (sem SQL manual nos controladores)
- **Modelo**: `./app/Models/CourseModel.js`
- **Estrutura de dados**: 
  - `id` (INTEGER, PK, auto-increment)
  - `name` (TEXT, obrigatório)
  - `professor` (TEXT, obrigatório)
  - `created_at` (TIMESTAMP)
  - `updated_at` (TIMESTAMP)

### 📁 Arquivos Implementados

**Controllers:**
- `app/Controllers/CourseApi/ListCourseController.js` — Listar cursos com paginação
- `app/Controllers/CourseApi/GetCourseController.js` — Buscar curso por ID
- `app/Controllers/CourseApi/CreateCourseController.js` — Criar novo curso
- `app/Controllers/CourseApi/UpdateCourseController.js` — Atualizar curso por ID
- `app/Controllers/CourseApi/DeleteCourseController.js` — Excluir curso por ID

**Roteador:**
- `routes/apis/courseRouter.js` — Centraliza todas as rotas de cursos

### 🔌 Rotas REST Disponíveis

| Método | Rota | Função |
|--------|------|--------|
| `GET` | `/courses` | Listar cursos (com paginação) |
| `POST` | `/courses` | Criar novo curso |
| `GET` | `/courses/:id` | Obter curso por ID |
| `PUT` | `/courses/:id` | Atualizar curso por ID |
| `DELETE` | `/courses/:id` | Excluir curso por ID |

### 🧪 Exemplos de Uso

**Listar cursos:**
```bash
curl http://localhost:3000/courses
# Com paginação:
curl http://localhost:3000/courses?page=1&limit=10
```

**Criar curso:**
```bash
curl -X POST http://localhost:3000/courses \
  -H "Content-Type: application/json" \
  -d '{"name": "Desenvolvimento Web", "professor": "Prof. Silva"}'
```

**Obter curso específico:**
```bash
curl http://localhost:3000/courses/1
```

**Atualizar curso:**
```bash
curl -X PUT http://localhost:3000/courses/1 \
  -H "Content-Type: application/json" \
  -d '{"name": "Web Avançado", "professor": "Prof. Costa"}'
```

**Excluir curso:**
```bash
curl -X DELETE http://localhost:3000/courses/1
```

### 📝 Padrão de Resposta

**Sucesso (GET):**
```json
{
  "id": 1,
  "name": "Desenvolvimento Web",
  "professor": "Prof. Silva",
  "created_at": "2026-05-14T10:30:00.000Z",
  "updated_at": "2026-05-14T10:30:00.000Z"
}
```

**Lista com paginação (GET /courses):**
```json
{
  "page": 1,
  "limit": 10,
  "total": 5,
  "next": null,
  "data": [...]
}
```

**Erro (validation):**
```json
{
  "error": ["name obrigatório!", "professor obrigatório!"]
}
```

**Erro (not found):**
```json
{
  "error": "Course not found"
}
```

### ✨ Características da Implementação

- ✅ Usa exclusivamente ORM Sequelize (sem SQL manual)
- ✅ Segue o padrão REST com status HTTP corretos
- ✅ Validação de campos obrigatórios
- ✅ Tratamento de erros com try/catch
- ✅ Paginação com limit/offset
- ✅ Sincronização com timestamps `created_at` e `updated_at`
- ✅ Integração com relacionamento many-to-many (users ↔ courses)

## 📦 Containers Docker

| Container           | Host            | Porta Interna | Porta Externa (localhost) |
|--------------------|-----------------|---------------|---------------|
| postgres-container | postgres_host   | 5432          | 6789          |
| nginx-container | nginx-container   | 80          | 8080          |
| nodeweb-container | nodeweb_host   | 3000          | -         |