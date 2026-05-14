# TF 11 - CRUD de Cursos com Sequelize ORM

Documentação completa do código implementado para o exercício TF 11.

---

## 📋 Índice

1. [Model](#model)
2. [Controllers](#controllers)
3. [Router](#router)
4. [Resumo da Implementação](#resumo-da-implementação)

---

## Model

### `app/Models/CourseModel.js`

```javascript
import { DataTypes } from "sequelize";
import sequelize from "../../database/connections/sequelize.js";

const CourseModel = sequelize.define(
    "CourseModel",
    {
        id: {
            type: DataTypes.INTEGER,
            primaryKey: true,
            autoIncrement: true
        },

        name: {
            type: DataTypes.TEXT,
            allowNull: false
        },

        professor: {
            type: DataTypes.TEXT,
            allowNull: false
        }
    },
    {
        tableName: "courses",
        timestamps: true,
        createdAt: 'created_at',
        updatedAt: 'updated_at'
    }
);

export default CourseModel;
```

**Descrição**: Define a estrutura da tabela `courses` no banco de dados usando Sequelize ORM.

---

## Controllers

### 1. ListCourseController

**Arquivo**: `app/Controllers/CourseApi/ListCourseController.js`

```javascript
import CourseModel from "../../Models/CourseModel.js";

export default async function ListCourseController(request, response) {
    try {
        const pageRequest = Number(request.query.page) || 1;
        const limitRequest = Number(request.query.limit) || 10;

        const page = pageRequest < 1 ? 1 : pageRequest;
        const limit = limitRequest > 20 ? 20 : limitRequest < 1 ? 10 : limitRequest;
        const offset = (page - 1) * limit;

        const { rows, count: total } = await CourseModel.findAndCountAll({
            order: [["id", "ASC"]],
            limit: limit + 1,
            offset: offset,
            distinct: true
        });

        const courses = rows;
        let next = null;

        if (courses.length > limit) {
            next = page + 1;
            courses.pop();
        }

        return response.json({
            page: page,
            limit: limit,
            total: total,
            next: next,
            data: courses
        });
    } catch (error) {
        console.error(error);

        return response.status(500).json({
            error: "Internal server error"
        });
    }
}
```

**Rota**: `GET /courses`

**Funcionalidades**:
- Lista cursos com paginação
- Parâmetros: `page` (padrão: 1), `limit` (padrão: 10, máximo: 20)
- Retorna total de registros e indicador de próxima página

**Exemplo de Uso**:
```bash
curl http://localhost:3000/courses?page=1&limit=10
```

---

### 2. GetCourseController

**Arquivo**: `app/Controllers/CourseApi/GetCourseController.js`

```javascript
import CourseModel from "../../Models/CourseModel.js";

export default async function GetCourseController(request, response) {
    try {
        const { id } = request.params;

        const course = await CourseModel.findByPk(id);

        if (!course) {
            return response.status(404).json({
                error: "Course not found"
            });
        }

        return response.json(course);
    } catch (error) {
        console.error(error);

        return response.status(500).json({
            error: "Internal server error"
        });
    }
}
```

**Rota**: `GET /courses/:id`

**Funcionalidades**:
- Busca um curso pelo ID
- Retorna 404 se não encontrado

**Exemplo de Uso**:
```bash
curl http://localhost:3000/courses/1
```

---

### 3. CreateCourseController

**Arquivo**: `app/Controllers/CourseApi/CreateCourseController.js`

```javascript
import CourseModel from "../../Models/CourseModel.js";

export default async function CreateCourseController(request, response) {
    try {
        const { name, professor } = request.body;
        const error = [];

        if (!name) {
            error.push("name obrigatório!");
        }

        if (!professor) {
            error.push("professor obrigatório!");
        }

        if (error.length > 0) {
            return response.status(400).json({ error: error });
        }

        const course = await CourseModel.create({
            name: name,
            professor: professor
        });

        return response.status(201).json(course);
    } catch (error) {
        console.error(error);

        return response.status(500).json({
            error: "Internal server error"
        });
    }
}
```

**Rota**: `POST /courses`

**Funcionalidades**:
- Cria um novo curso
- Valida campos obrigatórios: `name` e `professor`
- Retorna status 201 (Created) em caso de sucesso

**Exemplo de Uso**:
```bash
curl -X POST http://localhost:3000/courses \
  -H "Content-Type: application/json" \
  -d '{"name": "Desenvolvimento Web", "professor": "Prof. Silva"}'
```

---

### 4. UpdateCourseController

**Arquivo**: `app/Controllers/CourseApi/UpdateCourseController.js`

```javascript
import CourseModel from "../../Models/CourseModel.js";

export default async function UpdateCourseController(request, response) {
    try {
        const { id } = request.params;
        const { name, professor } = request.body;

        if (!name || !professor) {
            return response.status(400).json({
                error: "Name and professor are required"
            });
        }

        const course = await CourseModel.findByPk(id);

        if (!course) {
            return response.status(404).json({
                error: "Course not found"
            });
        }

        course.name = name;
        course.professor = professor;

        await course.save();

        return response.json(course);
    } catch (error) {
        console.error(error);

        return response.status(500).json({
            error: "Internal server error"
        });
    }
}
```

**Rota**: `PUT /courses/:id`

**Funcionalidades**:
- Atualiza um curso existente
- Valida campos obrigatórios
- Retorna 404 se curso não encontrado

**Exemplo de Uso**:
```bash
curl -X PUT http://localhost:3000/courses/1 \
  -H "Content-Type: application/json" \
  -d '{"name": "Web Avançado", "professor": "Prof. Costa"}'
```

---

### 5. DeleteCourseController

**Arquivo**: `app/Controllers/CourseApi/DeleteCourseController.js`

```javascript
import CourseModel from "../../Models/CourseModel.js";

export default async function DeleteCourseController(request, response) {
    try {
        const { id } = request.params;

        const course = await CourseModel.findByPk(id);

        if (!course) {
            return response.status(404).json({
                error: "Course not found"
            });
        }

        await course.destroy();

        return response.status(204).send();
    } catch (error) {
        console.error(error);

        return response.status(500).json({
            error: "Internal server error"
        });
    }
}
```

**Rota**: `DELETE /courses/:id`

**Funcionalidades**:
- Excluir um curso existente
- Retorna status 204 (No Content) em caso de sucesso
- Retorna 404 se curso não encontrado

**Exemplo de Uso**:
```bash
curl -X DELETE http://localhost:3000/courses/1
```

---

## Router

### `routes/apis/courseRouter.js`

```javascript
import { Router } from 'express';

import ListCourseController from '../../app/Controllers/CourseApi/ListCourseController.js';
import GetCourseController from '../../app/Controllers/CourseApi/GetCourseController.js';
import CreateCourseController from '../../app/Controllers/CourseApi/CreateCourseController.js';
import UpdateCourseController from '../../app/Controllers/CourseApi/UpdateCourseController.js';
import DeleteCourseController from '../../app/Controllers/CourseApi/DeleteCourseController.js';

export default (() => {
    const router = Router();

    /** TF 11 */
    router.get('/', ListCourseController);
    router.post('/', CreateCourseController);
    router.get('/:id', GetCourseController);
    router.put('/:id', UpdateCourseController);
    router.delete('/:id', DeleteCourseController);

    return router;
})();
```

**Descrição**: Centraliza todas as rotas de cursos e as associa aos controladores correspondentes.

**Rota Base**: `/courses` (registrada em `routes/router.js`)

---

## Resumo da Implementação

### ✅ Checklist de Requisitos

- [x] CRUD completo (Create, Read, Update, Delete)
- [x] Uso obrigatório de ORM Sequelize
- [x] Nenhum SQL manual nos controladores
- [x] Modelo dedicado: `CourseModel.js`
- [x] Controladores separados: 5 arquivos
- [x] Roteador centralizado: `courseRouter.js`
- [x] Padrão REST implementado
- [x] Tratamento de erros com try/catch
- [x] Validação de campos obrigatórios
- [x] Paginação em listagem
- [x] Status HTTP corretos (201, 204, 400, 404, 500)

### 📊 Estrutura de Dados

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `id` | INTEGER | Chave primária, auto-incremento |
| `name` | TEXT | Nome do curso (obrigatório) |
| `professor` | TEXT | Nome do professor (obrigatório) |
| `created_at` | TIMESTAMP | Data de criação (automático) |
| `updated_at` | TIMESTAMP | Data de última atualização (automático) |

### 🔗 Integração com o Projeto

O roteador `courseRouter.js` é registrado em `routes/router.js`:

```javascript
import courseRouter from './apis/courseRouter.js';
// ...
router.use("/courses", courseRouter);
```

Todos os controllers usam o mesmo padrão implementado em `UserApi` (TF anterior).

### 📚 Relacionamentos

Conforme definido em `database/relations.js`:

```javascript
UserModel.belongsToMany(CourseModel, {
    through: CourseUserModel,
    foreignKey: "id_user",
    otherKey: "id_course",
    as: "courses"
});

CourseModel.belongsToMany(UserModel, {
    through: CourseUserModel,
    foreignKey: "id_course",
    otherKey: "id_user",
    as: "users"
});
```

Courses e Users possuem relacionamento **many-to-many** através da tabela `course_users`.

---

## 🚀 Como Utilizar

1. Certifique-se de que o servidor está rodando:
   ```bash
   node _web.js
   ```

2. Teste as rotas usando curl, Postman ou qualquer cliente HTTP.

3. Veja exemplos de uso no arquivo [README.md](./readme.md#exercício-tf-11--crud-de-cursos-implementado).

---

**Data de Conclusão**: 14 de maio de 2026  
**Status**: ✅ Concluído e Testado
