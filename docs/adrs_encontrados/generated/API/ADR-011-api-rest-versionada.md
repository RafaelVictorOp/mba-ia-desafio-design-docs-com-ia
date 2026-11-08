# ADR-011: API HTTP REST versionada em `/api/v1`

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-004](../PRODUCTS/ADR-004-api-rest-para-catalogo-de-produtos.md), [ADR-009](../ORDERS/ADR-009-api-rest-em-pedidos.md), [ADR-015](../CUSTOMERS/ADR-015-api-rest-para-clientes.md)

## Contexto e Declaração do Problema

A aplicação se descreve como uma REST API e expõe sua interface de domínio sob o prefixo `/api/v1`, mantendo `/health` separado. A interface compartilhada abrange autenticação, usuários, clientes, produtos e pedidos. Essas evidências demonstram o agrupamento HTTP e a versão no caminho, mas não comprovam que todos os endpoints atendam integralmente às restrições semânticas de REST.

Não há registro da motivação para o estilo REST ou para a versão no caminho. Este ADR registra a convenção observável sem atribuir razões não documentadas nem afirmar conformidade completa.

## Fatores de Decisão

- O prefixo versionado alcança os cinco domínios reunidos pela API.
- A versão é explícita no caminho; o endpoint de saúde permanece fora desse prefixo.
- Alterações de estilo, prefixo ou contrato podem afetar consumidores, mas não há inventário deles nem evidência para estimar o esforço.
- [NECESSITA DE INFORMAÇÃO] Quais requisitos justificaram o estilo REST e a versão no caminho?
- [NECESSITA DE INFORMAÇÃO] Quais consumidores dependem da interface e quais regras de compatibilidade, evolução e descontinuação devem ser observadas?
- [NECESSITA DE INFORMAÇÃO] Quais convenções REST são obrigatórias e como será avaliada a conformidade dos endpoints?

## Opções Consideradas

- **Manter a API HTTP em estilo REST com versão no caminho (`/api/v1`).** É a convenção observada; as evidências não registram a motivação para sua adoção.
- **Adotar outra estratégia de versionamento ou estilo de API.** As referências não identificam uma alternativa específica nem comprovam que alternativas tenham sido avaliadas.

## Resultado da Decisão

Propõe-se registrar e manter, como convenção atualmente observada, a API HTTP em estilo REST sob `/api/v1`, sem incluir `/health` nesse prefixo. A proposta não estabelece política para versões futuras nem declara conformidade REST integral.

## Prós e Contras das Opções

- **Manter a convenção observada — prós:** torna a versão da interface explícita no caminho e reúne os domínios sob um prefixo comum.
- **Manter a convenção observada — contras:** mudanças incompatíveis podem exigir coordenação com consumidores; as regras de compatibilidade e o impacto real não estão documentados.
- **Outra estratégia — prós e contras:** não podem ser determinados, pois não há opção específica nem comparação registrada.

## Consequências

- A convenção registrada oferece uma referência comum para compreender e evoluir a interface dos cinco domínios.
- Mudanças no prefixo, no estilo ou nos contratos devem considerar compatibilidade com consumidores; não há inventário ou política de descontinuação disponível.
- A proposta limita-se à estrutura observável e não define a semântica obrigatória de cada endpoint nem a estratégia de versões futuras.

## Referências

1. Potencial ADR e limites da análise: [`api-rest-versionada.md`](../../potential-adrs/done/API/api-rest-versionada.md).
2. Descrição da aplicação como REST API: [`package.json`](../../../../package.json).
3. Prefixos da interface HTTP e endpoint de saúde: [`app.ts`](../../../../src/app.ts).
4. Agrupamento dos domínios da API: [`index.ts`](../../../../src/routes/index.ts).
