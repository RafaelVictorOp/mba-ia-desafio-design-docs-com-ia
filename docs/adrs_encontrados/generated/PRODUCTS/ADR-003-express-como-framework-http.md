# ADR-003: Express como framework HTTP do módulo PRODUCTS

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-008](../ORDERS/ADR-008-express-como-framework-em-pedidos.md), [ADR-010](../API/ADR-010-express-como-framework-principal.md), [ADR-013](../CUSTOMERS/ADR-013-express-para-api-de-clientes.md)

## 1. Contexto e Declaração do Problema

O módulo PRODUCTS usa o Express para compor suas rotas HTTP e tipar os handlers. As rotas delegam operações ao controller do módulo. A evidência confirma o uso em PRODUCTS, mas não permite concluir que todos os módulos adotam o mesmo framework.

A motivação original da escolha não está registrada. [NECESSITA DE INFORMAÇÃO] Qual necessidade levou à escolha do Express?

## 2. Fatores de Decisão

- O Express já está presente na composição de rotas e handlers HTTP de PRODUCTS.
- A evidência disponível está limitada a PRODUCTS; a abrangência nos demais módulos não foi verificada. [NECESSITA DE INFORMAÇÃO] O Express é uma decisão compartilhada por toda a aplicação?
- Não há registro da motivação ou de uma comparação entre frameworks. [NECESSITA DE INFORMAÇÃO] Quais alternativas foram avaliadas?

## 3. Opções Consideradas

1. Manter o Express como framework HTTP atualmente usado em PRODUCTS.
2. Substituir o Express por outro framework HTTP. A potencial ADR não identifica alternativas específicas nem registra uma avaliação comparativa.

## 4. Resultado da Decisão

Registrar o Express como framework HTTP atualmente usado pelo módulo PRODUCTS e manter esse uso como estado atual. Esta decisão documenta a evidência existente; não afirma uma motivação histórica, uma comparação concluída ou uma escolha global para a aplicação.

## 5. Prós e Contras das Opções

1. **Manter o Express em PRODUCTS**
   - **Prós:** preserva o padrão de rotas e handlers já observado no módulo.
   - **Contras:** mantém a dependência dos componentes HTTP do Express; a justificativa para essa dependência não está documentada.
2. **Substituir por outro framework HTTP**
   - **Prós:** permite reavaliar a escolha no escopo da aplicação, mas não há benefício concreto registrado para uma alternativa.
   - **Contras:** exigiria adaptar rotas e handlers; o esforço e os custos não podem ser quantificados com as evidências disponíveis. [NECESSITA DE INFORMAÇÃO] Quais custos e requisitos de compatibilidade devem orientar uma eventual migração?

## 6. Consequências

- As rotas e os handlers HTTP de PRODUCTS permanecem vinculados ao Express.
- A abrangência da decisão fora de PRODUCTS precisa ser validada antes de tratá-la como padrão global.
- A motivação, a comparação com alternativas e os custos de eventual migração permanecem desconhecidos.

## 7. Referências

- Potencial ADR: `docs/adrs/potential-adrs/done/PRODUCTS/express-como-framework-http-do-modulo.md`.
- Evidência de roteamento: `src/modules/products/product.routes.ts`.
- Evidência de handlers HTTP: `src/modules/products/product.controller.ts`.
