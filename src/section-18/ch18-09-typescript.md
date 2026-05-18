# TypeScript with Express

I resisted TypeScript for a long time. "JavaScript works fine," I thought. Then I spent an hour debugging a typo in a property name that TypeScript would have caught instantly. Now I do not start a project without it.

## Setup

Install TypeScript and types for Express and Node:

```bash
npm install -D typescript @types/express @types/node
npm install -D tsx
```

Initialize the TypeScript config:

```bash
npx tsc --init
```

Update `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "moduleResolution": "node",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

The `strict` flag enables all type checking. It is strict for a reason. Turn it on and fix the errors.

## Typing Request and Response

Express types for `req` and `res` are generic. I extend them to add my own types:

```ts
// src/types/express.d.ts
import { User } from '../models/User';

declare global {
  namespace Express {
    interface Request {
      user?: User;
    }
  }
}

export {};
```

Now `req.user` is typed everywhere in the app.

## Typing Route Handlers

I create a typed wrapper for async route handlers:

```ts
// src/types/index.ts
import { Request, Response, NextFunction } from 'express';

export interface ApiResponse<T = unknown> {
  success: boolean;
  data?: T;
  message?: string;
}

export type AsyncHandler = (
  req: Request,
  res: Response,
  next: NextFunction
) => Promise<void>;
```

```ts
// src/utils/asyncHandler.ts
import { NextFunction, Request, Response } from 'express';
import { ApiError } from './ApiError';

export const asyncHandler = (fn: AsyncHandler) => {
  return (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};
```

## Typing Controllers

Here is a typed controller for the product module:

```ts
// src/modules/product/product.controller.ts
import { Request, Response, NextFunction } from 'express';
import { Product, IProduct } from './product.model';
import { ApiError } from '../../utils/ApiError';
import { asyncHandler } from '../../utils/asyncHandler';

interface CreateProductBody {
  name: string;
  description: string;
  price: number;
  stock: number;
  category: string;
  brand?: string;
}

interface GetProductsQuery {
  page?: string;
  limit?: string;
  sort?: string;
  search?: string;
  category?: string;
  minPrice?: string;
  maxPrice?: string;
}

export const createProduct = asyncHandler(
  async (req: Request<{}, {}, CreateProductBody>, res: Response, next: NextFunction) => {
    const product: IProduct = await Product.create(req.body);
    res.status(201).json({ success: true, data: product });
  }
);

export const getProducts = asyncHandler(
  async (req: Request<{}, {}, {}, GetProductsQuery>, res: Response) => {
    const { page = '1', limit = '10', category, search } = req.query;

    const filter: Record<string, unknown> = {};
    if (category) filter.category = category;
    if (search) filter.$text = { $search: search };

    const products = await Product.find(filter)
      .skip((Number(page) - 1) * Number(limit))
      .limit(Number(limit));

    res.json({ success: true, data: products });
  }
);
```

The `Request<Params, ResBody, ReqBody, Query>` generic types tell TypeScript exactly what shape each part of the request has.

## Typing Middleware

```ts
// src/middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import { User, IUser } from '../modules/user/user.model';
import { ApiError } from '../utils/ApiError';

export const protect = asyncHandler(
  async (req: Request, res: Response, next: NextFunction) => {
    const authHeader = req.headers.authorization;
    if (!authHeader?.startsWith('Bearer')) {
      throw new ApiError(401, 'Not authenticated');
    }

    const token = authHeader.split(' ')[1];
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as { id: string };

    const user: IUser | null = await User.findById(decoded.id);
    if (!user) throw new ApiError(401, 'User no longer exists');

    req.user = user;
    next();
  }
);
```

The `!` after `process.env.JWT_SECRET` tells TypeScript "I promise this is not undefined." Use it sparingly and only when you are sure.

TypeScript catches mistakes at compile time instead of runtime. It takes extra effort to write types, but it saves hours of debugging.
