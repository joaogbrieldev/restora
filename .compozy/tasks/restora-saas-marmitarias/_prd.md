# PRD: Restora para Marmitarias

## 1. Visão Geral e Problema

Restora é um SaaS multi-tenant para restaurantes, com foco inicial em marmitarias brasileiras, restaurantes, lanchonetes, que substitui controles em papel por um painel simples de operação financeira e, em fases posteriores, adiciona pedidos online por link para clientes finais.

O problema inicial é operacional: a marmitaria precisa registrar despesas, presença de funcionários, vendas manuais de balcão e fechamento de caixa sem depender de caderno, memória ou planilhas soltas. O primeiro produto deve provar valor em uma marmitaria real antes de expandir para pedidos online e onboarding self-serve.

Premissas:

- O primeiro piloto será operado por dono e caixa/funcionário.
- A Fase 1 não integra pagamentos, WhatsApp, emissão fiscal, marketplaces ou estoque.
- Vendas manuais de balcão entram como totais por método de pagamento no fechamento, não como pedidos itemizados.

## 2. Objetivos e Métricas de Sucesso

Objetivos:

- Substituir o fechamento de caixa em papel da marmitaria, restaurantes, lanchonetes piloto por um fluxo digital diário simples.
- Permitir que despesas, sangrias e custo de pessoal sejam considerados no lucro do dia e no acumulado do mês.
- Dar ao dono uma visão confiável de lucro diário/mensal sem exigir operação complexa.
- Preparar a base operacional para receber pedidos online, CRM e pagamentos na Fase 2.

Métricas da Fase 1:

- Dono fecha o caixa no Restora por 14 dias corridos de operação sem voltar ao papel como fonte principal.
- Fechamento diário concluído em até 10 minutos em pelo menos 80% dos dias do piloto.
- 100% dos dias operados no piloto têm registro de abertura e fechamento.
- Despesas e presença do dia são registradas antes ou durante o fechamento em pelo menos 80% dos dias.
- Dono declara confiança nos números de lucro diário e acumulado mensal ao final do piloto.

## 3. Personas e Dores

### Dono da marmitaria

- Dor: fecha caixa em papel, perde histórico, mistura faturamento com lucro e não enxerga custo real do dia.
- Necessidade: abrir/fechar caixa, lançar despesas/sangrias, ver lucro diário/mensal e controlar acesso.

### Caixa/funcionário

- Dor: anota dados de operação de forma solta e depende de instruções verbais.
- Necessidade: registrar informações simples do dia sem acessar dados sensíveis.

### Cliente final

- Dor: pedir por WhatsApp pode gerar erro, demora e repetição de endereço.
- Necessidade: acessar link, montar pedido, identificar-se por telefone e acompanhar status.

## 4. Escopo por Fase

### Fase 1: MVP gestão interna caixa-first

Entra:

- Abertura e fechamento de caixa.
- Vendas manuais por método de pagamento.
- Despesas e sangrias.
- Funcionários e presença do dia.
- Custo de pessoal no fechamento.
- Resumo de lucro diário e acumulado mensal.
- Papéis dono e caixa/funcionário.
- Configuração de dados do restaurante e categorias de despesas.

Fica fora:

- App do cliente, catálogo, carrinho, Pix integrado, WhatsApp automático, marketplaces, fiscal, estoque, CRM e cobrança SaaS.

### Fase 2: Pedidos do cliente

Entra:

- Catálogo isolado por restaurante.
- Link compartilhável.
- Carrinho, identificação por telefone e endereços salvos.
- Pedidos com snapshot de preço.
- Pagamento Pix via Asaas.
- Pagamento na entrega.
- Kanban de pedidos.
- CRM leve.

Fica fora:

- Bot WhatsApp automático, iFood/marketplaces, fidelidade, cupons, app nativo e emissão fiscal.

### Fase 3: Self-serve e monetização

Entra:

- Onboarding autônomo de restaurantes.
- Configuração inicial guiada.
- Gestão de assinatura/cobrança do SaaS.
- Materiais de ativação.

Fica fora:

- Franquias/redes complexas, automações avançadas de marketing e integrações enterprise.

## 5. Requisitos Funcionais por Módulo

### Financeiro e caixa

- FR1. Permitir abrir uma sessão de caixa diária por restaurante.
- FR2. Registrar totais de vendas manuais por método de pagamento (dinheiro, PIX, cartão débito, cartão crédito, vale refeição/alimentação).
- FR3. Registrar sangrias com valor, horário, responsável e observação.
- FR4. Lançar despesas com data, categoria, valor, descrição e responsável.
- FR5. Calcular fechamento do dia com vendas, sangrias, despesas e custo de pessoal.
- FR6. Mostrar lucro estimado do dia e acumulado do mês.
- FR7. Manter histórico de caixas fechados.
- FR8. Permitir corrigir dados antes do fechamento final.
- FR9. Preservar caixa fechado como histórico, com ajustes posteriores visíveis ao dono.

### Pessoas

- FR10. Cadastrar funcionários com nome, função, status e custo diário ou custo simples.
- FR11. Registrar presença dos funcionários no dia.
- FR12. Usar presença para calcular custo de pessoal no fechamento.
- FR13. Permitir consulta de presença por dia.

### Painel (dashboard)

- FR14. Exibir painel inicial com status do caixa do dia (aberto/fechado, horário de abertura), resumo de vendas, despesas, pessoal e lucro estimado, e avisos de pendências (dias sem fechamento, presença não registrada).
- FR15. Disponibilizar atalhos rápidos no painel para lançar despesa, registrar sangria e abrir/fechar caixa.

### Relatórios

- FR16. Exibir resumo do dia com vendas, despesas, sangrias, pessoal e lucro.
- FR17. Exibir acumulado mensal de faturamento, despesas, pessoal e lucro.
- FR18. Permitir filtro simples por período.
- FR19. Destacar dias sem fechamento ou com dados incompletos.
- FR20. Permitir exportar o relatório do período selecionado (CSV ou PDF).

### Acesso

- FR21. Suportar papéis dono e caixa/funcionário na Fase 1.
- FR22. Dono acessa relatórios, cadastros, histórico e configurações.
- FR23. Caixa/funcionário registra dados operacionais sem acessar relatórios financeiros completos, salvo permissão do dono.
- FR24. Caixa/funcionário pode fechar o caixa do dia sozinho, sem exigir confirmação do dono; ajustes pós-fechamento continuam exclusivos do dono (ver FR9).
- FR25. Permitir cadastrar funcionário sem acesso ao sistema (apenas para controle de presença e custo de pessoal), distinto do funcionário com login e papel definido.

### Configurações

- FR26. Permitir editar dados cadastrais do restaurante (nome, telefone, endereço, horário de funcionamento).
- FR27. Permitir configurar categorias de despesas do restaurante, com conjunto padrão pré-configurado (Insumos, Infraestrutura, Manutenção, Outros).
- FR28. Permitir ao dono visualizar e gerenciar o papel de acesso de cada usuário (dono, caixa, sem acesso).

### Catálogo e loja, Fase 2

- FR29. Configurar categorias, produtos, adicionais, preços e disponibilidade.
- FR30. Configurar horário, taxa/raio de entrega, métodos de pagamento e pausa de pedidos.
- FR31. Expor catálogo isolado por restaurante via link compartilhável.

### Pedidos e clientes, Fase 2

- FR32. Cliente acessa link, navega no catálogo, monta carrinho e informa telefone.
- FR33. Cliente informa ou seleciona endereço salvo.
- FR34. Pedido guarda snapshot de preços, itens e adicionais.
- FR35. Cliente escolhe Pix via Asaas ou pagamento na entrega.
- FR36. Pedido Pix aguarda confirmação assíncrona.
- FR37. Restaurante acompanha pedidos em kanban.
- FR38. CRM leve por telefone, últimos pedidos, endereços e frequência.

### SaaS, Fase 3

- FR39. Novo restaurante cria conta sem intervenção manual.
- FR40. Restaurante configura loja, usuários e métodos iniciais.
- FR41. Produto suporta cobrança recorrente do SaaS.

## 6. Fluxos Principais

### Dono fechando o caixa

1. Dono ou caixa abre o caixa do dia.
2. Equipe registra despesas, sangrias e presença quando necessário.
3. No fim do expediente, responsável informa totais por método de pagamento.
4. Sistema mostra entradas, sangrias, despesas, custo de pessoal e lucro estimado.
5. Responsável revisa inconsistências simples e fecha o caixa.
6. Dono consulta resultado do dia e acumulado mensal.

### Cliente fazendo pedido pelo link

1. Cliente recebe link do restaurante pelo WhatsApp.
2. Cliente acessa catálogo sem instalar app.
3. Cliente escolhe produtos, adicionais e quantidades.
4. Cliente informa telefone para identificação.
5. Cliente informa endereço ou seleciona endereço salvo.
6. Cliente escolhe Pix ou pagamento na entrega.
7. Sistema registra pedido com snapshot de preço.
8. Cliente acompanha status.
9. Restaurante gerencia pedido no kanban.

## 7. Requisitos Não-Funcionais

- Multi-tenancy deve seguir schema único no PostgreSQL, `restaurant_id` nas tabelas de tenant, RLS e `restaurant_id`/role no JWT.
- Stack da Fase 1: monorepo Turborepo, Next.js, NestJS, Prisma + PostgreSQL. Redis e BullMQ ficam fora da Fase 1 até existir necessidade real de jobs assíncronos.
- Dados financeiros e de clientes devem ser isolados por restaurante.
- Ações financeiras sensíveis exigem autenticação e permissão compatível.
- Telas críticas da Fase 1 devem responder em até 2 segundos em condições normais.
- Fase 2 deve tratar Pix via Asaas com confirmação assíncrona por webhook.
- Pagamento na entrega deve ser método de primeira classe.
- Pedidos devem preservar snapshot de preço.
- O MVP não deve depender de bot, envio automático ou aprovação da WhatsApp Business API.
- Fechamento de caixa deve contabilizar vendas manuais de balcão, não só pedidos online.

## 8. Riscos e Dependências

- Adoção: dono pode voltar ao papel por hábito. Mitigação: fechamento simples, curto e com linguagem do dia a dia.
- Escopo: produto pode crescer para competir com suites completas cedo demais. Mitigação: manter Fase 1 caixa-first.
- Qualidade dos dados: lançamentos manuais incompletos podem reduzir confiança. Mitigação: destacar dias incompletos.
- Equipe: funcionários podem resistir a registrar presença/dados. Mitigação: permissões simples e telas com poucas ações.
- WhatsApp: Business API tem custo e aprovação. Mitigação: usar apenas link compartilhável nas fases iniciais.
- Pagamentos: Pix depende de confirmação assíncrona do Asaas. Mitigação: status claro para cliente e restaurante.
- Concorrência: players brasileiros oferecem suites completas. Mitigação: posicionar Restora como opção simples para quem ainda opera no papel.
- Capacidade: 1 dev limita velocidade. Mitigação: faseamento rígido.

## 9. Fora de Escopo no MVP

- App do cliente final.
- Catálogo, carrinho e pedidos por link.
- Pix integrado e webhooks de pagamento.
- Automação de WhatsApp, bot ou envio ativo de mensagens.
- Integração com iFood, Rappi, aiqfome ou marketplaces.
- Emissão fiscal, NFC-e ou integração contábil.
- Estoque, CMV detalhado e ficha técnica.
- Programa de fidelidade, cupons, cashback e campanhas.
- Relatórios de mais vendidos sem vendas itemizadas.
- Onboarding self-serve e cobrança SaaS.
- App mobile nativo, totem, QR Code de mesa e chatbot com IA.

## 10. Architecture Decision Records

- [ADR-001: Priorizar MVP caixa-first](adrs/adr-001.md) — Define fechamento de caixa diário simples como eixo da Fase 1 e principal mecanismo de validação.

## 11. Open Questions

- Qual custo diário ou modelo de remuneração deve ser usado para funcionários no piloto? (o design de referência assume custo diário fixo por funcionário; falta decidir se um modelo por hora ou variável será necessário.)
- ~~O dono precisa reabrir caixa fechado ou apenas registrar ajuste posterior?~~ Resolvido: apenas ajuste posterior, caixa fechado não é reaberto (ver FR9).
- ~~Quais categorias mínimas de despesas devem vir pré-configuradas?~~ Resolvido: Insumos, Infraestrutura, Manutenção e Outros, editável pelo dono (ver FR26).
- ~~O caixa/funcionário poderá fechar o caixa sozinho ou apenas preparar dados para o dono confirmar?~~ Resolvido: o caixa/funcionário pode fechar sozinho, sem confirmação do dono (ver FR24).
- Na Fase 2, pedido Pix deve entrar no kanban antes ou apenas depois da confirmação de pagamento?
