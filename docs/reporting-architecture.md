# Arquitetura dos Relatórios

Relatórios do Restora usam snapshots para preservar histórico financeiro confiável.

## Durante Caixa Aberto

Dados operacionais podem vir de:

- totais de pagamento;
- sangrias;
- despesas;
- presença e custo diário de funcionários.

Esses dados podem mudar enquanto o caixa está aberto.

## Após Fechamento

Relatórios históricos devem usar exclusivamente:

- snapshot persistido no fechamento;
- ajustes owner-only feitos após o fechamento.

Não recalcular relatórios históricos diretamente a partir de tabelas operacionais após o caixa ser fechado.

## Visibilidade

- Dono acessa relatórios diários, mensais, histórico e exportações.
- Caixa pode ver os totais operacionais da sessão atual necessários para revisar e fechar o caixa.
- Caixa não acessa relatórios mensais, histórico amplo ou ajustes pós-fechamento.

## Referências

- [Domínio de Caixa](cash-session-domain.md)
- [ADR-003: Immutable Cash Sessions with Owner Adjustments](../.compozy/tasks/restora-saas-marmitarias/adrs/adr-003.md)
- [ADR-005: Cashier-Enabled Cash Session Close](../.compozy/tasks/restora-saas-marmitarias/adrs/adr-005.md)
