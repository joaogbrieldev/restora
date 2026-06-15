# Dependências principais

## Raiz

- **pnpm** — gerenciador de pacotes e workspaces
- **turbo** — orquestração de tasks no monorepo
- **nx** — tooling auxiliar (`@nx/js`, `@swc-node/register`)

## `apps/api`

| Categoria   | Pacotes principais                                      |
|-------------|---------------------------------------------------------|
| Framework   | `@nestjs/common`, `@nestjs/core`, `@nestjs/platform-express` |
| Runtime     | `rxjs`, `reflect-metadata`                              |
| Tooling     | `@nestjs/cli`, `typescript`, `jest`, `supertest`        |
| Workspace   | `@restora/config`                                       |

## `apps/web-admin`

| Categoria   | Pacotes principais                                      |
|-------------|---------------------------------------------------------|
| Framework   | `next` 16, `react` 19, `react-dom` 19                   |
| Estilo      | `tailwindcss` 4, `@tailwindcss/postcss`                 |
| Testes      | `jest`, `@testing-library/react`, `@playwright/test`    |
| Tooling     | `@biomejs/biome`, `typescript`                          |
| Workspace   | `@restora/config`                                       |

## `packages/config` (`@restora/config`)

| Categoria   | Pacotes principais                                      |
|-------------|---------------------------------------------------------|
| Build       | `tsup`                                                  |
| Exports     | configs compartilhadas de Biome, TypeScript e Jest      |

## Planejado (Fase 1 — ver `_techspec.md`)

- **Prisma** + **PostgreSQL** — persistência e migrations
- **JWT** — autenticação com contexto de tenant (`restaurant_id`, role)
