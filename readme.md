# Prisma and ORM with Express

## 1. What are ORMs?

**Object-Relational Mappers (ORMs)** are libraries that let developers interact with relational databases using **object-oriented programming concepts** instead of raw SQL. They abstract database operations and provide a higher-level API.

### Core Features:

* **Object Mapping**: Tables map to classes, rows to objects.
* **CRUD Abstraction**: Perform Create, Read, Update, Delete without SQL.
* **Migrations**: Version control for database schema changes.
* **Query Builders**: Programmatically construct complex queries.
* **Validation**: Enforce data constraints at the application layer.
* **Relation Handling**: Fetch related data efficiently (eager/lazy loading).

---

## 2. Why Use ORMs?

* **Readable Syntax**: Database operations resemble object manipulation.
* **Database Independence**: Switch between databases with minimal code changes.
* **Type-Safety & Auto-Completion**: Reduces runtime errors, improves DX.
* **Built-in Tools**: Includes migrations, validations, query helpers, and relations.

---

## 3. What is Prisma?

**Prisma** is a next-gen ORM for JavaScript/TypeScript, designed for **type-safety, intuitive queries, and developer productivity**.

### Key Components:

1. **Prisma Schema** – declarative schema (`schema.prisma`) defining models, relations, and configs.
2. **Prisma Client** – auto-generated, type-safe client for queries.
3. **Prisma Migrate** – handles schema versioning & migrations.
4. **Prisma Studio** – GUI for managing database data.

### Example Schema:

```prisma
model User {
  id    Int    @id @default(autoincrement())
  name  String
  email String @unique
  posts Post[]
}

model Post {
  id       Int    @id @default(autoincrement())
  title    String
  content  String
  authorId Int
  author   User   @relation(fields: [authorId], references: [id])
}
```

---

## 4. Installing Prisma in a New Project

### Initialize Project

```bash
mkdir my-prisma-app && cd my-prisma-app
npm init -y
```

### Install Dependencies

```bash
npm install prisma --save-dev
npm install @prisma/client
```

### Setup Prisma

```bash
npx prisma init
```

This creates:

* `prisma/schema.prisma` – schema definition
* `.env` – database connection URL

### Configure Database

Example `.env`:

```env
DATABASE_URL="postgresql://USER:PASSWORD@HOST:PORT/DB"
```

(Supported: PostgreSQL, MySQL, SQLite, MongoDB, SQL Server)

### Define Models

```prisma
model User {
  id       Int    @id @default(autoincrement())
  username String @unique
  password String
  age      Int
  city     String
}
```

### Run Migration

```bash
npx prisma migrate dev --name init
```

### Generate Prisma Client

```bash
npx prisma generate
```

---

## 5. Using Prisma Client

Example CRUD operations:

```javascript
import { PrismaClient } from "@prisma/client";
const prisma = new PrismaClient();

// Create
await prisma.user.create({
  data: { username: "Alice", password: "1234", age: 20, city: "NY" },
});

// Read
const users = await prisma.user.findMany();

// Update
await prisma.user.update({
  where: { id: 1 },
  data: { username: "UpdatedName" },
});

// Delete
await prisma.user.delete({ where: { id: 1 } });
```

---

## 6. Relationships in Prisma

Prisma supports **1:1, 1:N, and N:M** relations.

Example 1:N relation:

```prisma
model User {
  id    Int    @id @default(autoincrement())
  name  String
  posts Post[]
}

model Post {
  id       Int  @id @default(autoincrement())
  title    String
  userId   Int
  user     User @relation(fields: [userId], references: [id])
}
```

Querying relations:

```javascript
const user = await prisma.user.findUnique({
  where: { id: 1 },
  include: { posts: true },
});
```

---

## 7. Express + Prisma Example

### Install Express

```bash
npm install express
npm install --save-dev @types/express
```

### Create Server

```javascript
import { PrismaClient } from "@prisma/client";
import express from "express";

const prisma = new PrismaClient();
const app = express();
app.use(express.json());

// Get all users
app.get("/users", async (req, res) => {
  const users = await prisma.user.findMany();
  res.json(users);
});

// Get user with todos
app.get("/users/:id", async (req, res) => {
  const id = parseInt(req.params.id);
  const user = await prisma.user.findFirst({
    where: { id },
    select: { username: true, password: true, todos: true },
  });
  res.json(user);
});

app.listen(3000, () => console.log("Server running on port 3000"));
```

---

## 8. Recommended Project Structure

```
my-prisma-app/
├── node_modules/
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   ├── dev.db (if SQLite)
├── src/
│   ├── index.ts (Express entrypoint)
│   ├── routes/
│   ├── controllers/
├── prisma/seed.ts
├── .env
├── package.json
```

---

## 9. Seeding Database

Create a `prisma/seed.ts` file to preload initial data:

```typescript
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

async function main() {
  await prisma.user.createMany({
    data: [
      { username: 'Alice', password: '1234', age: 20, city: 'New York' },
      { username: 'Bob', password: '5678', age: 25, city: 'San Francisco' },
    ],
  });
}

main()
  .catch(e => console.error(e))
  .finally(async () => await prisma.$disconnect());
```

Run the seed script:

```bash
ts-node prisma/seed.ts
```

---

## 10. Advanced Topics

* **Middleware Support**: Prisma allows custom middleware for logging, auth, or query validation.
* **Pagination & Filtering**: Built-in helpers for efficient queries.
* **Transactions**: Use `prisma.$transaction()` for atomic operations.
* **Connection Pooling**: Supported via external tools like PgBouncer or Prisma Accelerate.
* **Deployment**: Works seamlessly with services like Vercel, Railway, and Docker.

---

## 11. Resources

* [Prisma Docs](https://www.prisma.io/docs)
* [Express Docs](https://expressjs.com/)
* [Awesome Prisma](https://github.com/catalinmiron/awesome-prisma)


