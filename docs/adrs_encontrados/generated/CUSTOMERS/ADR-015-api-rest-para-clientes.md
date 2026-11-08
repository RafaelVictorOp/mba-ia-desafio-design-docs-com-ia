# ADR-015: API REST para clientes

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-004](../PRODUCTS/ADR-004-api-rest-para-catalogo-de-produtos.md), [ADR-009](../ORDERS/ADR-009-api-rest-em-pedidos.md), [ADR-011](../API/ADR-011-api-rest-versionada.md)

## Contexto e Declaração do Problema

O módulo CUSTOMERS expõe operações de consulta e manutenção de clientes por uma interface HTTP compatível com o estilo REST. As evidências caracterizam o padrão observado, mas não registram a motivação da escolha nem uma decisão formal. Também não demonstram conformidade integral com todas as restrições de REST.

É necessário documentar essa convenção sem atribuir uma justificativa histórica não comprovada nem estendê-la automaticamente aos demais módulos.

## Fatores de Decisão

- O estilo observado define contratos HTTP de clientes usados por consumidores da aplicação.
- Mudanças no estilo podem afetar esses contratos e seus consumidores; não há inventário de consumidores nem análise de impacto disponível.
- As evidências consultadas não registram critérios de compatibilidade ou evolução dos contratos.
- [NECESSITA DE INFORMAÇÃO: confirmar os requisitos que motivaram a escolha do estilo de API.]
- [NECESSITA DE INFORMAÇÃO: confirmar se outros protocolos ou estilos foram avaliados e quais critérios foram usados.]
- [NECESSITA DE INFORMAÇÃO: definir garantias de compatibilidade e evolução dos contratos para consumidores existentes.]

## Opções Consideradas

1. **API HTTP orientada a recursos, compatível com REST.** É a caracterização sustentada pelas operações observadas no módulo; não há registro de que tenha sido escolhida após comparação.
2. **Outro estilo ou protocolo de API.** Nenhuma alternativa foi documentada nas evidências consultadas; não é possível afirmar que opções alternativas tenham sido avaliadas.

## Resultado da Decisão

Propõe-se registrar a API HTTP de CUSTOMERS como compatível com o estilo REST, conforme o padrão observado. Este registro descreve a interface existente, não comprova a motivação histórica, uma comparação entre alternativas ou conformidade integral com REST.

- [NECESSITA DE INFORMAÇÃO: confirmar se o estilo deve ser tratado como decisão comum da API ou restrito ao módulo CUSTOMERS.]

## Prós e Contras das Opções

### API HTTP orientada a recursos, compatível com REST

- **Prós:** fornece uma convenção observável para as operações de clientes e seus contratos HTTP.
- **Contras:** consumidores dependem dos contratos atuais; mudanças de estilo podem afetá-los, e não há informação suficiente para estimar esse impacto.

### Outro estilo ou protocolo de API

- **Prós e contras:** não determináveis com as evidências disponíveis; nenhuma alternativa específica ou avaliação foi registrada.

## Consequências

- O estilo descrito pode orientar a leitura e a evolução dos contratos HTTP de CUSTOMERS, sem implicar conformidade integral com REST.
- Alterações nesses contratos devem considerar possíveis efeitos sobre consumidores, cujo inventário e garantias de compatibilidade não estão documentados.
- Esta proposta limita-se a CUSTOMERS e não estabelece uma convenção para os demais módulos.
- Motivação, avaliação de alternativas, compatibilidade e abrangência da decisão permanecem pendentes de validação.

## Referências

1. `docs/adrs/potential-adrs/done/CUSTOMERS/api-rest-clientes.md` — potencial ADR, evidências e lacunas que fundamentam esta proposta.
2. `src/modules/customers/customer.routes.ts` — evidência das operações HTTP observadas no módulo.
3. `src/modules/customers/customer.controller.ts` — evidência das respostas HTTP do módulo.
4. `docs/adrs/potential-adrs/done/API/api-rest-versionada.md` — análise relacionada que recomenda avaliar uma decisão compartilhada de estilo e versionamento da API; permanece um candidato, não um ADR formal.
