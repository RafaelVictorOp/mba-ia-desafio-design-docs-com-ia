# ADR-012: MySQL para dados de clientes

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-016](../DATA/ADR-016-mysql-como-banco-relacional.md)

## Contexto e Declaração do Problema

O módulo CUSTOMERS persiste seus registros no banco configurado como MySQL pelo datasource do Prisma. O ambiente de desenvolvimento declara MySQL 8.0, e o modelo de clientes integra o schema relacional e é acessado pelo repositório do módulo.

Essa configuração demonstra a tecnologia em uso, mas não comprova a motivação original nem uma aprovação arquitetural formal. Também não há evidência de comparação com outros bancos. É necessário registrar o estado observado sem atribuir justificativas ou critérios que não foram documentados.

## Fatores de Decisão

- O datasource do projeto está configurado para MySQL.
- O ambiente de desenvolvimento declara um serviço MySQL 8.0.
- Os dados de clientes integram o schema relacional usado pelo módulo.
- A configuração observada vincula a persistência ao banco escolhido; uma mudança pode exigir alterações coordenadas na estratégia de armazenamento.
- [NECESSITA DE INFORMAÇÃO] Qual necessidade de negócio ou técnica motivou a escolha de MySQL?
- [NECESSITA DE INFORMAÇÃO] Quais critérios e bancos alternativos foram efetivamente avaliados?
- [NECESSITA DE INFORMAÇÃO] Quais requisitos de operação, disponibilidade e recuperação devem orientar a escolha?

## Opções Consideradas

1. **MySQL:** opção atualmente configurada e observada para a persistência dos dados de clientes. A documentação disponível não registra uma avaliação comparativa.

Não há evidência de outras opções que tenham sido consideradas. Não se presume que alternativas tenham sido avaliadas ou rejeitadas.

## Resultado da Decisão

Propõe-se registrar MySQL como a tecnologia atualmente usada para persistir os dados de clientes. O status permanece Proposto porque as referências comprovam a configuração existente, mas não demonstram uma decisão formal aprovada nem sua justificativa.

## Prós e Contras das Opções

**MySQL**

- **Prós:** corresponde à configuração existente do datasource e do ambiente de desenvolvimento; os dados de clientes já fazem parte do schema relacional.
- **Contras:** a persistência está vinculada ao provider configurado; uma troca pode demandar mudanças coordenadas no datasource, no schema e no acesso aos dados. O esforço não foi medido.
- **Lacuna:** não há evidência suficiente para comparar benefícios, custos ou requisitos operacionais com outras opções.

## Consequências

- O registro descreve o uso observado e não afirma que a motivação, os critérios de escolha ou a aprovação formal sejam conhecidos.
- Mudanças futuras na estratégia de armazenamento podem exigir revisão coordenada da configuração e da persistência de clientes.
- A análise relacionada do módulo DATA trata MySQL como escolha compartilhada. Esta ADR limita-se aos dados de clientes e não formaliza nem reprocessa aquela candidata; a consolidação de uma decisão sistêmica permanece em aberto.

## Referências

1. Potencial ADR de CUSTOMERS, fonte movida para `docs/adrs/potential-adrs/done/CUSTOMERS/banco-de-dados-mysql.md`.
2. `docker-compose.yml` — serviço MySQL 8.0 declarado para desenvolvimento.
3. `prisma/schema.prisma` — datasource MySQL e modelo relacional de clientes.
4. `src/modules/customers/customer.repository.ts` — acesso do repositório aos dados de clientes.
5. Potencial ADR relacionada de DATA, consultada apenas como referência de escopo compartilhado: `docs/adrs/potential-adrs/done/DATA/banco-relacional-mysql.md`.
