# Packages

Packages, plugins, extensions, or *packages* are like fragments of wisdom from **Johan Chat**. Each package contains specific functionalities accessible through endpoints with Express + PrismaORM (when necessary).

To ensure functionality within Johan Chat, the package must follow the standard entry structure. If you deviate from the Johan Chat structure, it may not recognize the functionalities of a particular *package*.

## Packages Structure

While you can create your own architecture for more complex packages, it's important to follow the specified entry structure below to ensure the package is recognized by Johan Chat.

### 1. **package.js**
This is a common `package.js` file.

Example:

```json
{
    "name": "my-package",
    "version": "1.0.0",
    "description": "Package description",
    "main": "index.js"
}
```

### 2. **{entry-file}.js**
If the `package.json` contains the `main` property, a corresponding entry file must be present at the root of the package.

**Example `package.json`:**

```json
{
    "main": "my-entry-file.js"
}
```

**Example `my-entry-file.js`:**

```js
module.exports = {
    setup: () => {
        // anything
    },
    remove: () => {
        // anything
    }
};
```

Consider naming it `index.js` if possible.

#### Setup

This is a hook that can be used when the package is **activated**.

#### Remove

This is a hook that can be used when the package is **deactivated**.

Note: You are not always required to use `setup` and `remove`, only use them when necessary.

### 3. **routes.js**

The `routes.js` file defines the available routes and endpoints. Routes must follow the structure below and should be exported as an array of objects.

Each Route Object contains:

- `method`: The HTTP method (GET, POST, PUT, DELETE, PATCH)
- `description`: A brief description of the function.
- `router`: The route, which will be equivalent to {packageName}/{router}.
- `call`: A function that defines the route's behavior.

**Simple Example:**

```js
const { getUser } = require('./services');

module.exports = [
    {
        method: 'get',
        description: 'Returns the User model information',
        router: "profile",
        call: async (req, res) => {
            res.json(await getUser(req.user?.id));
        }
    }
];
```

**Example²:**

If the `routes.js` file becomes large, it's recommended to modularize the routes...

```js
const { getUser } = require('./services');
const myEndpoint = require('../routes/myEndpoint');

module.exports = [
    ...myEndpoint,

    {
        method: 'get',
        description: 'Returns user information',
        router: "profile",
        call: async (req, res) => {
            res.json(await getUser(req.user?.id));
        }
    }
];
```

### 4. **services.js**

The `services.js` file contains the main functions that are used by the routes.
It implements the logic of each functionality, usually within `try/catch` blocks to ensure better error handling.

**Example:**

```js
// services.js
const prisma = require('prisma');

async function getUser(id) {
    try {
        return await prisma.user.findUnique({ where: { id } });
    } catch (error) {
        throw new Error("Error fetching user");
    }
}

module.exports = { getUser };
```

### 5. **schema.prisma**

The `schema.prisma` file should define the models used by the package, but **it is not necessary to create a full schema**. Only add the fragments necessary for the models your package uses.

The model structure already includes some common models like `User`, `Agent`, `Chat`, and `ChatMessage`, and you should only add the fields that are necessary for your functionality.

#### Example of Standard Models:

```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  password  String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Agent {
  id           Int      @id @default(autoincrement())
  iam          String
  model        String
  tools        Json 
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
}

model Chat {
  id        Int      @id @default(autoincrement())
  userId    Int      @unique
  messages  ChatMessage[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model ChatMessage {
  id        Int      @id @default(autoincrement())
  chat      Chat     @relation(fields: [chatId], references: [id])
  chatId    Int
  role      String
  content   String
  createdAt DateTime @default(now())
}
```

### 6. **Adding Fields to Existing Models**

To add new fields to an existing model, you don’t need to repeat all the properties of the model. Just add the new field you need.

**Example:**

Adding a `new_label` field to the `User` model:

```prisma
model User {
  lastname   String
}
```

**Common Error:**

Don't try to add a field with the same name and a different type, as this will cause inconsistencies.

**Incorrect Example:**

```prisma
model User {
  password  Int
}
```

In this case, `password` is already defined as a `String` by default.
