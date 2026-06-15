# Arquitetura

## Estrutura do monorepo

```
apps/
  api/           # Backend NestJS — regras de negócio, auth, relatórios
  web-admin/     # Frontend Next.js (App Router) — painel interno
packages/
  config/        # @restora/config — TypeScript, Biome e Jest compartilhados
```

## Portas

| Serviço              | Porta padrão | Configuração                          |
|----------------------|--------------|---------------------------------------|
| `web-admin` (Next.js)| `3000`       | Padrão do `next dev`                  |
| `api` (NestJS)       | `3001`       | `process.env.PORT` (fallback `3001`)  |
| PostgreSQL           | `5432`       | Planejado na Fase 1 (`DATABASE_URL`)  |

URLs locais típicas:

- Admin: http://localhost:3000
- API: http://localhost:3001

## Documentos de arquitetura

- [Arquitetura do Backend](backend-architecture.md)
- [Estrutura dos Módulos](module-structure.md)
- [Multi-Tenancy](multi-tenancy.md)
- [Estratégia de Transações](transaction-strategy.md)
- [Arquitetura dos Relatórios](reporting-architecture.md)
- [Domínio de Caixa](cash-session-domain.md)
