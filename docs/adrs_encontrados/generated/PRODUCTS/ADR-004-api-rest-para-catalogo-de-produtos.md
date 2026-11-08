# ADR-004: API REST para o catálogo de produtos

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-009](../ORDERS/ADR-009-api-rest-em-pedidos.md), [ADR-011](../API/ADR-011-api-rest-versionada.md), [ADR-015](../CUSTOMERS/ADR-015-api-rest-para-clientes.md)

## 1. Contexto e Declaração do Problema

O módulo PRODUCTS expõe operações HTTP para consultar a coleção e um produto por identificador, criar, atualizar parcialmente e excluir produtos. A combinação de recursos com métodos HTTP distintos é compatível com o estilo REST, mas essa classificação é inferida das rotas: o código não nomeia explicitamente o estilo nem registra a motivação da escolha.

Como o contrato HTTP do catálogo é consumido por clientes da API, a convenção precisa ser compreensível e alterações devem considerar compatibilidade. A evidência disponível se limita a PRODUCTS; não demonstra se há uma decisão compartilhada com outros módulos.

## 2. Fatores de Decisão

- Impacto no contrato HTTP do catálogo consumido por clientes da API.
- Efeito de uma mudança de estilo sobre rotas e clientes; o esforço não pode ser estimado com as evidências disponíveis.
- Necessidade de entendimento da convenção por quem integra ou mantém o catálogo.

## 3. Opções Consideradas

1. **API HTTP orientada a recursos, compatível com REST:** é a abordagem observável nas rotas do catálogo. A evidência não confirma que tenha sido escolhida após comparação.
2. **Alternativas de protocolo ou estilo:** não foram registradas nas evidências consultadas. Não é possível afirmar que tenham sido avaliadas.

## 4. Resultado da Decisão

Registra-se, com status proposto, a classificação da API HTTP do catálogo como compatível com REST, conforme as operações observadas. Isso documenta a convenção aparente, não uma motivação histórica comprovada nem uma comparação entre alternativas.

- [NECESSITA DE INFORMAÇÃO: confirmar quais requisitos motivaram o uso de endpoints HTTP orientados a recursos.]
- [NECESSITA DE INFORMAÇÃO: confirmar se outros protocolos ou estilos foram avaliados e quais critérios foram usados.]
- [NECESSITA DE INFORMAÇÃO: definir quais garantias de compatibilidade do contrato devem ser preservadas para os consumidores existentes.]
- [NECESSITA DE INFORMAÇÃO: confirmar se há uma decisão de API compartilhada com outros módulos antes de tratar esta convenção como abrangente ao sistema.]

## 5. Prós e Contras das Opções

### API HTTP orientada a recursos, compatível com REST

**Prós**

- As operações do catálogo estão organizadas em torno da coleção e de produtos individuais, com métodos HTTP distintos.
- A classificação é compatível com o estilo REST, conforme a análise das rotas documentada na evidência.

**Contras**

- Uma mudança de estilo afetaria rotas e clientes; o esforço correspondente não está estimado.
- Não há registro dos motivos da escolha nem de comparação que permita avaliar vantagens frente a alternativas.

### Alternativas de protocolo ou estilo

**Prós e contras:** não documentados; não há evidência suficiente para caracterizá-los.

## 6. Consequências

- O contrato HTTP do catálogo deve ser compreendido por quem o integra ou mantém.
- Alterações futuras devem considerar compatibilidade com consumidores existentes.
- O registro descreve PRODUCTS e não estabelece, sem confirmação, uma convenção para outros módulos.
- Motivação, alternativas avaliadas, garantias de compatibilidade e abrangência da decisão permanecem pendentes de validação.

## 7. Referências

1. `docs/adrs/potential-adrs/done/PRODUCTS/api-rest-para-catalogo-produtos.md` — potencial ADR e origem das lacunas e impactos registrados.
2. `src/modules/products/product.routes.ts` — evidência das rotas do catálogo.
3. `docs/adrs/potential-adrs/done/PRODUCTS/express-como-framework-http-do-modulo.md` — potencial ADR relacionado; não é uma decisão formal.
4. `docs/adrs/potential-adrs/done/PRODUCTS/prisma-como-camada-de-acesso-a-dados.md` — potencial ADR relacionado; não é uma decisão formal.
