# @igorchugurov/public-api-sdk

Переиспользуемый SDK для работы с универсальными сущностями (Entity Instances) в Axon Dashboard и других клиентских приложениях.

## 📦 Установка

```bash
# В монорепо (локально)
pnpm add @igorchugurov/public-api-sdk@workspace:*

# Или после публикации в npm
pnpm add @igorchugurov/public-api-sdk
```

## 🚀 Быстрый старт

### 1. Server Component (SSR)

```typescript
import { createServerSDK } from '@igorchugurov/public-api-sdk/server';
import { cookies } from 'next/headers';

export default async function MyPage({ params }) {
  const { projectId } = await params;
  const cookieStore = await cookies();

  const sdk = await createServerSDK(
    projectId,
    {
      supabaseUrl: process.env.NEXT_PUBLIC_SUPABASE_URL!,
      supabaseAnonKey: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
      cookies: {
        getAll: () => cookieStore.getAll(),
        setAll: (cookiesToSet) => {
          cookiesToSet.forEach(({ name, value, options }) =>
            cookieStore.set(name, value, options)
          );
        },
      },
    },
    {
      enableCache: true, // Кэшировать конфигурацию
    }
  );

  // Получаем список экземпляров
  const { data, pagination } = await sdk.getInstances(entityDefinitionId, {
    page: 1,
    limit: 20,
    search: 'test',
  });

  return <div>{/* ... */}</div>;
}
```

### 2. Client Component

```typescript
'use client';

import { createClientSDK } from '@igorchugurov/public-api-sdk';

const sdk = createClientSDK(
  projectId,
  {
    supabaseUrl: process.env.NEXT_PUBLIC_SUPABASE_URL!,
    supabaseAnonKey: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
  },
  {
    enableCache: true,
  }
);

// Используем SDK
const { data, pagination } = await sdk.getInstances(entityDefinitionId, {
  page: 1,
  limit: 20,
});
```

## 📚 API Reference

### CRUD операции

#### `getInstances(entityDefinitionId, params?)`

Получить список экземпляров с фильтрацией, поиском и пагинацией.

```typescript
const { data, pagination } = await sdk.getInstances(entityDefinitionId, {
  page?: number; // default: 1
  limit?: number; // default: 20
  search?: string;
  filters?: Record<string, string[]>;
  relationFilterModes?: Record<string, 'any' | 'all'>;
  sortBy?: string; // default: 'created_at'
  sortOrder?: 'asc' | 'desc'; // default: 'desc'
  relationsAsIds?: boolean; // default: false
});
```

#### `getInstance(entityDefinitionId, id, params?)`

Получить один экземпляр.

```typescript
const instance = await sdk.getInstance(entityDefinitionId, id, {
  relationsAsIds?: boolean; // default: false
});
```

#### `createInstance(entityDefinitionId, data)`

Создать новый экземпляр.

```typescript
const instance = await sdk.createInstance(entityDefinitionId, {
  data: Record<string, unknown>;
  relations?: Record<string, string[]>;
});
```

#### `updateInstance(entityDefinitionId, id, data)`

Обновить экземпляр.

```typescript
const instance = await sdk.updateInstance(entityDefinitionId, id, {
  data: Record<string, unknown>;
  relations?: Record<string, string[]>;
});
```

#### `deleteInstance(entityDefinitionId, id)`

Удалить экземпляр.

```typescript
await sdk.deleteInstance(entityDefinitionId, id);
```

### Конфигурация

#### `getEntityDefinitionConfig(entityDefinitionId)`

Получить конфигурацию EntityDefinition с полями.

```typescript
const config = await sdk.getEntityDefinitionConfig(entityDefinitionId);
```

#### `getEntityDefinitionWithUIConfig(entityDefinitionId)`

Получить EntityDefinition с полями и сгенерированной UI конфигурацией.

```typescript
const config = await sdk.getEntityDefinitionWithUIConfig(entityDefinitionId);
// config: { entityDefinition, fields, uiConfig }
```

## 🔧 Типы

```typescript
import type {
  EntityDefinition,
  Field,
  EntityInstanceWithFields,
  QueryParams,
  CreateInstanceData,
  UpdateInstanceData,
  PaginationResult,
} from '@igorchugurov/public-api-sdk';
```

## 🛠️ Обработка ошибок

SDK использует типизированные ошибки:

```typescript
import {
  NotFoundError,
  PermissionDeniedError,
  ValidationError,
  AuthenticationError,
  SDKError,
} from '@igorchugurov/public-api-sdk';

try {
  const instance = await sdk.getInstance(entityDefinitionId, id);
} catch (error) {
  if (error instanceof NotFoundError) {
    // 404 - экземпляр не найден
  } else if (error instanceof PermissionDeniedError) {
    // 403 - нет прав доступа
  } else if (error instanceof ValidationError) {
    // 400 - ошибка валидации
  } else if (error instanceof AuthenticationError) {
    // 401 - ошибка авторизации
  }
}
```

## ⚙️ Кэширование

SDK поддерживает кэширование конфигурации для оптимизации:

```typescript
const sdk = await createServerSDK(projectId, config, {
  enableCache: true, // default: true
  cacheTTL: 5 * 60 * 1000, // default: 5 минут
});
```

## 🚧 Разработка

```bash
# Установка зависимостей
pnpm install

# Сборка
pnpm build

# Проверка типов
pnpm type-check

# Режим разработки (watch mode)
pnpm dev
```

## 📄 Лицензия

MIT

