# ADR-007: Prisma como ORM em pedidos

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-005](../PRODUCTS/ADR-005-prisma-como-camada-de-dados-de-produtos.md), [ADR-014](../CUSTOMERS/ADR-014-prisma-para-dados-de-clientes.md), [ADR-017](../DATA/ADR-017-prisma-como-orm.md)

## Contexto e Declaração do Problema

O módulo ORDERS utiliza Prisma na camada de persistência. As consultas e operações transacionais observadas dependem das APIs e dos tipos do ORM. Há transações que abrangem a criação do pedido e alterações de status associadas ao histórico e ao estoque.

As evidências confirmam o uso atual, mas não registram a motivação da adoção nem uma aprovação formal. **[NECESSITA DE INFORMAÇÃO]** Quais requisitos ou problemas motivaram a escolha do Prisma para ORDERS?

## Fatores de Decisão

- O uso atual do Prisma está evidenciado na persistência do módulo ORDERS.
- As operações observadas usam transações para manter agrupadas alterações relacionadas ao pedido, histórico e estoque.
- O acoplamento a APIs e tipos específicos do ORM deve ser considerado em futuras mudanças na camada de persistência.
- **[NECESSITA DE INFORMAÇÃO]** Quais requisitos de persistência, consistência e manutenção devem orientar a escolha do ORM?

## Opções Consideradas

1. **Prisma:** tecnologia em uso no módulo; oferece consultas tipadas e mecanismos transacionais observados no código.
2. **Alternativa de ORM não identificada:** **[NECESSITA DE INFORMAÇÃO]** A análise não registra nomes de alternativas nem critérios ou resultados de comparação.

## Resultado da Decisão

Propõe-se registrar o Prisma como ORM do módulo ORDERS e manter o uso observado no escopo desse módulo. O status permanece proposto porque não há evidência da justificativa original nem de aprovação formal. Esta decisão não afirma uma escolha comum a outros módulos.

## Prós e Contras das Opções

### Prisma

- **Prós:** fornece consultas tipadas e operações transacionais usadas na persistência observada de ORDERS.
- **Contras:** tipos e APIs específicos do ORM criam acoplamento; uma substituição exigiria rever a persistência e os limites das transações do módulo.

### Alternativa de ORM não identificada

- **Prós e contras:** não podem ser comparados com as evidências disponíveis; nenhuma alternativa concreta foi documentada.

## Consequências

- A evolução da persistência de ORDERS depende de conhecimento das APIs e dos tipos do Prisma.
- Uma eventual substituição do ORM exigiria rever o acesso a dados e os limites transacionais observados no módulo.
- **[NECESSITA DE INFORMAÇÃO]** O impacto da escolha em outros módulos não foi avaliado.

## Referências

1. [Potencial ADR: Prisma como ORM no módulo ORDERS](../../potential-adrs/done/ORDERS/prisma-orm-orders.md).
2. [Código da persistência de ORDERS](../../../src/modules/orders/order.repository.ts).
3. [Código dos serviços de ORDERS](../../../src/modules/orders/order.service.ts).
