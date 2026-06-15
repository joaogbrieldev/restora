# Comandos

## Desenvolvimento

Rodar **todos** os apps em modo dev (via Turborepo):

```bash
pnpm exec turbo run start:dev
```

Rodar um app específico:

```bash
# API NestJS (porta 3001)
pnpm --filter api start:dev

# Painel admin Next.js (porta 3000)
pnpm --filter web-admin start:dev
```

## Build

```bash
# Build de todo o monorepo
pnpm exec turbo run build

# Build de um pacote/app
pnpm --filter @restora/config build
pnpm --filter api build
pnpm --filter web-admin build
```

## Qualidade de código

Biome é usado nos apps. Na raiz, via Turborepo:

```bash
pnpm exec turbo run lint          # lint
pnpm exec turbo run format        # format (check)
pnpm exec turbo run check         # lint + format
pnpm exec turbo run lint:fix      # lint com auto-fix
pnpm exec turbo run format:fix    # format com write
pnpm exec turbo run check:fix     # check com auto-fix
```

Por app:

```bash
pnpm --filter api check
pnpm --filter web-admin check
```

## Testes

```bash
# Todos os workspaces
pnpm exec turbo run test

# Por app
pnpm --filter api test
pnpm --filter api test:e2e
pnpm --filter api test:cov

pnpm --filter web-admin test
pnpm --filter web-admin test:e2e   # Playwright
pnpm --filter web-admin test:cov
```

Modo watch:

```bash
pnpm exec turbo run test:watch
```

## Produção (web-admin)

```bash
pnpm --filter web-admin build
pnpm --filter web-admin start
```
