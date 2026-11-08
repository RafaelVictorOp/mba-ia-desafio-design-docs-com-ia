# ADR-008: Express como framework HTTP em Pedidos

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-003](../PRODUCTS/ADR-003-express-como-framework-http.md), [ADR-010](../API/ADR-010-express-como-framework-principal.md), [ADR-013](../CUSTOMERS/ADR-013-express-para-api-de-clientes.md)

## Contexto e Declaração do Problema

O código examinado do módulo ORDERS usa Express na camada HTTP, tanto na organização das rotas quanto na interface dos handlers. Essa evidência confirma a tecnologia em uso no módulo, mas não comprova a motivação da escolha, sua aprovação arquitetural ou uma intenção de longo prazo. O escopo desta decisão limita-se a ORDERS; outros módulos e a composição geral da aplicação não foram avaliados.

Como registrar e validar se o uso atual do Express deve continuar como escolha para a camada HTTP de ORDERS, considerando a dependência observada e os requisitos que essa camada precisa atender?

## Fatores de Decisão

- O Express já está presente na camada HTTP de ORDERS.
- As rotas e os handlers dependem das APIs do framework; uma substituição exigiria adaptação da camada HTTP deste módulo.
- A análise disponível não registra critérios de adequação, requisitos técnicos ou objetivos que tenham motivado a escolha. [NECESSITA DE INFORMAÇÃO: identificar os requisitos e critérios que levaram ao uso do Express em ORDERS.]
- A avaliação deve permanecer limitada ao módulo ORDERS até que haja evidência sobre o uso compartilhado em outros módulos.

## Opções Consideradas

1. **Express:** tecnologia observada na camada HTTP de ORDERS. Não há evidência que explique sua seleção ou comprove uma avaliação formal.
2. **Outro framework HTTP:** alternativa genérica para eventual substituição. Nenhum framework específico ou avaliação dessa opção foi identificado.

## Resultado da Decisão

Registra-se que Express é o framework HTTP atualmente observado no módulo ORDERS. As evidências disponíveis não permitem afirmar que houve comparação entre alternativas nem que a continuidade do uso foi formalmente ratificada. Portanto, a decisão de continuidade permanece pendente de validação; este ADR não prescreve uma migração nem declara uma alternativa escolhida.

## Prós e Contras das Opções

### Express

- **Prós:** o módulo utiliza recursos de roteamento e tipos HTTP fornecidos pelo framework.
- **Contras:** rotas e handlers dependem das APIs do Express, criando acoplamento; a evolução da camada requer conhecimento do framework.

### Outro framework HTTP

- **Prós e contras:** não podem ser comparados com base nas evidências disponíveis. [NECESSITA DE INFORMAÇÃO: indicar quais frameworks foram considerados e os critérios usados para compará-los.]

## Consequências

- A camada HTTP de ORDERS permanece vinculada ao Express enquanto essa tecnologia for mantida.
- Uma eventual substituição exigiria adaptar a camada HTTP do módulo; não há avaliação do impacto além de ORDERS.
- A estimativa preliminar de duas a oito semanas para a adaptação é analítica, não comprovada por registro de execução. [NECESSITA DE INFORMAÇÃO: validar o esforço estimado e seu escopo.]
- Não há evidência sobre uso compartilhado do framework fora de ORDERS. [NECESSITA DE INFORMAÇÃO: confirmar se outros módulos compartilham essa escolha.]

## Referências

1. [Potencial ADR: Express como framework HTTP no módulo ORDERS](../../potential-adrs/done/ORDERS/express-framework-orders.md) — origem da análise; não constitui registro de decisão aprovada.
2. [Rotas HTTP de ORDERS](../../../../src/modules/orders/order.routes.ts) — evidência do uso de Express na camada de rotas.
3. [Handlers HTTP de ORDERS](../../../../src/modules/orders/order.controller.ts) — evidência da dependência do framework na interface dos handlers.
4. [Potencial ADR: Prisma como ORM no módulo ORDERS](../../potential-adrs/done/ORDERS/prisma-orm-orders.md) — referência contextual consultada; não constitui decisão aprovada.
5. [Potencial ADR: API HTTP em estilo REST no módulo ORDERS](../../potential-adrs/done/ORDERS/api-rest-orders.md) — referência contextual consultada; não constitui decisão aprovada.
