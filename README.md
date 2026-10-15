# API de Agendamentos

API REST simples de agendamentos em **Node.js + TypeScript + Express 5**, usando **PostgreSQL**, organizada no **Service Pattern** (Controller → Service → Repository) e empacotada em uma imagem Docker **multi-stage** mínima.

## Arquitetura

```
src/
├── config/        # variáveis de ambiente
├── database/      # pool de conexões + migração automática
├── models/        # tipos de domínio
├── repositories/  # acesso a dados (somente SQL)
├── services/      # regras de negócio (sem HTTP, sem SQL)
├── controllers/   # camada HTTP (valida entrada, chama service)
├── validators/    # schemas Zod
├── routes/        # rotas + injeção de dependências
├── middlewares/   # tratamento global de erros
├── errors/        # AppError
├── app.ts         # configuração do Express
└── server.ts      # bootstrap + graceful shutdown
```

| Camada | Responsabilidade |
|---|---|
| **Controller** | Recebe `req`, valida com Zod, devolve `res` |
| **Service** | Regras: horário futuro, término > início, sem conflito, transições de status |
| **Repository** | Queries SQL parametrizadas e mapeamento `snake_case` ↔ `camelCase` |

Conflitos de horário são checados no service **e** garantidos no banco por uma constraint `EXCLUDE USING gist` (protege contra race conditions).

## Executando com Docker

```bash
docker compose up -d --build
curl http://localhost:3000/health
```

## Executando localmente

```bash
cp .env.example .env
docker compose up -d db
npm install
npm run dev
```

## Endpoints

| Método | Rota | Descrição |
|---|---|---|
| GET | `/api-docs` (ou `/docs`) | Documentação interativa Swagger UI |
| GET | `/api-docs/json` | Especificação OpenAPI 3.0 em JSON |
| GET | `/health` | Status da API e do banco |
| POST | `/appointments` | Cria agendamento |
| GET | `/appointments?status=&from=&to=` | Lista (filtros opcionais) |
| GET | `/appointments/:id` | Detalha |
| PUT | `/appointments/:id` | Atualiza (parcial) |
| PATCH | `/appointments/:id/cancel` | Cancela |
| PATCH | `/appointments/:id/complete` | Conclui |
| DELETE | `/appointments/:id` | Remove |

### Exemplo

```bash
curl -X POST http://localhost:3000/appointments \
  -H "Content-Type: application/json" \
  -d '{
    "clientName": "Maria Silva",
    "clientEmail": "maria@email.com",
    "service": "Corte de cabelo",
    "startsAt": "2030-01-10T14:00:00-03:00",
    "endsAt":   "2030-01-10T15:00:00-03:00",
    "notes": "Primeira visita"
  }'
```

### Códigos de resposta

| Código | Quando |
|---|---|
| 400 | Body/params inválidos |
| 404 | Agendamento não encontrado |
| 409 | Conflito de horário |
| 422 | Regra de negócio violada (horário passado, status inválido...) |

## Imagem Docker

O `Dockerfile` usa 2 estágios:

1. **build** – instala as dependências e gera, com **esbuild**, um único `server.js` minificado (express, pg e zod embutidos).
2. **final** – `node:22-alpine` com apenas o `server.js`. Sem `node_modules` nem código-fonte; roda com o usuário não-root `node`.

```bash
docker images api-agendamentos
```

---

## Deploy e validação prática | DevOps

Esta seção documenta as atividades práticas de conteinerização, publicação e hospedagem realizadas a partir do projeto-base acadêmico.

### Infraestrutura publicada

| Componente | Tecnologia | Hospedagem |
|---|---|---|
| Front-end | React, TypeScript, Vite e Nginx | Render |
| Back-end | Node.js, TypeScript e Express | Render |
| Banco de dados | PostgreSQL | Render |
| Imagens | Docker | Docker Hub |
| Código-fonte | Git | GitHub |

### Links da aplicação

- [Aplicação web](https://agendamentos-frontend-fkxd.onrender.com)
- [API - Health Check](https://api-agendamentos-flpb.onrender.com/health)
- [Documentação Swagger](https://api-agendamentos-flpb.onrender.com/api-docs)
- [Repositório do front-end](https://github.com/alexsabrasil/agendamentos-frontend)
- [Imagem Docker do back-end](https://hub.docker.com/r/alexsasec/api-agendamentos)
- [Imagem Docker do front-end](https://hub.docker.com/r/alexsasec/agendamentos-frontend)

### Etapas executadas

1. Preparação de uma cópia independente do projeto-base.
2. Instalação de dependências e validação do TypeScript.
3. Compilação da aplicação e execução dos testes automatizados.
4. Construção e teste local da imagem Docker.
5. Criação de repositório independente no GitHub.
6. Publicação da imagem no Docker Hub.
7. Provisionamento do PostgreSQL no Render.
8. Deploy da API e configuração das variáveis de ambiente.
9. Publicação do front-end React com Nginx.
10. Validação funcional da integração entre os serviços.

### Testes realizados

- 34 testes automatizados aprovados no back-end.
- Build e verificação de tipos concluídos.
- Auditoria das dependências de produção sem vulnerabilidades identificadas.
- Health check da API com conexão ao PostgreSQL.
- Documentação Swagger acessível.
- Cadastro, alteração de status, cancelamento e conclusão de agendamentos.
- Persistência dos registros após atualização da página.

### Competências praticadas

Docker, Docker Hub, Git, GitHub CLI, Node.js, TypeScript, PostgreSQL, Nginx, Render, testes automatizados e deploy de aplicações.

### Próximas evoluções

- Implementar pipeline CI/CD.
- Adicionar verificações SAST e SCA.
- Ampliar testes de integração e segurança.
- Configurar monitoramento e observabilidade.
- Aprimorar controles de autenticação e autorização.

### Contexto acadêmico

Projeto-base disponibilizado pelo professor e utilizado em atividade prática de hospedagem. A preparação dos repositórios independentes, a conteinerização, a publicação das imagens, o provisionamento da infraestrutura e a validação da aplicação foram realizados como parte da prática de DevOps.

**Nota:** a hospedagem utiliza recursos gratuitos do Render, sujeitos a limitações e expiração do banco de dados de demonstração.