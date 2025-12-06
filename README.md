# @igorchugurov/public-api-sdk

Переиспользуемый SDK для работы с универсальными сущностями (Entity Instances) в Axon Dashboard и других клиентских приложениях.

## 📦 Установка

```bash
# В монорепо (локально)
pnpm add @igorchugurov/public-api-sdk@workspace:*

# Или после публикации в npm
pnpm add @igorchugurov/public-api-sdk
```

## ✨ Возможности

SDK предоставляет полный набор инструментов для работы с универсальными сущностями:

### 🔄 CRUD операции

- **Получение списка** с расширенной фильтрацией, поиском и пагинацией
- **Получение одного экземпляра** с автоматической загрузкой связей и файлов
- **Создание** экземпляров с поддержкой relations и автоматической установкой `created_by`
- **Обновление** экземпляров с поддержкой частичных обновлений и relations
- **Удаление** экземпляров с автоматической очисткой связей

### 🔗 Работа со связями (Relations)

- Автоматическое определение relation-полей из конфигурации
- Поддержка всех типов связей: `manyToMany`, `manyToOne`, `oneToMany`, `oneToOne`
- Batch-загрузка связанных объектов для оптимизации производительности
- Фильтрация по relations с режимами `any` (OR) и `all` (AND)
- Опция получения relations как ID или полных объектов

### 🔍 Поиск и фильтрация

- **Умный поиск** по полям с флагом `searchable: true`
- **JSONB фильтрация** для обычных полей
- **Relation фильтрация** с автоматическим определением relation-полей
- **Гибкая сортировка** по любому полю с поддержкой `asc`/`desc`

### 📁 Работа с файлами

- Автоматическая загрузка файлов и изображений для полей типа `files` и `images`
- Batch-загрузка файлов для оптимизации запросов
- Поддержка множественных файлов на поле

### ⚡ Производительность

- **Кэширование конфигурации** EntityDefinition и Fields (TTL: 5 минут по умолчанию)
- **Batch-запросы** для relations и файлов
- **Оптимизированные RPC функции** для поиска и загрузки связей
- Поддержка ESM и CJS форматов

### 🎯 Типобезопасность

- Полная типизация всех методов и параметров
- Экспорт всех типов для использования в вашем коде
- Типизированные ошибки для удобной обработки

### 🔐 Безопасность

- Интеграция с Supabase RLS (Row Level Security)
- Автоматическая проверка прав доступа
- Типизированные ошибки для различных сценариев доступа

### 🎨 UI конфигурация

- Автоматическая генерация UI конфигурации из EntityDefinition и Fields
- Поддержка кастомных UI настроек
- Генерация конфигурации колонок таблицы из полей

## 🚀 Быстрый старт

### 1. Server Component (SSR)

```typescript
import { createServerSDK } from "@igorchugurov/public-api-sdk/server";
import { cookies } from "next/headers";

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
    search: "test",
  });

  return <div>{/* ... */}</div>;
}
```

### 2. Client Component

```typescript
"use client";

import { createClientSDK } from "@igorchugurov/public-api-sdk";

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

Получить один экземпляр по ID.

```typescript
const instance = await sdk.getInstance(entityDefinitionId, id, {
  relationsAsIds?: boolean; // default: false
  loadFiles?: boolean; // default: false
});
```

**Пример:**

```typescript
// Базовое использование - без файлов
const instance = await sdk.getInstance("entity-def-id", "instance-id");

// Для редактирования - нужны только ID связей
const instance = await sdk.getInstance("entity-def-id", "instance-id", {
  relationsAsIds: true,
});

// Для отображения с файлами - полные объекты
const instance = await sdk.getInstance("entity-def-id", "instance-id", {
  relationsAsIds: false,
  loadFiles: true, // файлы и изображения будут загружены как полные объекты EntityFile
});
```

#### `getInstanceBySlug(entityDefinitionId, slug, params?)`

Получить один экземпляр по slug.

```typescript
const instance = await sdk.getInstanceBySlug(entityDefinitionId, slug, {
  relationsAsIds?: boolean; // default: false
  loadFiles?: boolean; // default: false
});
```

**Пример:**

```typescript
// Базовое использование - без файлов
const instance = await sdk.getInstanceBySlug(
  "entity-def-id",
  "my-article-slug"
);

// Для отображения с файлами - полные объекты
const instance = await sdk.getInstanceBySlug(
  "entity-def-id",
  "my-article-slug",
  {
    relationsAsIds: false,
    loadFiles: true, // файлы и изображения будут загружены как полные объекты EntityFile
  }
);
```

**Особенности:**

- Валидирует формат slug перед запросом (только строчные латинские буквы, цифры и дефисы)
- Работает аналогично `getInstance`, но ищет по slug вместо id
- Поддерживает те же параметры, что и `getInstance`
- Параметр `relationsAsIds`:
  - `false` (по умолчанию) - возвращает полные объекты связанных сущностей
  - `true` - возвращает только массивы ID связанных сущностей
- Параметр `loadFiles`:
  - `false` (по умолчанию) - файлы и изображения не загружаются
  - `true` - файлы и изображения загружаются как полные объекты `EntityFile` (с `fileUrl`, `fileName`, `fileSize` и т.д.)

#### `createInstance(entityDefinitionId, data)`

Создать новый экземпляр. Автоматически генерирует уникальный `slug` из поля `name`.

```typescript
const instance = await sdk.createInstance(entityDefinitionId, {
  data: Record<string, unknown>; // Обязательно должно содержать поле 'name' (string)
  relations?: Record<string, string[]>;
});
```

**Особенности:**

- Автоматически генерирует уникальный `slug` из поля `name`
- Если slug уже существует, добавляет случайный суффикс
- Автоматически устанавливает `created_by` из текущего пользователя
- Поддерживает создание relations в одном запросе

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

#### `getAllEntityDefinitions()`

Получить все EntityDefinitions проекта с полями одним запросом (JOIN). Используется для загрузки всех сущностей в layout.

```typescript
const entityDefinitions = await sdk.getAllEntityDefinitions();
// Возвращает массив EntityDefinitionConfig с полями
// Каждый элемент содержит: { id, name, slug, fields, ... }
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
} from "@igorchugurov/public-api-sdk";
```

### Slug поддержка

Все экземпляры и определения сущностей теперь содержат поле `slug`:

```typescript
// EntityInstance содержит slug
const instance: EntityInstanceWithFields = {
  id: string;
  slug: string; // URL-friendly идентификатор
  entityDefinitionId: string;
  // ...
};

// EntityDefinition также содержит slug
const entityDef: EntityDefinition = {
  id: string;
  name: string;
  slug: string; // URL-friendly идентификатор
  // ...
};
```

Slug автоматически генерируется при создании экземпляра из поля `name` и гарантирует уникальность в рамках одной EntityDefinition.

## 🛠️ Обработка ошибок

SDK использует типизированные ошибки:

```typescript
import {
  NotFoundError,
  PermissionDeniedError,
  ValidationError,
  AuthenticationError,
  SDKError,
} from "@igorchugurov/public-api-sdk";

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
