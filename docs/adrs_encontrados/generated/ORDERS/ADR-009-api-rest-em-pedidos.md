# ADR-009: API HTTP em estilo REST no módulo de pedidos

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-004](../PRODUCTS/ADR-004-api-rest-para-catalogo-de-produtos.md), [ADR-011](../API/ADR-011-api-rest-versionada.md), [ADR-015](../CUSTOMERS/ADR-015-api-rest-para-clientes.md)

## Contexto e Declaração do Problema

No módulo de pedidos, as rotas observadas usam recursos HTTP com os métodos GET, POST, PATCH e DELETE, e as respostas incluem os códigos 200, 201 e 204. Esse conjunto é compatível com um estilo de API HTTP REST, mas não demonstra conformidade completa com todas as restrições de REST.

É necessário registrar o estilo visível nos contratos HTTP desse módulo sem atribuir uma motivação não documentada nem generalizar a decisão para a API inteira. As referências disponíveis não explicam por que esse estilo foi adotado.

## Fatores de Decisão

- O escopo evidenciado limita-se às rotas e às respostas HTTP do módulo de pedidos.
- O uso de métodos e recursos HTTP oferece uma interface direta para as operações observadas.
- Os consumidores permanecem vinculados aos endpoints e às semânticas de status; o impacto de mudanças sobre consumidores externos não foi analisado.
- [NECESSITA DE INFORMAÇÃO] Quais requisitos ou critérios motivaram a escolha desse estilo de API?
- [NECESSITA DE INFORMAÇÃO] Quais consumidores dependem dos endpoints atuais e quais compromissos de compatibilidade existem?

## Opções Consideradas

1. **API HTTP em estilo REST, conforme observado no módulo de pedidos.** As rotas e os códigos de resposta disponíveis sustentam essa caracterização limitada, sem comprovar conformidade REST completa.
2. **Outro estilo de API.** Nenhuma alternativa foi identificada nas referências examinadas; não há evidência de que esta ou outras opções tenham sido avaliadas.

## Resultado da Decisão

Propõe-se registrar a API HTTP em estilo REST como a caracterização do contrato atualmente observado no módulo de pedidos. O registro fica restrito a esse módulo e não declara conformidade completa com REST nem uma decisão comum a outros módulos.

- [NECESSITA DE INFORMAÇÃO] A motivação da escolha e a avaliação de alternativas precisam ser confirmadas antes de tratar a proposta como decisão aprovada.
- [NECESSITA DE INFORMAÇÃO] É necessário confirmar se o estilo também é uma decisão comum a outros módulos.

## Prós e Contras das Opções

### API HTTP em estilo REST, conforme observado

- **Prós:** os recursos e métodos HTTP oferecem uma interface direta para as operações observadas.
- **Contras:** consumidores ficam vinculados aos endpoints e às semânticas de status; alterar o estilo exigiria rever rotas e contratos, e o impacto sobre consumidores externos não está determinado.

### Outro estilo de API

- **Prós:** não determináveis com as evidências disponíveis.
- **Contras:** não determináveis com as evidências disponíveis; nenhuma alternativa específica ou avaliação foi registrada.

## Consequências

- O estilo descrito pode orientar a leitura e a evolução dos contratos HTTP do módulo de pedidos, sem implicar conformidade integral com REST.
- Mudanças nas rotas ou nas semânticas de resposta podem afetar consumidores; não há inventário de consumidores nem análise de compatibilidade disponível.
- O escopo desta proposta não estabelece uma escolha para outros módulos.

## Referências

1. Potencial ADR e limites da análise: [`api-rest-orders.md`](../../potential-adrs/done/ORDERS/api-rest-orders.md).
2. Rotas HTTP do módulo: [`order.routes.ts`](../../../../src/modules/orders/order.routes.ts).
3. Respostas HTTP do módulo: [`order.controller.ts`](../../../../src/modules/orders/order.controller.ts).
