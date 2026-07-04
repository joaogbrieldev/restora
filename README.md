# Restora

Restora é um SaaS multi-tenant para gestão de marmitarias e pequenos restaurantes. A Fase 1 foca em substituir o fechamento de caixa em papel por um fluxo digital simples para abertura/fechamento de caixa, despesas, sangrias, presença de funcionários e relatórios de lucro diário/mensal.

## Stack

- **Monorepo:** Turborepo + pnpm workspaces
- **Backend:** NestJS (`apps/api`)
- **Frontend admin:** Next.js App Router (`apps/web-admin`)
- **Banco planejado:** PostgreSQL + Prisma
- **Config compartilhada:** `@restora/config` em `packages/config`
- **Qualidade:** TypeScript, Biome, Jest e Playwright

## Pré-Requisitos

- Node.js `24.x`
- pnpm `10.14.0`

```bash
corepack enable
corepack prepare pnpm@10.14.0 --activate
pnpm install
```

Execute comandos sempre na raiz do repositório.

## Desenvolvimento

Rodar todos os apps:

```bash
pnpm exec turbo run start:dev
```

Rodar apps específicos:

```bash
pnpm --filter api start:dev
pnpm --filter web-admin start:dev
```

URLs locais padrão:

- Admin: http://localhost:3000
- API: http://localhost:3001

## Comandos Úteis

```bash
pnpm exec turbo run check
pnpm exec turbo run test
pnpm exec turbo run build
```

Build da configuração compartilhada:

```bash
pnpm --filter @restora/config build
```

Veja todos os comandos em [docs/commands.md](docs/commands.md).

## Estrutura

```text
apps/
  api/        # API NestJS
  web-admin/  # Painel administrativo Next.js
packages/
  config/     # Configurações compartilhadas
docs/         # Documentação técnica e arquitetural
.compozy/     # PRD, TechSpec e ADRs do workflow
```

## Escopo da Fase 1

Entra:

- Abertura e fechamento de caixa.
- Vendas manuais por método de pagamento.
- Despesas e sangrias.
- Funcionários e presença do dia.
- Custo de pessoal no fechamento.
- Resumo de lucro diário e acumulado mensal.
- Papéis de dono e caixa/funcionário.
- Configuração do restaurante e categorias de despesas.

Fica fora da Fase 1: app do cliente, catálogo, carrinho, Pix integrado, WhatsApp automático, marketplaces, fiscal, estoque, CRM, onboarding self-serve e cobrança SaaS.

## Documentação

- [Guia para agentes](AGENTS.md)
- [Arquitetura do monorepo](docs/architecture.md)
- [Arquitetura do backend](docs/backend-architecture.md)
- [Estrutura dos módulos](docs/module-structure.md)
- [Multi-tenancy](docs/multi-tenancy.md)
- [Transações](docs/transaction-strategy.md)
- [Relatórios](docs/reporting-architecture.md)
- [Domínio de caixa](docs/cash-session-domain.md)
- [TechSpec da Fase 1](.compozy/tasks/restora-saas-marmitarias/_techspec.md)
- [PRD da Fase 1](.compozy/tasks/restora-saas-marmitarias/_prd.md)

## Decisões Arquiteturais

As decisões ficam em `.compozy/tasks/restora-saas-marmitarias/adrs/`.

Pontos obrigatórios da Fase 1:

- Clean Architecture pragmática com módulos NestJS.
- Controllers chamam Use Cases e não contêm regra de negócio.
- Prisma fica na camada de infrastructure.
- Toda tabela de tenant usa `restaurant_id`.
- Acesso cross-tenant retorna `404`.
- Valores monetários usam centavos inteiros.
- Caixas fechados são imutáveis.
- Relatórios históricos usam snapshots e ajustes owner-only.

## Status

Projeto em fase inicial de implementação do MVP caixa-first.
