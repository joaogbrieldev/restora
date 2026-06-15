# Estratégia de Transações

Operações que alteram dinheiro, permissões ou snapshots devem ser atômicas.

## Operações Obrigatoriamente Transacionais

- Fechamento de caixa.
- Criação de ajuste pós-fechamento.
- Provisionamento de usuário ligado a funcionário.
- Registro de presença quando afeta cálculo de custo do dia.
- Exclusão ou soft-delete de categoria de despesa.

## Fechamento de Caixa

```text
BEGIN
  lock cash_session
  validate tenant and status
  calculate totals from operational entries
  persist close snapshot
  mark session as CLOSED
COMMIT
```

Implementação esperada:

```ts
await prisma.$transaction(async (tx) => {
  // repository operations using tx
});
```

## Concorrência

O fechamento deve impedir double submit, double close e race conditions. A validação de status precisa acontecer dentro da transação, depois do lock ou da escrita condicional equivalente.

## Referências

- [Domínio de Caixa](cash-session-domain.md)
- [ADR-003: Immutable Cash Sessions with Owner Adjustments](../.compozy/tasks/restora-saas-marmitarias/adrs/adr-003.md)
