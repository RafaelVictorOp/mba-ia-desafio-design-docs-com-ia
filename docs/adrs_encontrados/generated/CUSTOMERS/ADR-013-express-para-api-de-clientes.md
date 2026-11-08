# ADR-013: Express para a API de clientes

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-003](../PRODUCTS/ADR-003-express-como-framework-http.md), [ADR-008](../ORDERS/ADR-008-express-como-framework-em-pedidos.md), [ADR-010](../API/ADR-010-express-como-framework-principal.md)

## Contexto e Declaração do Problema

O módulo CUSTOMERS e a composição central da aplicação usam Express na camada HTTP. O código confirma essa escolha em uso, mas não registra sua motivação nem uma aprovação formal. É necessário documentar a plataforma observada sem atribuir à equipe razões ou avaliações não comprovadas.

[NECESSITA DE INFORMAÇÃO: quais necessidades ou restrições levaram à escolha do Express para a API de clientes?]

## Fatores de Decisão

- O Express é usado no roteamento HTTP de CUSTOMERS e na composição central da aplicação.
- O código consultado não registra critérios de comparação entre frameworks.
- A análise limita-se ao módulo CUSTOMERS e à composição central necessária para confirmar o framework.

[NECESSITA DE INFORMAÇÃO: quais critérios técnicos ou operacionais devem orientar a confirmação desta decisão?]

## Opções Consideradas

1. **Express:** tecnologia observada no módulo e na aplicação.
2. **Outro framework HTTP:** nenhuma alternativa concreta ou avaliação comparativa está registrada.

[NECESSITA DE INFORMAÇÃO: quais alternativas foram efetivamente avaliadas e por que foram aceitas ou descartadas?]

## Resultado da Decisão

Registrar o Express como framework HTTP em uso na API de clientes. O status permanece como proposta porque as fontes confirmam o uso, mas não demonstram aprovação formal nem a justificativa original. A confirmação da decisão depende do preenchimento das lacunas registradas.

## Prós e Contras das Opções

- **Express**
  - **Prós:** corresponde à plataforma já usada pelo módulo CUSTOMERS e pela composição central, conforme o código consultado.
  - **Contras:** vincula a integração HTTP do módulo ao framework; uma substituição exigiria adaptar essa integração.
- **Outro framework HTTP**
  - **Prós e contras:** não avaliáveis sem identificar alternativas concretas e os critérios aplicados.

## Consequências

- A API de clientes permanece vinculada ao framework HTTP atualmente observado, enquanto esta proposta não for substituída por uma decisão validada.
- Uma eventual troca exigirá adequações no módulo e na composição central; o esforço e os efeitos sobre outros módulos não foram determinados.

[NECESSITA DE INFORMAÇÃO: quais impactos de longo prazo, condições para eventual substituição e efeitos fora de CUSTOMERS devem ser considerados?]

## Referências

1. `src/modules/customers/customer.routes.ts` — uso do Express na camada HTTP de CUSTOMERS.
2. `src/modules/customers/customer.controller.ts` — uso do Express na camada de handlers HTTP de CUSTOMERS.
3. `src/app.ts` — uso do Express na composição central da aplicação.
4. `package.json` — dependência Express declarada pela aplicação.
