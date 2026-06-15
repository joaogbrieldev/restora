# Multi-Tenancy

O Restora usa multi-tenancy em schema compartilhado no PostgreSQL.

## Requisitos

Toda tabela pertencente a um restaurante deve possuir:

```sql
restaurant_id UUID NOT NULL
```

O JWT deve carregar `user_id`, `restaurant_id` e `role`. Guards extraem esse contexto e os repositórios aplicam filtro por `restaurant_id` em toda consulta tenant-scoped.

## Camadas de Proteção

1. JWT com tenant e papel.
2. Guards e decorators de contexto no NestJS.
3. Repositórios sempre filtrando por `restaurant_id`.
4. PostgreSQL RLS quando a estratégia de conexão Prisma estiver definida.

## Resposta Cross-Tenant

Acesso a recurso de outro restaurante deve retornar `404`.

Não retornar `403`, `401`, lista vazia ou `null` para recurso existente em outro tenant, pois isso pode revelar existência de dados fora do restaurante atual.

## Constraints

Constraints únicas em tabelas tenant-scoped devem incluir `restaurant_id`.

```sql
UNIQUE (restaurant_id, business_date)
```

## Referências

- [TechSpec](../.compozy/tasks/restora-saas-marmitarias/_techspec.md)
- [ADR-002: Seeded JWT Authentication for the Pilot](../.compozy/tasks/restora-saas-marmitarias/adrs/adr-002.md)
