# Restora — Guia para Agentes

Restora é um monorepo **Turborepo + pnpm** para um SaaS de gestão de marmitarias. A Fase 1 entrega API NestJS, painel admin Next.js, PostgreSQL + Prisma e configurações compartilhadas em `packages/config`.

## Quick Reference

- **Node.js:** `24.x`
- **pnpm:** `10.14.0`
- **Instalação:** executar `pnpm install` sempre na raiz.
- **Dev:** `pnpm exec turbo run start:dev`
- **Check:** `pnpm exec turbo run check`
- **Test:** `pnpm exec turbo run test`
- **Build:** `pnpm exec turbo run build`

Use `pnpm --filter <workspace> <task>` para comandos específicos, por exemplo `pnpm --filter api start:dev` ou `pnpm --filter web-admin start:dev`.

## Antes de Implementar

1. Consulte `.compozy/tasks/restora-saas-marmitarias/_techspec.md` para escopo técnico e decisões da Fase 1.
2. Consulte os ADRs em `.compozy/tasks/restora-saas-marmitarias/adrs/` antes de mudar arquitetura, permissões, dinheiro, relatórios ou multi-tenancy.
3. Trabalhe na raiz do repositório; não rode instalação dentro de apps ou pacotes isolados.
4. Rode `pnpm --filter @restora/config build` antes de consumir alterações em `@restora/config`.

## Documentação Principal

- [Comandos](docs/commands.md) — dev, build, lint, testes e produção.
- [Arquitetura do Monorepo](docs/architecture.md) — apps, packages e portas.
- [Arquitetura do Backend](docs/backend-architecture.md) — camadas, dependências permitidas e limites.
- [Estrutura dos Módulos](docs/module-structure.md) — padrão de pastas por feature.
- [Multi-Tenancy](docs/multi-tenancy.md) — tenant scope, RLS e respostas cross-tenant.
- [Transações](docs/transaction-strategy.md) — operações atômicas e concorrência.
- [Relatórios](docs/reporting-architecture.md) — snapshots e histórico.
- [Domínio de Caixa](docs/cash-session-domain.md) — estados e invariantes.
- [Dependências](docs/dependencies.md) — stack por workspace.
- [Ambiente](docs/environment.md) — variáveis planejadas da Fase 1.

## Regras Obrigatórias

- Controllers não possuem regra de negócio; Controllers chamam Use Cases.
- Use Cases orquestram fluxos, transações e coordenação entre serviços.
- Domain Services concentram regras de negócio que não pertencem a uma entidade.
- Prisma fica apenas em infrastructure; Domain não depende de Prisma, NestJS ou HTTP.
- Toda tabela de tenant possui `restaurant_id`; acesso cross-tenant retorna `404`.
- Valores monetários usam centavos inteiros.
- Caixas fechados são imutáveis; correções são ajustes owner-only.
- Relatórios históricos usam snapshots, nunca recálculo livre de tabelas operacionais.
- Operações críticas devem usar transação.
- Nova decisão arquitetural relevante deve gerar ou atualizar um ADR.

## Filosofia

Preferir Clean Architecture pragmática com NestJS Modules, Repository Pattern, Use Cases, Domain Services pontuais, snapshots para relatórios e segurança multi-tenant. Evitar CQRS completo, Event Sourcing, Mediators, Specification Pattern, factories complexas e DDD tático completo sem necessidade real.
