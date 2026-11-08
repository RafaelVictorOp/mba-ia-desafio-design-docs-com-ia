# ADR-017: Prisma como ORM e camada de acesso a dados

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-005](../PRODUCTS/ADR-005-prisma-como-camada-de-dados-de-produtos.md), [ADR-007](../ORDERS/ADR-007-prisma-como-orm-em-pedidos.md), [ADR-014](../CUSTOMERS/ADR-014-prisma-para-dados-de-clientes.md)

## Contexto e Declaração do Problema

O projeto já utiliza Prisma na camada de acesso a dados. O schema e a configuração do projeto confirmam sua integração com o banco MySQL. O PRD e o RFC também descrevem Prisma como tecnologia existente. Essas evidências comprovam o uso atual, mas não explicam a motivação original da escolha.

É necessário registrar se o projeto deve manter Prisma como ORM e camada de acesso a dados, considerando sua integração existente e as consequências de depender dessa tecnologia.

## Fatores de Decisão

- Prisma está integrado à modelagem e ao acesso tipado aos dados existentes.
- Os módulos de domínio dependem da camada DATA, portanto a ferramenta tem abrangência transversal.
- A substituição afetaria a definição do modelo de dados, as migrations e os acessos existentes; o esforço não foi medido.
- [NECESSITA DE INFORMAÇÃO] Registrar os critérios e necessidades que motivaram originalmente a escolha do Prisma.
- [NECESSITA DE INFORMAÇÃO] Validar os benefícios e as limitações do acesso tipado para este projeto.
- [NECESSITA DE INFORMAÇÃO] Comparar acesso SQL direto e outros ORMs com base nos critérios do projeto.
- [NECESSITA DE INFORMAÇÃO] Definir como serão avaliadas atualizações do Prisma, mudanças em migrations e uma eventual substituição.

## Opções Consideradas

1. Manter Prisma como ORM e camada de acesso a dados. É a tecnologia observada atualmente; a justificativa original não está documentada.
2. Usar acesso SQL direto. É uma alternativa a avaliar, mas não há evidência de que tenha sido considerada ou implementada.
3. Adotar outro ORM. Não há registro de ferramentas alternativas avaliadas.

## Resultado da Decisão

Propõe-se manter Prisma como ORM e camada de acesso a dados, em coerência com o uso atual comprovado. Esta proposta não atribui ao projeto uma motivação original não documentada nem substitui a validação dos critérios pendentes.

## Prós e Contras das Opções

| Opção | Prós | Contras |
| --- | --- | --- |
| Manter Prisma | Preserva a integração existente entre schema, migrations e acesso tipado. | Mantém o acoplamento à ferramenta e ao seu modelo; mudanças podem afetar os acessos e a evolução do schema. |
| Acesso SQL direto | Não há benefícios registrados para este projeto. | Não há custos de migração ou manutenção registrados. |
| Outro ORM | Não há ferramentas ou benefícios registrados. | Não há comparação de compatibilidade ou custos de transição registrada. |

## Consequências

- A proposta preserva o uso atual do Prisma e os artefatos de persistência associados.
- Uma futura substituição exigirá avaliar impacto no schema, nas migrations e no acesso aos dados; não há estimativa comprovada de duração.
- Não há política de atualização registrada; essa definição permanece pendente.

## Referências

1. [Candidato de ADR: Uso do Prisma como ORM e camada de acesso a dados](../../potential-adrs/done/DATA/orm-prisma.md) — evidências disponíveis e lacunas identificadas.
2. [Schema Prisma](../../../../prisma/schema.prisma) — configuração atual do schema e do datasource MySQL.
3. [Configuração de acesso a dados](../../../../src/config/database.ts) — evidência da integração atual com Prisma.
4. [Manifesto do projeto](../../../../package.json) — versões fixadas de Prisma e do cliente.
5. [Candidato de ADR: Uso do MySQL como banco relacional](../../potential-adrs/done/DATA/banco-relacional-mysql.md) — decisão relacionada sobre o banco, distinta da escolha do ORM.
