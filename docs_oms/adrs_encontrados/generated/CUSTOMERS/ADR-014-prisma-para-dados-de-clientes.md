# ADR-014: Prisma para dados de clientes

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-005](../PRODUCTS/ADR-005-prisma-como-camada-de-dados-de-produtos.md), [ADR-007](../ORDERS/ADR-007-prisma-como-orm-em-pedidos.md), [ADR-017](../DATA/ADR-017-prisma-como-orm.md)

## Contexto e Declaração do Problema

O módulo CUSTOMERS utiliza Prisma como camada de acesso a dados: as operações de persistência de clientes e os tipos associados dependem do Prisma Client. O schema declara a geração do cliente Prisma, e o projeto declara as dependências correspondentes na versão 5.22.0.

O uso está comprovado, mas os documentos examinados não registram os requisitos que motivaram a escolha nem uma aprovação formal. Este ADR registra a escolha observada como proposta, sem atribuir uma justificativa histórica não documentada. A seleção do banco de dados não faz parte desta decisão.

## Fatores de Decisão

- A camada atual de persistência do módulo CUSTOMERS é baseada em Prisma.
- A integração observada utiliza tipos e operações próprios do Prisma.
- Uma substituição exigiria adaptar o acesso a dados e os tipos relacionados.
- Não há evidência dos critérios que motivaram a adoção ou de comparação entre alternativas.

## Opções Consideradas

- **Prisma como camada de acesso a dados:** opção em uso no módulo e proposta para registro nesta decisão.
- **Alternativas:** não foram identificadas alternativas efetivamente consideradas nas evidências disponíveis; não é possível compará-las sem inventar critérios ou resultados.

## Resultado da Decisão

Registra-se, como proposta, manter o Prisma como camada de acesso a dados do módulo CUSTOMERS, refletindo o uso atual. O status permanece Proposto porque as evidências não demonstram uma aprovação formal nem esclarecem os motivos originais da escolha.

## Prós e Contras das Opções

**Prisma como camada de acesso a dados**

- **Prós:** mantém a abordagem de persistência já observada no módulo e seu uso de tipos associados ao cliente Prisma.
- **Contras:** vincula o acesso e os tipos do módulo a APIs específicas do Prisma; uma eventual substituição requer adaptação desses elementos.

**Alternativas não documentadas**

- Não há evidências para atribuir prós, contras ou critérios comparativos a opções específicas.

## Consequências

- A manutenção da abordagem proposta depende de conhecimento do Prisma para evoluir a persistência de clientes.
- Uma eventual troca de camada exige reavaliar o acesso a dados e os tipos relacionados.
- A decisão limita-se ao módulo CUSTOMERS e não decide a tecnologia do banco nem consolida a escolha de ORM para outros módulos.
- [NECESSITA DE INFORMAÇÃO] Quais requisitos motivaram a adoção do Prisma?
- [NECESSITA DE INFORMAÇÃO] Quais alternativas e critérios foram avaliados?
- [NECESSITA DE INFORMAÇÃO] Como serão tratadas consultas específicas do banco e a evolução do schema?
- [NECESSITA DE INFORMAÇÃO] Quem deve ratificar a proposta e sua relação com a escolha compartilhada nos demais módulos?

## Referências

1. `docs/adrs/potential-adrs/done/CUSTOMERS/camada-de-acesso-a-dados-prisma.md` — análise de origem e evidências do módulo.
2. `src/modules/customers/customer.repository.ts` — evidência do uso de Prisma no acesso a dados de clientes.
3. `prisma/schema.prisma` — configuração do cliente Prisma e modelo de clientes.
4. `package.json` — dependências Prisma declaradas pelo projeto.
5. `src/app.ts` — composição do módulo CUSTOMERS com o cliente Prisma.
