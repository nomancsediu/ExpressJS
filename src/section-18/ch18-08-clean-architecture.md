# Clean Architecture

I used to put everything in the controller. Database queries, business logic, email sending, all mixed together. It works at first, but it becomes a mess. Clean architecture separates concerns so each piece has one job.

## The Idea

Clean architecture organizes your code into layers. Each layer only depends on the layer inside it. The dependency rule: outer layers depend on inner layers, never the reverse.

```
Controllers (outer) -> Use Cases (inner) -> Entities (core)
```

Controllers handle HTTP. Use cases handle business logic. Entities define the domain. The database and external services are implementation details at the outermost layer.

## Entities (Core Domain)

Entities are plain objects that represent the business domain. No database, no Express, no external dependencies.

```js
// src/domain/entities/User.js
class User {
  constructor({ id, name, email, role }) {
    this.id = id;
    this.name = name;
    this.email = email;
    this.role = role;
  }

  isAdmin() {
    return this.role === 'admin';
  }

  canModifyProduct() {
    return this.role === 'admin' || this.role === 'manager';
  }
}

module.exports = User;
```

## Use Cases (Business Logic)

Use cases contain the business rules. They do not know about HTTP or databases. They receive input, apply rules, and return output.

```js
// src/domain/usecases/RegisterUser.js
class RegisterUser {
  constructor({ userRepository, hashingService }) {
    this.userRepository = userRepository;
    this.hashingService = hashingService;
  }

  async execute({ name, email, password }) {
    // Business rule: email must be unique
    const existing = await this.userRepository.findByEmail(email);
    if (existing) {
      throw new Error('Email already registered');
    }

    // Business rule: password must be hashed
    const hashedPassword = await this.hashingService.hash(password);

    // Business rule: default role is user
    const user = await this.userRepository.create({
      name,
      email,
      password: hashedPassword,
      role: 'user',
    });

    return user;
  }
}

module.exports = RegisterUser;
```

Notice how the use case does not import Mongoose or bcrypt directly. It receives dependencies through the constructor. This is dependency injection.

## Repositories (Data Access)

Repositories implement the interface that use cases expect. They handle the actual database operations.

```js
// src/infrastructure/repositories/MongoUserRepository.js
const User = require('../../models/user.model');
const UserEntity = require('../../domain/entities/User');

class MongoUserRepository {
  async findByEmail(email) {
    const doc = await User.findOne({ email });
    if (!doc) return null;
    return new UserEntity(doc.toObject());
  }

  async create(data) {
    const doc = await User.create(data);
    return new UserEntity(doc.toObject());
  }

  async findById(id) {
    const doc = await User.findById(id);
    if (!doc) return null;
    return new UserEntity(doc.toObject());
  }
}

module.exports = MongoUserRepository;
```

The repository converts database documents to domain entities. The use case never sees Mongoose documents.

## Controllers (HTTP Layer)

Controllers are thin. They parse the request, call the use case, and format the response.

```js
// src/interfaces/http/controllers/UserController.js
class UserController {
  constructor({ registerUserUseCase }) {
    this.registerUserUseCase = registerUserUseCase;
  }

  register = async (req, res, next) => {
    try {
      const user = await this.registerUserUseCase.execute(req.body);
      res.status(201).json({ success: true, data: user });
    } catch (err) {
      next(err);
    }
  };
}

module.exports = UserController;
```

## Wiring It Together

I use a simple factory to create and connect everything:

```js
// src/infrastructure/container.js
const MongoUserRepository = require('./repositories/MongoUserRepository');
const RegisterUser = require('../domain/usecases/RegisterUser');
const BcryptHashingService = require('./services/BcryptHashingService');
const UserController = require('../interfaces/http/controllers/UserController');

const userRepository = new MongoUserRepository();
const hashingService = new BcryptHashingService();
const registerUser = new RegisterUser({ userRepository, hashingService });
const userController = new UserController({ registerUserUseCase: registerUser });

module.exports = { userController };
```

Clean architecture takes more code upfront. But when you need to swap Mongoose for PostgreSQL, or bcrypt for argon2, you only change the infrastructure layer. The business logic stays untouched. That is the payoff.
