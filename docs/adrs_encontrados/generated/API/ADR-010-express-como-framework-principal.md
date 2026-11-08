# ADR-010: Express como framework principal da aplicação HTTP

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-003](../PRODUCTS/ADR-003-express-como-framework-http.md), [ADR-008](../ORDERS/ADR-008-express-como-framework-em-pedidos.md), [ADR-013](../CUSTOMERS/ADR-013-express-para-api-de-clientes.md)

## Contexto e Declaração do Problema

A aplicação HTTP usa Express como framework principal. O manifesto declara Express como dependência direta. A composição observada integra middleware e roteamento de cinco domínios; a inicialização do servidor está separada da construção da aplicação.

Este ADR registra a escolha estrutural evidenciada pelo estado atual do código. As fontes consultadas não comprovam a motivação original nem uma aprovação formal anterior.

## Fatores de Decisão

- Express é o framework central observado na aplicação HTTP.
- O framework participa da composição de middleware e das rotas dos domínios.
- [NECESSITA DE INFORMAÇÃO] Qual necessidade original levou à escolha do Express?
- [NECESSITA DE INFORMAÇÃO] Quais alternativas foram efetivamente avaliadas e quais critérios foram usados?

## Opções Consideradas

- **Express como framework principal:** opção refletida na arquitetura atual e no manifesto do projeto.
- Nenhuma alternativa específica consta nas evidências consultadas; não se presume que outras opções tenham sido comparadas.

## Resultado da Decisão

Propõe-se registrar o Express como framework principal da aplicação HTTP, conforme o uso observado. O status de proposta não afirma que houve aprovação formal ou que a motivação histórica seja conhecida.

## Prós e Contras das Opções

- **Express como framework principal**
  - **Prós:** fornece o modelo usado atualmente para compor middleware e roteamento dos cinco domínios; a composição da aplicação e a inicialização do servidor permanecem separadas.
  - **Contras:** a composição HTTP fica vinculada ao modelo de aplicação, middleware e roteamento do Express. Uma substituição exigiria adaptar a inicialização e a integração dessas responsabilidades.

## Consequências

- As decisões futuras sobre middleware e roteamento da aplicação HTTP permanecem vinculadas ao Express enquanto ele for o framework principal.
- Uma eventual substituição exigiria adaptar a inicialização e a integração das rotas e dos middlewares; a extensão do trabalho não está determinada pelas evidências.
- [NECESSITA DE INFORMAÇÃO] Quais condições e consequências de longo prazo devem orientar uma eventual substituição?
- [NECESSITA DE INFORMAÇÃO] Qual é a extensão real do esforço de migração, caso uma substituição seja considerada?

## Referências

- `package.json` — declara Express como dependência de execução.
- `src/app.ts` — evidencia a composição da aplicação HTTP com Express.
- `src/server.ts` — evidencia a inicialização do servidor a partir da aplicação.
- `src/routes/index.ts` — evidencia a agregação dos routers dos domínios.
- `docs/adrs/mapping.md` — descreve o escopo do módulo API e sua composição HTTP.
