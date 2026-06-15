# Estrutura dos Módulos

Todo módulo do backend deve seguir a mesma separação de camadas descrita em [Arquitetura do Backend](backend-architecture.md).

```text
feature/
  presentation/
    controllers/
    dto/
  application/
    use-cases/
    ports/
  domain/
    entities/
    services/
    value-objects/
  infrastructure/
    repositories/
    mappers/
  feature.module.ts
```

## Exemplo

```text
cash-sessions/
  presentation/
    controllers/
      cash-sessions.controller.ts
    dto/
  application/
    use-cases/
      open-cash-session.use-case.ts
      record-payment-totals.use-case.ts
      close-cash-session.use-case.ts
      create-cash-adjustment.use-case.ts
    ports/
      cash-session.repository.ts
  domain/
    entities/
      cash-session.entity.ts
    services/
      cash-close-calculator.service.ts
    value-objects/
      money.vo.ts
  infrastructure/
    repositories/
      prisma-cash-session.repository.ts
    mappers/
      cash-session.mapper.ts
  cash-sessions.module.ts
```

## Use Cases Esperados

- `OpenCashSessionUseCase`
- `RecordPaymentTotalsUseCase`
- `CreateWithdrawalUseCase`
- `CreateExpenseUseCase`
- `RecordAttendanceUseCase`
- `CloseCashSessionUseCase`
- `CreateCashAdjustmentUseCase`
- `GenerateDailyReportUseCase`
- `GenerateMonthlyReportUseCase`

## Regras

- Controllers chamam Use Cases.
- Use Cases dependem de ports/interfaces, não de Prisma diretamente.
- Implementações Prisma ficam em `infrastructure/repositories`.
- Mapeadores traduzem entre modelos Prisma e objetos de domínio/DTOs.
- Criar Value Object apenas quando houver regra real, como dinheiro, datas de negócio ou status.
