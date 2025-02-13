# Packages

Os pacotes, plugins, extensões ou *packages* são como fragmentos de sabedoria do **Johan Chat**, cada pacote contém funcionalidades específicas acessíveis através de endpoints com Express + PrismaORM *(quando necessário)*.

Para garantir a funcionalidade dentro do Johan Chat, o pacote deve seguir uma estrutura padrão de entrada, se você fugir da estrutura Johan Chat pode não reconhecer as funcionalidades de um respectivo *package*.

## Packages Structes

Embora você possa criar sua própria arquitetura para pacotes mais complexos, é importante seguir a estrutura de entrada especificada abaixo para garantir que o pacote sejam reconhecidos por Johan Chat.

### 1. **package.js**
Se trata de um  `package.js` comum.

Exemplo:

```json
{
    "name": "my-package",
    "version": "1.0.0",
    "description": "Descrição do pacote",
    "main": "index.js"
}
```

### 2. **{entry-file}.js**
Se em ```package.json``` houver a propriedade `main`, será necessário ter um arquivo de entrada correspondente na raiz do pacote. 

**Exemplo de `package.json`**:

```json
{
    "main": "my-entry-file.js"
}
```

**Exemplo de `my-entry-file.js`**:

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

Considere, se possível nomear como `index.js`.

#### Setup

É um hook que pode ser usado na Ativação do Pacote.

#### Remove

É um hook que pode ser usado na Desativação do Pacote.

OBs: Você não precisar se ver sempre obrigado a usar `setup` e `remove`, apenas use quando for realmente necessário.

### 3. **routes.js**

O arquivo `routes.js` define as rotas e os endpoints disponíveis. As rotas devem seguir o padrão abaixo e devem ser exportadas como um array de objetos.

Cada Objeto de Rota contém:

- `method`: O método HTTP (GET, POST, PUT, DELETE e PATCH)
- `description`: Uma breve descrição da Function.
- `router`: A rota, que será equivalente a {packageName}/{router}.
- `call`: Function que define o comportamento da Rota.

**Exemplo Simples**:

```js
const { getUser } = require('./services');

module.exports = [
    {
        method: 'get',
        description: 'Retorna as informações do model User',
        router: "profile",
        call: async (req, res) => {
            res.json(await getUser(req.user?.id));
        }
    }
];
```

**Exemplo²**:

Se o arquivo `routes.js` se tornar grande, recomenda-se modularizar as rotas...

```js
const { getUser } = require('./services');
const myEndpoint = require('../routes/myEndpoint');

module.exports = [
    ...myEndpoint,

    {
        method: 'get',
        description: 'Retorna as informações do usuário',
        router: "profile",
        call: async (req, res) => {
            res.json(await getUser(req.user?.id));
        }
    }
];
```

### 4. **services.js**

O arquivo `services.js` contém as principais funções que são utilizadas pelas rotas.
Aqui ficam implementadas as lógicas de cada funcionalidade, geralmente dentro de blocos `try/catch` para garantir maior robustez no tratamento de erros.

**Exemplo:**:

```js
// services.js
const prisma = require('prisma');

async function getUser(id) {
    try {
        return await prisma.user.findUnique({ where: { id } });
    } catch (error) {
        throw new Error("Erro ao buscar usuário");
    }
}

module.exports = { getUser };
```

### 5. **schema.prisma**

O arquivo `schema.prisma` deve definir os modelos usados pelo pacote, mas **não é necessário criar um schema completo**. Apenas adicione os fragmentos necessários para os modelos que o seu pacote utilizar.

A estrutura do modelo já inclui alguns padrões como `User`, `Agent`, `Chat`, e `ChatMessage` e você deve adicionar apenas os campos que realmente precisar.

#### Exemplos de Modelos Padrão:

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

### 6. **Adicionando Campos nos Modelos Existentes**

Para adicionar novos campos a um modelo existente, você não precisa repetir todas as propriedades do modelo. Basta adicionar apenas o novo campo desejado.

**Exemplo**:

Adicionando um campo `new_label` ao modelo `User`:

```prisma
model User {
  lastname   String
}
```

**Erro Comum**:

Não tente adicionar um campo com o mesmo nome e tipo diferente, pois isso causará inconsistências.

**Errado**:

```prisma
model User {
  password  Int
}
```

Considere então que `password` já foi definido anteriormente por padrão como `String`.
