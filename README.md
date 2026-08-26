# Expense Management API — Progress Notes

### Overall project plan

| Step   | What we do                  | Goal                          |
| ------ | --------------------------- | ----------------------------- |
| **1**  | Create NestJS project       | Basic application             |
| **2**  | Install & initialize Prisma | Add ORM                       |
| **3**  | Connect Prisma to MySQL     | Database connection           |
| **4**  | Create Prisma schema        | Define User, Project, Expense |
| **5**  | Run migration               | Create tables in MySQL        |
| **6**  | Create PrismaService        | Use Prisma inside NestJS      |
| **7**  | Create Users module         | Users CRUD                    |
| **8**  | Create Projects module      | Projects CRUD                 |
| **9**  | Create Expenses module      | Expenses CRUD                 |
| **10** | Add/handle relationships    | Expense ↔ User/Project        |
| **11** | Add DTOs + validation       | Validate requests             |
| **12** | Add error handling          | Proper 400/404 responses      |
| **13** | Test everything             | Postman/REST Client           |
| **14** | Clean/refactor              | Professional structure        |
| **15** | README + presentation       | Prepare for interview         |

---

# Step 1 — Create NestJS project

### Create project

```bash
nest new expense-management
```

Choose **npm**.

### Run the project

```bash
npm run start:dev
```

The NestJS application runs on:

```text
http://localhost:3000
```

Default response:

```text
Hello World!
```

Required NestJS packages 
```
npm install @nestjs/common @nestjs/core
```
### Generate NestJS components

Example:

```bash
nest generate module users
nest generate service users
nest generate controller users
```

Short form:

```bash
nest g module users
nest g service users
nest g controller users
```

---

# Step 2 — Install & initialize Prisma

### Install Prisma

```bash
npm install prisma @prisma/client
```

### Initialize Prisma

```bash
npx prisma init
```

This creates:

```text
prisma/
├── schema.prisma

.env
prisma.config.ts
```
if here we have problem with version prisma 8 we can downgrade to 7.9.1 like below:
```
npm install prisma@7.9.1 @prisma/client@7.9.1
```

We are using **Prisma 7.9.1**.

---

# Step 3 — Connect Prisma to MySQL

## Install MySQL if necessary

```bash
sudo apt update
sudo apt install mysql-server
```

## Open MySQL

```bash
sudo mysql
```

## Create database

```sql
CREATE DATABASE expense_management;
```

Check:

```sql
SHOW DATABASES;
```

## Create project-specific MySQL user

```sql
CREATE USER 'isaac'@'localhost' IDENTIFIED BY 'your_password';
```

Give access to the project database:

```sql
GRANT ALL PRIVILEGES ON expense_management.* TO 'isaac'@'localhost';
```

Apply privileges:

```sql
FLUSH PRIVILEGES;
```

Exit:

```sql
EXIT;
```

### Test the user

```bash
mysql -u isaac -p
```

---

# Step 4 — Configure Prisma

### `.env`

Configure your database URL:

```env
DATABASE_URL="mysql://isaac:your_password@localhost:3306/expense_management"
```

### `prisma/schema.prisma`

Because we are using **Prisma 7**, the datasource is:

```prisma
datasource db {
  provider = "mysql"
}
```

The database URL is handled through the Prisma configuration.

---

# Step 5 — Create the database schema

We need exactly **3 main models**:

```text
User
 └── id
 └── name
 └── email
 └── expenses[]

Project
 └── id
 └── name
 └── expenses[]

Expense
 └── id
 └── amount
 └── description
 └── userId ───→ User
 └── projectId ─→ Project
```

Put this inside:

```text
prisma/schema.prisma
```

```prisma
generator client {
  provider     = "prisma-client"
  output       = "../generated/prisma"
  moduleFormat = "cjs"
}

datasource db {
  provider = "mysql"
}

model User {
  id       Int       @id @default(autoincrement())
  name     String
  email    String    @unique
  expenses Expense[]
}

model Project {
  id       Int       @id @default(autoincrement())
  name     String
  expenses Expense[]
}

model Expense {
  id          Int     @id @default(autoincrement())
  amount      Float
  description String?
  userId      Int
  projectId   Int

  user        User    @relation(fields: [userId], references: [id])
  project     Project @relation(fields: [projectId], references: [id])
}
```

### Important

`moduleFormat = "cjs"` is needed for our current NestJS/Prisma setup to avoid the:

```text
ReferenceError: exports is not defined in ES module scope
```

error we encountered.

---

# Step 5.1 — Prisma Migration

We initially tried:

```bash
npx prisma db pull
```

This was **wrong for our situation** because `expense_management` was an empty database.

### Difference

```text
db pull
    ↓
Existing MySQL tables → Prisma schema
```

We need:

```text
Prisma schema → MySQL tables
```

Therefore we use:

```bash
npx prisma migrate dev --name init
```

### Shadow database permission

Prisma Migrate needs to temporarily create a **shadow database**.

Give the project user permission:

```bash
sudo mysql
```

```sql
GRANT CREATE, DROP ON *.* TO 'isaac'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Then:

```bash
npx prisma migrate dev --name init
```

This creates the actual tables in:

```text
expense_management
```

and Prisma's temporary shadow database during migration.

---

# Step 6 — Connect Prisma to NestJS

## Generate Prisma Client

```bash
npx prisma generate
```

With our configuration:

```prisma
output = "../generated/prisma"
```

Prisma generates:

```text
generated/
└── prisma/
    ├── client.ts
    ├── browser.ts
    ├── models.ts
    ├── enums.ts
    └── ...
```

**Important:** Prisma 7's generated client is in our custom `generated/prisma` directory, not the old `@prisma/client` location.

---

## Install MySQL adapter

Prisma 7 uses a driver adapter for this setup:

```bash
npm install @prisma/adapter-mariadb
```

---

## Create Prisma module and service

```bash
nest g module prisma
nest g service prisma
```

Creates:

```text
src/prisma/
├── prisma.module.ts
└── prisma.service.ts
```

---

# Step 6.1 — Configure `PrismaService`

`src/prisma/prisma.service.ts`:

```typescript
import { Injectable } from '@nestjs/common';
import { PrismaClient } from '../../generated/prisma/client';
import { PrismaMariaDb } from '@prisma/adapter-mariadb';

@Injectable()
export class PrismaService extends PrismaClient {
  constructor() {
    const adapter = new PrismaMariaDb({
      host: 'localhost',
      port: 3306,
      user: 'isaac',
      password: 'your_password',
      database: 'expense_management',
    });

    super({ adapter });
  }
}
```

Replace:

```text
your_password
```

with your actual MySQL password.

---

# Step 6.2 — Configure `PrismaModule`

`src/prisma/prisma.module.ts`:

```typescript
import { Global, Module } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

`@Global()` means `PrismaService` can be used throughout the application without importing `PrismaModule` into every feature module.

---

# Step 6.3 — Import PrismaModule

In:

```text
src/app.module.ts
```

```typescript
import { Module } from '@nestjs/common';
import { PrismaModule } from './prisma/prisma.module';

@Module({
  imports: [PrismaModule],
})
export class AppModule {}
```

---

# Step 6.4 — Verify everything

Run:

```bash
npm run start:dev
```

At this point we want:

```text
Found 0 errors.
```

and NestJS should start successfully.

---

# Current status

You have completed:

```text
✅ NestJS project
✅ MySQL
✅ Database
✅ Prisma 7
✅ Prisma schema
✅ User/Project/Expense models
✅ Relationships
✅ Prisma migration
✅ Prisma Client generation
✅ MySQL adapter
✅ PrismaService
✅ PrismaModule
```
---
### Next step

**Step 7 — Users CRUD**

We'll build:

```text
POST   /users
GET    /users
GET    /users/:id
PATCH  /users/:id
DELETE /users/:id
```
---
## Notes — After Step 6

### Step 7 — Users CRUD

**Generate Users files:**

```bash
nest g module users
nest g controller users
nest g service users
```

Creates:

```text
src/users/
├── users.module.ts
├── users.controller.ts
└── users.service.ts
```

### `UsersService`

The service contains the **database logic**.

```typescript
constructor(private prisma: PrismaService) {}
```

→ Gives the service access to Prisma/MySQL.

Implemented:

```text
createUser()  → CREATE user
findall()     → GET all users
findOne()     → GET one user
updateUser()  → UPDATE user
deleteUser()  → DELETE user
```

Using Prisma:

```typescript
this.prisma.user.create()
this.prisma.user.findMany()
this.prisma.user.findUnique()
this.prisma.user.update()
this.prisma.user.delete()
```

### `UsersController`

The controller handles **HTTP requests**.

```text
POST   /users
GET    /users
GET    /users/:id
PATCH  /users/:id
DELETE /users/:id
```

Important decorators:

| Code                   | Meaning             |
| ---------------------- | ------------------- |
| `@Controller('users')` | Base URL = `/users` |
| `@Post()`              | POST request        |
| `@Get()`               | GET request         |
| `@Get(':id')`          | GET with ID         |
| `@Patch(':id')`        | Update with ID      |
| `@Delete(':id')`       | Delete with ID      |
| `@Body()`              | Gets JSON body      |
| `@Param('id')`         | Gets ID from URL    |

### Important HTTP detail

URL parameters arrive as **strings**:

```typescript
@Param('id') id: string
```

So convert them:

```typescript
Number(id)
```

Example:

```typescript
@Get(':id')
async findOne(@Param('id') id: string) {
    return this.usersService.findOne(Number(id));
}
```

### Testing

You are using **Postman inside VS Code**.

Example:

```http
POST http://localhost:3000/users
```

Body:

```json
{
  "name": "Isaac",
  "email": "isaac@example.com"
}
```

Then:

```http
GET http://localhost:3000/users
```

to verify the user was saved.

### Current architecture

```text
Postman
   ↓
UsersController
   ↓
UsersService
   ↓
PrismaService
   ↓
MySQL
```

### Current status

```text
✅ NestJS
✅ MySQL
✅ Prisma 7
✅ Database schema
✅ Migration
✅ PrismaService
✅ Users module
✅ Users CRUD
✅ Postman testing
```

**Next: Step 8 — Projects CRUD.**
---
-------------------------------------------------------

## Notes — From Step 7 to Now

### Step 7 — Users CRUD

Generated:

```bash
nest g module users
nest g controller users
nest g service users
```

Created:

```text
src/users/
├── users.module.ts
├── users.controller.ts
└── users.service.ts
```

**UsersService** handles database operations:

```text
createUser() → CREATE
findall()    → READ all
findOne()    → READ one
updateUser() → UPDATE
deleteUser() → DELETE
```

**UsersController** exposes:

```text
POST   /users
GET    /users
GET    /users/:id
PATCH  /users/:id
DELETE /users/:id
```

Important:

```typescript
@Param('id') id: string
```

because URL parameters are strings, then:

```typescript
Number(id)
```

converts them to numbers.

---

### Step 8 — Projects CRUD

Generated:

```bash
nest g module projects
nest g controller projects
nest g service projects
```

Created the same CRUD structure:

```text
POST   /projects
GET    /projects
GET    /projects/:id
PATCH  /projects/:id
DELETE /projects/:id
```

Tested with Postman.

---

### Step 9 — Expenses CRUD

Generated:

```bash
nest g module expenses
nest g controller expenses
nest g service expenses
```

Created:

```text
POST   /expenses
GET    /expenses
GET    /expenses/:id
PATCH  /expenses/:id
DELETE /expenses/:id
```

Expense contains:

```text
amount
description
userId
projectId
```

Example:

```json
{
  "amount": 50.5,
  "description": "Restaurant",
  "userId": 1,
  "projectId": 1
}
```

---

### Step 10 — Relationships

Our Prisma relationships are:

```text
User 1 ────── N Expense N ────── 1 Project
```

Updated `ExpensesService` to include related data:

```typescript
include: {
  user: true,
  project: true,
}
```

Therefore:

```text
GET /expenses
```

returns the expense **plus its User and Project**.

Having multiple expenses with the same `userId` and `projectId` is correct.

---

### Step 11 — DTOs + Validation

Installed:

```bash
npm install class-validator class-transformer
```

Enabled global validation in `main.ts`:

```typescript
app.useGlobalPipes(new ValidationPipe());
```

Created DTOs:

```text
src/users/dto/
├── create-user.dto.ts
└── update-user.dto.ts

src/projects/dto/
├── create-project.dto.ts
└── update-project.dto.ts

src/expenses/dto/
├── create-expense.dto.ts
└── update-expense.dto.ts
```

Validation examples:

```typescript
@IsString()
@IsNotEmpty()
name!: string;
```

```typescript
@IsEmail()
email!: string;
```

```typescript
@IsNumber()
@Min(0)
amount!: number;
```

For PATCH, fields are optional:

```typescript
@IsOptional()
```

### DTO purpose

```text
Postman
   ↓
DTO
   ↓
ValidationPipe
   ↓
Controller
   ↓
Service
   ↓
Prisma
   ↓
MySQL
```

Invalid input → **400 Bad Request**.

---

## Current Status

```text
✅ NestJS
✅ MySQL
✅ Prisma 7
✅ Prisma schema + relationships
✅ Migration
✅ PrismaService
✅ Users CRUD
✅ Projects CRUD
✅ Expenses CRUD
✅ Postman testing
✅ Relationships included
✅ DTOs
✅ Request validation
```

### Next Step — Step 12

**Error handling**

We'll handle cases such as:

```text
GET /users/999
→ 404 Not Found

DELETE /expenses/999
→ 404 Not Found

Duplicate email
→ proper error response
```

Then we'll have a much more professional API.
---
## Short Note — Steps 12–16

### Step 12 — Error Handling

Added:

```text
404 → resource doesn't exist
409 → duplicate email
```

Used:

```typescript
NotFoundException
ConflictException
```

Before updating/deleting:

```typescript
const user = await prisma.user.findUnique({ where: { id } });

if (!user) {
  throw new NotFoundException('User not found');
}
```

Before creating an Expense, verify:

```text
userId   → User exists
projectId → Project exists
```

---

### Step 13 — Testing

Tested all:

```text
Users CRUD
Projects CRUD
Expenses CRUD
Validation
Relationships
404 / 409 errors
```

---

### Step 14 — Refactoring

Changed Controllers to use DTOs:

```text
Controller
   ↓
DTO + Validation
   ↓
Service
   ↓
Prisma
   ↓
MySQL
```

Removed unused `.spec.ts` files.

---

### Step 15 — Final Review

Verified:

```bash
npm run start:dev
npx prisma validate
npx prisma generate
```

Everything works with **0 TypeScript errors**.

---

### Step 16 — README

Added:

```text
Project description
Technologies
Architecture
Database relationships
API endpoints
Validation
Error handling
Setup instructions
```

### Current Status

```text
✅ CRUD
✅ MySQL + Prisma
✅ Relationships
✅ DTOs + Validation
✅ Error Handling
✅ Testing
✅ Refactoring
✅ README
```

**Next → Interview preparation.**
---
users
┌────┬───────┬──────────────────┐
│ id │ name  │ email            │
├────┼───────┼──────────────────┤
│ 1  │ Isaac │ isaac@example.com│
└────┴───────┴──────────────────┘

projects
┌────┬────────────────────┐
│ id │ name               │
├────┼────────────────────┤
│ 1  │ Web Development    │
└────┴────────────────────┘

expenses
┌────┬────────┬────────┬───────────┐
│ id │ amount │ userId │ projectId │
├────┼────────┼────────┼───────────┤
│ 1  │ 100    │ 1      │ 1         │
│ 2  │ 250    │ 1      │ 1         │
└────┴────────┴────────┴───────────┘
