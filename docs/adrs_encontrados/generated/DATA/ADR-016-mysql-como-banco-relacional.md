# ADR-016: MySQL como banco relacional

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-012](../CUSTOMERS/ADR-012-mysql-para-dados-de-clientes.md)

## Contexto e Declaração do Problema

O sistema utiliza MySQL como banco relacional: a configuração de execução declara MySQL 8.0, o schema declara o provider MySQL e a migration inicial define tabelas relacionais. O banco sustenta o armazenamento persistente compartilhado pelos módulos de domínio.

É necessário registrar essa escolha estrutural sem confundir a configuração observada com a motivação histórica, que não está documentada nas evidências examinadas.

## Fatores de Decisão

- A configuração atual e o schema estão vinculados ao provider MySQL.
- O schema e a migration existente definem tabelas e relações persistentes para entidades do sistema.
- A troca de banco exigiria compatibilizar schema, migrations, conexões e acesso aos dados.
- [NECESSITA DE INFORMAÇÃO] A necessidade original e os critérios que levaram à escolha de MySQL não estão registrados.

## Opções Consideradas

- **MySQL:** tecnologia atualmente configurada e utilizada pelo schema e pela migration.
- **Alternativa relacional não especificada:** não há fornecedor alternativo identificado nem evidência de avaliação comparativa.

## Resultado da Decisão

Registrar MySQL como o banco relacional atualmente adotado pelo sistema. Esta decisão formaliza a escolha demonstrada pela configuração e persistência existentes; não afirma qual foi sua justificativa histórica nem que alternativas tenham sido avaliadas.

## Prós e Contras das Opções

- **MySQL**
  - Prós: corresponde à configuração vigente e às tabelas relacionais já definidas no schema e na migration.
  - Contras: schema, migration e configuração de conexão estão vinculados às características e ao provider MySQL; uma substituição exigirá compatibilização desses elementos.
- **Alternativa relacional não especificada**
  - Prós: não há benefício específico documentado.
  - Contras: fornecedor, compatibilidade e impactos de migração não foram avaliados nas evidências disponíveis.

## Consequências

- A manutenção da escolha requer preservar a compatibilidade entre o MySQL, o schema e as migrations existentes.
- Alterações de versão, hospedagem ou tecnologia devem considerar a compatibilidade com os dados e as migrations existentes.
- [NECESSITA DE INFORMAÇÃO] Não estão definidos requisitos operacionais de hospedagem, disponibilidade, escala ou compatibilidade de versões.
- [NECESSITA DE INFORMAÇÃO] Não há estratégia documentada para preservar os dados e reduzir riscos em uma eventual substituição do banco.
- As propostas de outbox para webhooks mencionadas nos documentos de produto e design não demonstram que essa funcionalidade esteja implementada.

## Referências

1. `docker-compose.yml` — serviço MySQL 8.0 e configuração de persistência local.
2. `prisma/schema.prisma` — datasource configurado com provider MySQL.
3. `prisma/migrations/20260519182739_init/migration.sql` — tabelas e relações da migration inicial.
4. `docs/PRD.md` — descreve o MySQL existente como base para a proposta de outbox.
5. `docs/RFC.md` — trata o MySQL como existente e distingue a proposta de webhook da implementação atual.
