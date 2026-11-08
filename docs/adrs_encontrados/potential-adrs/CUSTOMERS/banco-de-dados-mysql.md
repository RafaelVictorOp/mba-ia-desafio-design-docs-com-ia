# Potencial ADR: Persistir os dados de clientes em MySQL

**Módulo:** CUSTOMERS  
**Categoria:** Tecnologia  
**Prioridade:** Documentação obrigatória (pontuação: 100)  
**Data da identificação:** 2026-11-08

---

## O que foi identificado

O módulo CUSTOMERS persiste seus registros usando Prisma Client, cujo datasource configurado no projeto aponta para MySQL. A infraestrutura de desenvolvimento declara um serviço MySQL 8.0 em `docker-compose.yml`; o modelo `Customer` integra o schema relacional e é acessado pelo repositório do módulo.

Esta é uma decisão estrutural da Categoria 1 da Etapa 0 (serviço de infraestrutura). O código confirma a tecnologia utilizada, mas não registra a motivação original nem compara MySQL com outras opções. A identificação não implica que o banco ou o módulo tenham sido formalmente aprovados por ADR.

O histórico Git disponível registra um commit de inclusão (`init repository`, 2026-06-24) nos arquivos consultados, sem alterações posteriores localizadas. Isso situa a presença desses arquivos no repositório, mas não comprova a data nem a justificativa da decisão arquitetural.

## Por que essa decisão pode merecer um ADR

- **Impacto:** o repositório de CUSTOMERS depende da persistência relacional compartilhada configurada no projeto.
- **Trade-offs:** a configuração observada centraliza os dados em MySQL e utiliza o schema Prisma; não há alternativa documentada no código examinado.
- **Complexidade:** mudar o serviço de dados escolhido afeta configuração, schema e acesso persistido; o custo numérico foi mantido no mínimo previsto para decisões estruturais.
- **Conhecimento da equipe:** entender a dependência do banco é necessário para modificar e operar o acesso aos clientes.
- **Implicações futuras:** alterações na estratégia de armazenamento podem exigir mudança coordenada do datasource e da camada de persistência.

**Pontuação:** base estrutural 75 + abrangência 10 + custo de mudança 10 + conhecimento da equipe 5 = **100/150**. Aplicam-se os pisos da categoria estrutural universal; as parcelas não representam estimativa de esforço medida.

## Evidências encontradas no código

### Arquivos principais

- [`docker-compose.yml`](../../../../../docker-compose.yml) — linhas 1-3: declara o serviço `mysql` usando a imagem `mysql:8.0`.
- [`prisma/schema.prisma`](../../../../../prisma/schema.prisma) — linhas 5-9 e 40-53: datasource MySQL e modelo `Customer`.
- [`src/modules/customers/customer.repository.ts`](../../../../../src/modules/customers/customer.repository.ts) — linhas 1-3, 25-35 e 38-55: repositório recebe `PrismaClient` e executa operações sobre `prisma.customer`.

### Evidências de código

```prisma
datasource db {
  provider          = "mysql"
  url               = env("DATABASE_URL")
  shadowDatabaseUrl = env("SHADOW_DATABASE_URL")
}
```

Trecho de `prisma/schema.prisma:5-9`.

```typescript
const [items, total] = await this.prisma.$transaction([
  this.prisma.customer.findMany({
    where,
    orderBy: { createdAt: 'desc' },
    skip: filters.skip,
    take: filters.take,
  }),
  this.prisma.customer.count({ where }),
]);
```

Trecho de `src/modules/customers/customer.repository.ts:25-35`.

### Análise de impacto

- **Introdução observada no histórico:** os arquivos consultados aparecem no commit `init repository`, em 2026-06-24; isso não estabelece quando ou por que a escolha foi tomada.
- **Modificações:** 1 commit de inclusão encontrado nos arquivos consultados; nenhuma modificação posterior localizada.
- **Última alteração localizada:** 2026-06-24, commit `init repository`.
- **Abrangência:** serviço MySQL declarado na infraestrutura, datasource/schema Prisma e acesso a clientes no módulo CUSTOMERS.
- **Temas recentes:** não há commits posteriores associados a mudança de banco nos arquivos consultados.

### Alternativas (se observáveis)

Nenhuma alternativa de banco ou motivação comparativa foi encontrada nos arquivos e commits examinados.

## Questões para abordar no ADR formal (se criado)

- Qual necessidade motivou MySQL para a persistência do OMS?
- Quais alternativas de banco foram avaliadas e por quais critérios?
- Quais requisitos de operação, disponibilidade e recuperação orientam essa escolha?

## Potenciais ADRs relacionados

- Há uma análise de candidata para a mesma escolha compartilhada no módulo DATA: [Uso do MySQL como banco relacional da aplicação](../DATA/banco-relacional-mysql.md). São análises de candidatos, não ADRs formais; ao formalizar, avaliar consolidar a decisão sistêmica em vez de criar registros redundantes por módulo.
- [Uso de Prisma como camada de acesso a dados](./camada-de-acesso-a-dados-prisma.md)

## Observações adicionais

Análise restrita ao módulo CUSTOMERS e às evidências comuns de infraestrutura/dados necessárias para identificar essa escolha. Não foram analisados outros módulos para pontuar impactos específicos.
