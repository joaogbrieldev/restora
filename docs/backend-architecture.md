# Arquitetura do Backend

O backend do Restora usa Clean Architecture pragmática implementada com módulos NestJS.

Objetivos:

- Isolar regras de negócio.
- Facilitar testes.
- Separar responsabilidades.
- Evitar acoplamento entre domínio, framework e banco.
- Evoluir o SaaS gradualmente sem introduzir arquitetura pesada cedo demais.

## Camadas

```text
Presentation
       ↓
Application
       ↓
Domain
       ↑
Infrastructure
```

Dependências apontam para dentro. A camada de domínio não conhece NestJS, Prisma, HTTP ou detalhes de banco.

## Responsabilidades

### Presentation

Inclui Controllers, DTOs, Guards, Pipes e Interceptors.

Controllers não possuem regra de negócio. Eles validam entrada, usam recursos do NestJS para autenticação/autorização e chamam Use Cases.

### Application

Inclui Use Cases, orquestração de fluxos, transações e coordenação entre serviços.

Use Cases são a porta de entrada dos fluxos da aplicação. Eles coordenam repositórios, Domain Services e transações, mas evitam conter cálculos ou invariantes complexas.

### Domain

Inclui entidades, Value Objects, Domain Services, regras de negócio e invariantes.

Domain Services concentram regras que atravessam entidades ou que ficariam artificiais dentro de uma entidade isolada.

### Infrastructure

Inclui Prisma, implementações de repositórios, mapeadores e integrações externas.

Prisma deve aparecer apenas aqui. A application layer consome interfaces de repositório, não implementações concretas.

## Regras de Dependência

Proibido:

```text
Controller -> Prisma
Controller -> Database
Domain -> Prisma
Domain -> NestJS
Domain -> HTTP
```

Permitido:

```text
Controller -> UseCase
UseCase -> Repository Interface
Repository Implementation -> Prisma
```

## Referências

- [Estrutura dos Módulos](module-structure.md)
- [Multi-Tenancy](multi-tenancy.md)
- [Estratégia de Transações](transaction-strategy.md)
- [Arquitetura dos Relatórios](reporting-architecture.md)
- [Domínio de Caixa](cash-session-domain.md)
- [ADR-006: Clean Architecture Pragmática](../.compozy/tasks/restora-saas-marmitarias/adrs/adr-006.md)
