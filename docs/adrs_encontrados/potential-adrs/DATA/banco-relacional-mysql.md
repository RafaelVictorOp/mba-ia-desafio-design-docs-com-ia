# Potencial ADR: Uso do MySQL como banco relacional da aplicação

**Módulo:** DATA  
**Categoria:** Tecnologia (infraestrutura de dados)  
**Prioridade:** Documentação obrigatória (pontuação: 135)  
**Data da identificação:** 2026-11-08

---

## Contexto de ADRs existentes

Não há ADRs formais na pasta `docs/adrs/generated/`. O `PRD` e o `RFC` existentes mencionam o MySQL em propostas de outbox para webhooks, mas são documentos de requisitos/design em rascunho e não formalizam a decisão estrutural do banco de dados do sistema.

## O que foi identificado

O módulo DATA utiliza MySQL como banco relacional: o serviço `mysql` no Docker Compose usa a imagem `mysql:8.0`, e o datasource do Prisma declara `provider = "mysql"`. O Compose também configura persistência em volume local, health check e opções de charset/collation. A migration inicial cria tabelas relacionais para as entidades do sistema.

O histórico Git disponível mostra os arquivos de infraestrutura e persistência no commit inicial `init repository`, datado de 2026-06-24. Não há commits posteriores nesses arquivos consultados nem mensagem que explique a motivação da escolha. A data do commit inicial não comprova quando ou por que a decisão foi tomada.

## Por que essa decisão pode merecer um ADR

- **Impacto:** o mapeamento identifica DATA como dependência de todos os módulos de domínio; o schema e as migrations definem o armazenamento persistente compartilhado.
- **Trade-offs:** a configuração observada acopla o datasource ao provider MySQL e às capacidades/características desse banco; alternativas não estão registradas no código ou no histórico consultado.
- **Complexidade:** uma eventual substituição exige compatibilizar o schema, migrations, configurações de conexão e acesso aos dados. O esforço é uma estimativa, não uma medição documentada.
- **Conhecimento da equipe:** entender o banco, o schema e a configuração é relevante para trabalho que cruza os módulos dependentes de DATA.
- **Implicações futuras:** mudanças de versão, hospedagem ou tecnologia de banco devem preservar compatibilidade com migrations e dados existentes. Não há evidência suficiente para afirmar requisitos de escala ou disponibilidade específicos.

### Pontuação

- Base da Etapa 0 — serviço de infraestrutura: **75**
- Abrangência e impacto — infraestrutura central usada pelos módulos de domínio: **20**
- Custo de mudança — estimativa de duas a oito semanas para uma troca de banco envolvendo schema e persistência: **15**
- Conhecimento da equipe — importante para a maioria das funcionalidades dependentes de persistência: **20**
- **Total: 130/150 — must-document**

Os pontos de custo e conhecimento são estimativas de análise, não fatos medidos no repositório. A categoria estrutural da Etapa 0 não passa pelos sinais de exclusão.

## Evidências encontradas no código

### Arquivos principais

- [`docker-compose.yml`](../../../../../docker-compose.yml) — linhas 1–3 e 11–14
  - Declara um serviço independente MySQL 8.0, expõe a porta 3306 e monta volume para os dados.
- [`prisma/schema.prisma`](../../../../../prisma/schema.prisma) — linhas 5–9
  - Declara MySQL como provider do datasource e lê as URLs de conexão do ambiente.
- [`prisma/migrations/20260519182739_init/migration.sql`](../../../../../prisma/migrations/20260519182739_init/migration.sql) — linhas 1–13
  - Demonstra que a migration inicial cria tabelas MySQL com tipos e collation próprios do banco.

### Evidências de código

```yaml
# docker-compose.yml:1-3
services:
  mysql:
    image: mysql:8.0
```

```prisma
// prisma/schema.prisma:5-9
datasource db {
  provider          = "mysql"
  url               = env("DATABASE_URL")
  shadowDatabaseUrl = env("SHADOW_DATABASE_URL")
}
```

### Análise de impacto

- **Introdução:** os arquivos relevantes estão presentes no commit inicial de 2026-06-24; o histórico não estabelece quando a decisão foi efetivamente tomada.
- **Modificações:** 1 commit observado nos arquivos de infraestrutura/schema/migration consultados.
- **Última alteração:** 2026-06-24, commit `init repository`.
- **Abrangência:** 1 módulo analisado (DATA), com dependência do banco indicada pelo mapeamento para todos os módulos de domínio; 3 arquivos principais citados.
- **Temas recentes:** não há commits posteriores nem temas de alteração documentados. A mensagem disponível é apenas `init repository`.

### Alternativas (se observáveis)

Nenhuma alternativa de banco relacional foi explicitada nos arquivos ou mensagens de commit consultados.

## Questões para abordar no ADR formal (se criado)

- Qual necessidade e quais critérios levaram à escolha de MySQL?
- Quais alternativas de banco foram avaliadas?
- Quais requisitos operacionais, de hospedagem e de compatibilidade devem orientar upgrades ou migrações?
- Que estratégia deve preservar os dados e reduzir riscos em uma eventual troca?

## Potenciais ADRs relacionados

- [Uso do Prisma como ORM e camada de acesso a dados](../orm-prisma.md) — escolha relacionada e complementar; este candidato trata do banco, enquanto o outro trata da camada de acesso.

## Observações adicionais

O código comprova a configuração atual, não a intenção original nem uma comparação entre fornecedores. O componente é de infraestrutura em execução; a existência de propostas de outbox em documentos `Draft` não demonstra que essa feature ou tabelas adicionais estejam implementadas.
