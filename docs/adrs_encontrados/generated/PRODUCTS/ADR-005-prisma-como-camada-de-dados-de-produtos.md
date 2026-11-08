# ADR-005: Prisma como camada de dados de produtos

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-007](../ORDERS/ADR-007-prisma-como-orm-em-pedidos.md), [ADR-014](../CUSTOMERS/ADR-014-prisma-para-dados-de-clientes.md), [ADR-017](../DATA/ADR-017-prisma-como-orm.md)

## Contexto e Declaração do Problema
O repositório do módulo PRODUCTS usa Prisma para operações de persistência de produtos e para os tipos associados a esses dados. A evidência disponível confirma o uso no módulo, mas não registra a motivação histórica da escolha nem demonstra que ela se estenda aos demais módulos.

É necessário registrar a solução observada sem atribuir a ela requisitos, critérios ou abrangência que não estejam comprovados.

## Fatores de Decisão
- O código de PRODUCTS realiza operações de consulta, contagem, criação, atualização e exclusão por meio do Prisma.
- O repositório utiliza tipos disponibilizados pelo Prisma para representar produtos e dados de persistência.
- A evidência consultada está limitada ao repositório de produtos.
- A motivação da escolha, as alternativas avaliadas e o escopo de uso fora de PRODUCTS não estão documentados.

## Opções Consideradas
1. **Prisma como camada de acesso a dados de PRODUCTS:** opção atualmente observada no código e formalizada por esta decisão.
2. **Outras opções:** não há evidência disponível de alternativas efetivamente avaliadas. [NECESSITA DE INFORMAÇÃO] Identificar quais alternativas foram consideradas e quais critérios orientaram a comparação.

## Resultado da Decisão
Registrar e manter o Prisma como a camada de acesso a dados atualmente usada pelo módulo PRODUCTS. Esta decisão descreve o estado demonstrado pelo código; não afirma uma justificativa histórica nem uma decisão compartilhada por outros módulos.

## Prós e Contras das Opções
### Prisma como camada de acesso a dados de PRODUCTS
- **Prós:** fornece a camada usada pelas operações de persistência observadas e os tipos associados aos dados de produtos.
- **Contras:** o repositório fica vinculado às APIs e aos tipos do Prisma; substituí-lo exigiria adaptar as operações e os tipos que dependem dessa solução. O esforço dessa mudança não está quantificado.

Não há evidência suficiente para comparar vantagens ou limitações de outras opções.

## Consequências
- A manutenção das operações de persistência e dos tipos do módulo depende do Prisma.
- Uma eventual substituição da camada exigirá adaptar o acesso a dados do repositório; o custo e o esforço não podem ser estimados com as evidências disponíveis.
- [NECESSITA DE INFORMAÇÃO] Confirmar como esta decisão se relaciona com o banco MySQL e com os demais módulos consumidores da persistência.
- [NECESSITA DE INFORMAÇÃO] Registrar os requisitos que motivaram a escolha do Prisma.

## Referências
1. Potencial ADR de origem: `docs\adrs\potential-adrs\done\PRODUCTS\prisma-como-camada-de-acesso-a-dados.md`.
2. Evidência de uso: `src\modules\products\product.repository.ts`.
3. Contexto relacionado, ainda sem caráter de decisão formal: `docs\adrs\potential-adrs\done\PRODUCTS\express-como-framework-http-do-modulo.md`.
4. Contexto relacionado, ainda sem caráter de decisão formal: `docs\adrs\potential-adrs\done\PRODUCTS\api-rest-para-catalogo-produtos.md`.
