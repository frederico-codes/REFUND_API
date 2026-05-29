# 💸 Refund API

API REST para gerenciar solicitações de reembolso de despesas. Desenvolvida com **Node.js**, **TypeScript**, **Express** e **Prisma**, a aplicação inclui:

- Autenticação JWT
- Controle de acesso por cargos (`employee`, `manager`)
- Upload e validação de comprovantes de despesa
- Persistência de dados com **SQLite** via **Prisma ORM**
- Validação de entrada com **Zod**
- Tratamento centralizado de erros

---

## 🚀 Funcionalidades principais

- Cadastro de usuários com cargo e criptografia de senha
- Autenticação e emissão de token JWT
- Criação de solicitações de reembolso associadas ao usuário
- Consulta de solicitações para gestores
- Upload de arquivos de comprovante com validação de tipo e tamanho
- Rotas protegidas por middleware de autenticação e autorização
- Persistência com banco local SQLite e esquema definido em Prisma

---

## 🧱 Arquitetura e organização

O projeto adota uma estrutura modular com separação clara entre:

- `src/routes/` — definição de rotas públicas e privadas
- `src/controllers/` — lógica de aplicação para cada recurso
- `src/middlewares/` — autenticação, autorização e tratamento de erros
- `src/configs/` — configurações de upload e JWT
- `src/database/` — inicialização do Prisma
- `src/providers/` — abstração de armazenamento em disco
- `src/utils/` — classes utilitárias e tratamento de exceções

---

## 🧾 Banco de dados

A aplicação usa SQLite com o arquivo de dados gerado por Prisma em `prisma/dev.db`.

### Modelo de dados (Prisma)

```prisma
model User {
  id        String   @id @default(uuid())
  name      String
  email     String   @unique
  password  String
  role      UserRole @default(employee)
  refunds   Refunds[]
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime? @updatedAt @map("updated_at")

  @@map("users")
}

model Refunds {
  id        String   @id @default(uuid())
  name      String
  amount    Float
  category  Category
  filename  String
  userId    String   @map("user_id")
  user      User     @relation(fields: [userId], references:[id])
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime? @updatedAt @map("updated_at")

  @@map("refunds")
}

enum UserRole {
  employee
  manager
}

enum Category {
  food
  others
  services
  transport
  accommodation
}
```

---

## 📦 Tecnologias

- Node.js
- TypeScript
- Express
- Prisma
- SQLite
- JSON Web Token (JWT)
- Bcrypt
- Multer
- Zod
- express-async-errors

---

## ⚙️ Instalação

```bash
npm install
```

### Inicializar Prisma

```bash
npx prisma generate
npx prisma migrate dev --name init
```



## ▶️ Executando em modo de desenvolvimento

```bash
npm run dev
```

O servidor será iniciado na porta `3333`.

---

## 🌐 Endpoints

### Usuários

- `POST /users`
  - Registra um novo usuário
  - Payload:
    - `name` (string)
    - `email` (string)
    - `password` (string)
    - `role` (`employee` | `manager`)

### Sessões

- `POST /sessions`
  - Autentica usuário e retorna token JWT
  - Payload:
    - `email` (string)
    - `password` (string)

### Reembolsos

> Essas rotas exigem cabeçalho `Authorization: Bearer <token>`.

- `POST /refunds`
  - Só para `employee`
  - Cria nova solicitação de reembolso
  - Payload:
    - `name` (string)
    - `category` (`food`, `others`, `services`, `transport`, `accommodation`)
    - `amount` (number)
    - `filename` (string) — nome do arquivo enviado via upload

- `GET /refunds`
  - Só para `manager`
  - Lista solicitações paginadas
  - Query params:
    - `name` (string, opcional)
    - `page` (number, padrão `1`)
    - `perPage` (number, padrão `2`)

- `GET /refunds/:id`
  - Acessível para `employee` e `manager`
  - Retorna os dados de uma solicitação específica

### Upload de comprovantes

- `POST /uploads`
  - Só para `employee`
  - Recebe arquivo no campo `file`
  - Tipos aceitos: `image/jpeg`, `image/jpg`, `image/png`
  - Tamanho máximo: `3MB`
  - Retorna JSON com o nome do arquivo salvo

---

## 🔐 Autenticação e autorização

- `ensureAuthenticated` valida o JWT no cabeçalho `Authorization`
- `verifyUserAuthorization` valida o `role` do usuário
- O token inclui `sub` com o ID do usuário e `role` do usuário

---

## 📂 Estrutura de pastas importantes

- `src/app.ts` — configura o Express e middlewares
- `src/server.ts` — inicia o servidor
- `src/routes/` — define rotas da API
- `src/controllers/` — implementa regras de negócio
- `src/middlewares/` — autenticação, autorização e erros
- `src/configs/` — configurações de upload e JWT
- `src/database/prisma.ts` — exporta o cliente Prisma
- `src/providers/disk-storage.ts` — gerencia uploads em disco
- `uploads/` — arquivos definitivamente salvos
- `tmp/` — uploads temporários antes de processamento

---

## 💡 Observações

- A chave JWT está atualmente definida em `src/configs/auth.ts`. Em produção, mova essa variável para um arquivo de ambiente seguro.
- O caminho público de arquivos está disponível em `/uploads`.
- A aplicação foi feita para ser usada como base de um serviço de reembolso, com validações de usuário, upload e papel.

---

## 📝 Licença

Projeto com licença `ISC`.
