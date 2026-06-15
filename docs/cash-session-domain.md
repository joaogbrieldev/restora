# Domínio de Caixa

O fechamento de caixa é o eixo principal da Fase 1.

## Máquina de Estados

```text
OPEN
  ↓
CLOSED
```

Não existe operação de reopen. Correções após fechamento são ajustes owner-only.

## Fórmula de Lucro

```text
lucro_estimado =
vendas
- sangrias
- despesas
- pessoal
+ ajustes
```

Todos os valores monetários são armazenados em centavos inteiros.

## Invariantes

- Existe no máximo uma sessão de caixa por restaurante/dia.
- Caixa fechado é imutável.
- Fechamento persiste snapshot dos totais.
- Ajustes pós-fechamento são exclusivos do owner.
- Fechamento é transacional.
- Relatórios históricos usam snapshot de fechamento mais ajustes.
- O usuário que fechou o caixa deve ser registrado em `closed_by_user_id`.

## Permissões

- Owner e cashier podem abrir, preencher dados operacionais e fechar a sessão atual.
- Owner controla ajustes pós-fechamento, relatórios históricos, configurações e custos sensíveis.
- Cashier pode ver o lucro estimado da sessão atual apenas para revisar o fechamento.

## Referências

- [ADR-003: Immutable Cash Sessions with Owner Adjustments](../.compozy/tasks/restora-saas-marmitarias/adrs/adr-003.md)
- [ADR-005: Cashier-Enabled Cash Session Close](../.compozy/tasks/restora-saas-marmitarias/adrs/adr-005.md)
- [ADR-007: Representação Monetária](../.compozy/tasks/restora-saas-marmitarias/adrs/adr-007.md)
