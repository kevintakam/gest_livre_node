# Node.js - Mini application de gestion de livres

## Objectif

Créer une mini-application Node.js permettant de gérer des livres avec :

* inscription utilisateur
* connexion utilisateur
* ajout de livres
* affichage des livres
* modification des livres
* suppression des livres
* authentification JWT ou session
* base de données MySQL ou MongoDB
* architecture Controller / Service / Repository
* templates EJS ou API REST

---

# 1. Prérequis

Avant de commencer, vérifier que les outils suivants sont installés :

```bash
node -v
npm -v
git --version
```

Rôle :

* `node` : exécuter le serveur JavaScript côté backend
* `npm` : installer les dépendances du projet
* `git` : versionner le code source

---

# 2. Créer le projet

```bash
mkdir node-books
cd node-books
npm init -y
```

Rôle :

* crée le dossier du projet
* initialise un fichier `package.json`
* prépare le projet Node.js

---

# 3. Installer Express

```bash
npm install express
```

Rôle :

* installe Express
* permet de créer un serveur HTTP
* permet de gérer les routes de l'application

---

# 4. Installer les dépendances principales

```bash
npm install dotenv cors helmet morgan
```

Rôle :

* `dotenv` : charger les variables d’environnement depuis `.env`
* `cors` : autoriser les appels HTTP entre frontend et backend
* `helmet` : ajouter des protections HTTP de base
* `morgan` : logger les requêtes HTTP

---

# 5. Installer les dépendances de développement

```bash
npm install -D nodemon
```

Rôle :

* redémarre automatiquement le serveur après modification du code

---

# 6. Ajouter les scripts npm

Dans `package.json` :

```json
"scripts": {
  "dev": "nodemon src/server.js",
  "start": "node src/server.js"
}
```

Rôle :

* `npm run dev` : lancer le serveur en développement
* `npm start` : lancer le serveur en mode classique

---

# 7. Créer la structure du projet

```bash
mkdir src
mkdir src/config
mkdir src/controllers
mkdir src/routes
mkdir src/services
mkdir src/repositories
mkdir src/models
mkdir src/middlewares
mkdir src/errors
mkdir src/utils
```

Rôle :

* séparer les responsabilités
* éviter de mettre toute la logique dans les routes
* rendre le projet maintenable

Structure attendue :

```txt
node-books/
├── src/
│   ├── config/
│   ├── controllers/
│   ├── routes/
│   ├── services/
│   ├── repositories/
│   ├── models/
│   ├── middlewares/
│   ├── errors/
│   ├── utils/
│   ├── app.js
│   └── server.js
├── .env
├── package.json
└── README.md
```

---

# 8. Créer le fichier `.env`

```bash
touch .env
```

Sur Windows PowerShell :

```powershell
New-Item .env
```

Contenu :

```env
APP_PORT=3000
APP_ENV=development
JWT_SECRET=change_me_in_production
```

Rôle :

* stocker les variables de configuration
* éviter de mettre les secrets directement dans le code

---

# 9. Créer le serveur Express

Créer `src/app.js` :

```js
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');

const bookRoutes = require('./routes/book.routes');
const authRoutes = require('./routes/auth.routes');

const app = express();

app.use(helmet());
app.use(cors());
app.use(morgan('dev'));
app.use(express.json());

app.get('/', (req, res) => {
  res.json({ message: 'Node Books API is running' });
});

app.use('/api/auth', authRoutes);
app.use('/api/books', bookRoutes);

module.exports = app;
```

Rôle :

* configure Express
* active les middlewares globaux
* déclare les routes principales

---

# 10. Créer le point d’entrée serveur

Créer `src/server.js` :

```js
require('dotenv').config();

const app = require('./app');

const port = process.env.APP_PORT || 3000;

app.listen(port, () => {
  console.log(`Server running on http://127.0.0.1:${port}`);
});
```

Rôle :

* charge les variables d’environnement
* démarre le serveur HTTP

---

# 11. Créer les routes Book

Créer `src/routes/book.routes.js` :

```js
const express = require('express');
const bookController = require('../controllers/book.controller');

const router = express.Router();

router.get('/', bookController.findAll);
router.get('/:id', bookController.findById);
router.post('/', bookController.create);
router.put('/:id', bookController.update);
router.delete('/:id', bookController.remove);

module.exports = router;
```

Rôle :

* déclare les endpoints liés aux livres
* délègue la logique au contrôleur

Routes disponibles :

```txt
GET    /api/books
GET    /api/books/:id
POST   /api/books
PUT    /api/books/:id
DELETE /api/books/:id
```

---

# 12. Créer le contrôleur Book

Créer `src/controllers/book.controller.js` :

```js
const bookService = require('../services/book.service');

async function findAll(req, res, next) {
  try {
    const books = await bookService.findAll();
    res.json(books);
  } catch (error) {
    next(error);
  }
}

async function findById(req, res, next) {
  try {
    const book = await bookService.findById(req.params.id);
    res.json(book);
  } catch (error) {
    next(error);
  }
}

async function create(req, res, next) {
  try {
    const book = await bookService.create(req.body);
    res.status(201).json(book);
  } catch (error) {
    next(error);
  }
}

async function update(req, res, next) {
  try {
    const book = await bookService.update(req.params.id, req.body);
    res.json(book);
  } catch (error) {
    next(error);
  }
}

async function remove(req, res, next) {
  try {
    await bookService.remove(req.params.id);
    res.status(204).send();
  } catch (error) {
    next(error);
  }
}

module.exports = {
  findAll,
  findById,
  create,
  update,
  remove,
};
```

Rôle :

* reçoit les requêtes HTTP
* récupère les paramètres
* appelle le service métier
* retourne une réponse HTTP

---

# 13. Créer le service Book

Créer `src/services/book.service.js` :

```js
const bookRepository = require('../repositories/book.repository');

async function findAll() {
  return bookRepository.findAll();
}

async function findById(id) {
  const book = await bookRepository.findById(id);

  if (!book) {
    const error = new Error('Book not found');
    error.statusCode = 404;
    throw error;
  }

  return book;
}

async function create(payload) {
  if (!payload.title || !payload.author) {
    const error = new Error('Title and author are required');
    error.statusCode = 400;
    throw error;
  }

  return bookRepository.create({
    title: payload.title,
    author: payload.author,
    isbn: payload.isbn || null,
    description: payload.description || null,
    available: payload.available ?? true,
  });
}

async function update(id, payload) {
  await findById(id);
  return bookRepository.update(id, payload);
}

async function remove(id) {
  await findById(id);
  return bookRepository.remove(id);
}

module.exports = {
  findAll,
  findById,
  create,
  update,
  remove,
};
```

Rôle :

* contient la logique métier
* valide les données
* évite de mettre la logique dans le contrôleur

---

# 14. Créer un repository Book en mémoire

Créer `src/repositories/book.repository.js` :

```js
let books = [];
let currentId = 1;

async function findAll() {
  return books;
}

async function findById(id) {
  return books.find((book) => book.id === Number(id)) || null;
}

async function create(data) {
  const book = {
    id: currentId++,
    title: data.title,
    author: data.author,
    isbn: data.isbn,
    description: data.description,
    available: data.available,
    createdAt: new Date().toISOString(),
  };

  books.push(book);
  return book;
}

async function update(id, data) {
  const book = await findById(id);

  Object.assign(book, {
    title: data.title ?? book.title,
    author: data.author ?? book.author,
    isbn: data.isbn ?? book.isbn,
    description: data.description ?? book.description,
    available: data.available ?? book.available,
  });

  return book;
}

async function remove(id) {
  books = books.filter((book) => book.id !== Number(id));
}

module.exports = {
  findAll,
  findById,
  create,
  update,
  remove,
};
```

Rôle :

* simule une base de données
* permet de commencer sans MySQL ni MongoDB
* sera remplaçable par un vrai repository SQL ou MongoDB

---

# 15. Ajouter la gestion d’erreurs globale

Dans `src/app.js`, avant `module.exports = app;`, ajouter :

```js
app.use((req, res) => {
  res.status(404).json({ message: 'Route not found' });
});

app.use((error, req, res, next) => {
  const statusCode = error.statusCode || 500;

  res.status(statusCode).json({
    message: error.message || 'Internal server error',
  });
});
```

Rôle :

* centralise la gestion des erreurs
* évite les `try/catch` mal gérés partout
* retourne des réponses JSON propres

---

# 16. Lancer l’application

```bash
npm run dev
```

Accès :

```txt
http://127.0.0.1:3000
```

Tester :

```txt
GET http://127.0.0.1:3000/api/books
```

---

# 17. Tester avec Postman ou Thunder Client

## Créer un livre

```http
POST http://127.0.0.1:3000/api/books
Content-Type: application/json
```

Body :

```json
{
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "9780132350884",
  "description": "Livre sur les bonnes pratiques de développement",
  "available": true
}
```

## Lister les livres

```http
GET http://127.0.0.1:3000/api/books
```

## Voir un livre

```http
GET http://127.0.0.1:3000/api/books/1
```

## Modifier un livre

```http
PUT http://127.0.0.1:3000/api/books/1
Content-Type: application/json
```

Body :

```json
{
  "available": false
}
```

## Supprimer un livre

```http
DELETE http://127.0.0.1:3000/api/books/1
```

---

# 18. Installer l’authentification JWT

```bash
npm install bcrypt jsonwebtoken
```

Rôle :

* `bcrypt` : hasher les mots de passe
* `jsonwebtoken` : générer et vérifier les tokens JWT

---

# 19. Créer les routes Auth

Créer `src/routes/auth.routes.js` :

```js
const express = require('express');
const authController = require('../controllers/auth.controller');

const router = express.Router();

router.post('/register', authController.register);
router.post('/login', authController.login);

module.exports = router;
```

Routes disponibles :

```txt
POST /api/auth/register
POST /api/auth/login
```

---

# 20. Créer le repository User en mémoire

Créer `src/repositories/user.repository.js` :

```js
let users = [];
let currentId = 1;

async function findByEmail(email) {
  return users.find((user) => user.email === email) || null;
}

async function create(data) {
  const user = {
    id: currentId++,
    email: data.email,
    password: data.password,
    roles: data.roles || ['ROLE_USER'],
    createdAt: new Date().toISOString(),
  };

  users.push(user);
  return user;
}

module.exports = {
  findByEmail,
  create,
};
```

Rôle :

* stocke temporairement les utilisateurs
* sera remplaçable par une vraie base de données

---

# 21. Créer le service Auth

Créer `src/services/auth.service.js` :

```js
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const userRepository = require('../repositories/user.repository');

async function register(payload) {
  if (!payload.email || !payload.password) {
    const error = new Error('Email and password are required');
    error.statusCode = 400;
    throw error;
  }

  const existingUser = await userRepository.findByEmail(payload.email);

  if (existingUser) {
    const error = new Error('Email already used');
    error.statusCode = 409;
    throw error;
  }

  const hashedPassword = await bcrypt.hash(payload.password, 12);

  const user = await userRepository.create({
    email: payload.email,
    password: hashedPassword,
  });

  return {
    id: user.id,
    email: user.email,
    roles: user.roles,
  };
}

async function login(payload) {
  if (!payload.email || !payload.password) {
    const error = new Error('Email and password are required');
    error.statusCode = 400;
    throw error;
  }

  const user = await userRepository.findByEmail(payload.email);

  if (!user) {
    const error = new Error('Invalid credentials');
    error.statusCode = 401;
    throw error;
  }

  const isPasswordValid = await bcrypt.compare(payload.password, user.password);

  if (!isPasswordValid) {
    const error = new Error('Invalid credentials');
    error.statusCode = 401;
    throw error;
  }

  const token = jwt.sign(
    {
      sub: user.id,
      email: user.email,
      roles: user.roles,
    },
    process.env.JWT_SECRET,
    { expiresIn: '1h' }
  );

  return { token };
}

module.exports = {
  register,
  login,
};
```

Rôle :

* gère l’inscription
* hash le mot de passe
* vérifie les identifiants
* génère le JWT

---

# 22. Créer le contrôleur Auth

Créer `src/controllers/auth.controller.js` :

```js
const authService = require('../services/auth.service');

async function register(req, res, next) {
  try {
    const user = await authService.register(req.body);
    res.status(201).json(user);
  } catch (error) {
    next(error);
  }
}

async function login(req, res, next) {
  try {
    const result = await authService.login(req.body);
    res.json(result);
  } catch (error) {
    next(error);
  }
}

module.exports = {
  register,
  login,
};
```

Rôle :

* reçoit les requêtes `/register` et `/login`
* délègue la logique au service Auth

---

# 23. Créer le middleware JWT

Créer `src/middlewares/auth.middleware.js` :

```js
const jwt = require('jsonwebtoken');

function requireAuth(req, res, next) {
  const authorization = req.headers.authorization;

  if (!authorization || !authorization.startsWith('Bearer ')) {
    return res.status(401).json({ message: 'Missing token' });
  }

  const token = authorization.split(' ')[1];

  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch (error) {
    return res.status(401).json({ message: 'Invalid token' });
  }
}

module.exports = {
  requireAuth,
};
```

Rôle :

* vérifie le token JWT
* protège les routes sensibles
* ajoute l’utilisateur décodé dans `req.user`

---

# 24. Protéger les routes Book

Modifier `src/routes/book.routes.js` :

```js
const express = require('express');
const bookController = require('../controllers/book.controller');
const { requireAuth } = require('../middlewares/auth.middleware');

const router = express.Router();

router.get('/', bookController.findAll);
router.get('/:id', bookController.findById);
router.post('/', requireAuth, bookController.create);
router.put('/:id', requireAuth, bookController.update);
router.delete('/:id', requireAuth, bookController.remove);

module.exports = router;
```

Rôle :

* lecture publique des livres
* création, modification et suppression protégées par JWT

---

# 25. Tester l’authentification

## Inscription

```http
POST http://127.0.0.1:3000/api/auth/register
Content-Type: application/json
```

Body :

```json
{
  "email": "admin@test.com",
  "password": "password123"
}
```

## Connexion

```http
POST http://127.0.0.1:3000/api/auth/login
Content-Type: application/json
```

Body :

```json
{
  "email": "admin@test.com",
  "password": "password123"
}
```

Réponse attendue :

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

## Créer un livre avec token

```http
POST http://127.0.0.1:3000/api/books
Authorization: Bearer VOTRE_TOKEN
Content-Type: application/json
```

Body :

```json
{
  "title": "Domain-Driven Design",
  "author": "Eric Evans"
}
```

---

# 26. Installer MySQL avec Prisma

```bash
npm install prisma @prisma/client
npx prisma init
```

Rôle :

* `prisma` : ORM moderne pour Node.js
* `@prisma/client` : client généré pour interroger la base
* `prisma init` : crée le dossier `prisma/` et le fichier `.env`

---

# 27. Configurer la base de données Prisma

Dans `.env` :

```env
DATABASE_URL="mysql://root:@localhost:3306/node_books"
```

Créer la base dans MySQL :

```sql
CREATE DATABASE node_books;
```

---

# 28. Créer le schéma Prisma

Dans `prisma/schema.prisma` :

```prisma
generator client {
  provider = "prisma-client-js"
}

 datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  password  String
  roles     Json
  createdAt DateTime @default(now())
}

model Book {
  id          Int      @id @default(autoincrement())
  title       String
  author      String
  isbn        String?
  description String?
  available   Boolean  @default(true)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

Rôle :

* définit les tables `User` et `Book`
* prépare les migrations SQL

---

# 29. Créer et exécuter la migration Prisma

```bash
npx prisma migrate dev --name init
```

Rôle :

* crée les tables dans MySQL
* génère le client Prisma

---

# 30. Créer le client Prisma

Créer `src/config/prisma.js` :

```js
const { PrismaClient } = require('@prisma/client');

const prisma = new PrismaClient();

module.exports = prisma;
```

Rôle :

* centralise l’accès à la base de données
* évite de créer plusieurs instances Prisma

---

# 31. Remplacer le repository Book par Prisma

Modifier `src/repositories/book.repository.js` :

```js
const prisma = require('../config/prisma');

async function findAll() {
  return prisma.book.findMany({
    orderBy: { createdAt: 'desc' },
  });
}

async function findById(id) {
  return prisma.book.findUnique({
    where: { id: Number(id) },
  });
}

async function create(data) {
  return prisma.book.create({
    data,
  });
}

async function update(id, data) {
  return prisma.book.update({
    where: { id: Number(id) },
    data,
  });
}

async function remove(id) {
  return prisma.book.delete({
    where: { id: Number(id) },
  });
}

module.exports = {
  findAll,
  findById,
  create,
  update,
  remove,
};
```

Rôle :

* remplace le stockage mémoire par MySQL
* garde la même interface de repository
* ne modifie presque pas le service

---

# 32. Remplacer le repository User par Prisma

Modifier `src/repositories/user.repository.js` :

```js
const prisma = require('../config/prisma');

async function findByEmail(email) {
  return prisma.user.findUnique({
    where: { email },
  });
}

async function create(data) {
  return prisma.user.create({
    data: {
      email: data.email,
      password: data.password,
      roles: data.roles || ['ROLE_USER'],
    },
  });
}

module.exports = {
  findByEmail,
  create,
};
```

Rôle :

* connecte l’inscription et la connexion à MySQL
* garde le service Auth inchangé

---

# 33. Commandes Prisma utiles

```bash
npx prisma studio
npx prisma migrate dev
npx prisma generate
npx prisma migrate reset
```

Rôle :

* `prisma studio` : ouvrir une interface graphique pour voir la base
* `migrate dev` : créer/appliquer les migrations
* `generate` : régénérer le client Prisma
* `migrate reset` : réinitialiser la base en développement

---

# 34. Installer les tests

```bash
npm install -D jest supertest
```

Rôle :

* `jest` : framework de test
* `supertest` : tester les routes HTTP Express

---

# 35. Ajouter le script de test

Dans `package.json` :

```json
"scripts": {
  "dev": "nodemon src/server.js",
  "start": "node src/server.js",
  "test": "jest"
}
```

---

# 36. Exemple de test HTTP

Créer `src/book.test.js` :

```js
const request = require('supertest');
const app = require('./app');

describe('GET /api/books', () => {
  it('should return 200', async () => {
    const response = await request(app).get('/api/books');

    expect(response.status).toBe(200);
    expect(Array.isArray(response.body)).toBe(true);
  });
});
```

Lancer :

```bash
npm test
```

---

# 37. Commandes utiles Node.js

```bash
npm install
npm run dev
npm start
npm test
npm outdated
npm audit
npm audit fix
```

Rôle :

* `npm install` : installer les dépendances
* `npm run dev` : lancer en développement
* `npm start` : lancer le serveur
* `npm test` : exécuter les tests
* `npm outdated` : voir les dépendances obsolètes
* `npm audit` : détecter les vulnérabilités
* `npm audit fix` : corriger automatiquement certaines vulnérabilités

---

# 38. Workflow quotidien

```bash
npm install
npx prisma migrate dev
npm run dev
```

Puis ouvrir :

```txt
http://127.0.0.1:3000
```

---

# 39. Résumé des grandes étapes

```txt
1. Créer le projet Node.js
2. Installer Express
3. Créer la structure Controller / Service / Repository
4. Créer les routes Book
5. Créer les contrôleurs
6. Créer les services
7. Créer les repositories
8. Ajouter la gestion d’erreurs
9. Tester les routes Book
10. Ajouter l’authentification JWT
11. Protéger les routes sensibles
12. Ajouter Prisma
13. Connecter MySQL
14. Ajouter les tests
15. Lancer l’application
```

---

# 40. Bonnes pratiques

* Ne pas mettre la logique métier dans les routes
* Les routes doivent uniquement déclarer les endpoints
* Les contrôleurs doivent gérer HTTP
* Les services doivent gérer la logique métier
* Les repositories doivent gérer la persistance
* Les erreurs doivent être centralisées
* Les mots de passe doivent être hashés avec `bcrypt`
* Les secrets doivent être dans `.env`
* Le token JWT doit avoir une durée de vie limitée
* La base de données doit être manipulée via un repository

---

# 41. Architecture cible propre

```txt
HTTP Request
    ↓
Route
    ↓
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

Responsabilités :

* `Route` : point d’entrée HTTP
* `Controller` : adaptation HTTP
* `Service` : règles métier
* `Repository` : accès aux données
* `Database` : persistance

---

# 42. Auteur

Projet Node.js - apprentissage backend avec Express, JWT, Prisma et MySQL.
