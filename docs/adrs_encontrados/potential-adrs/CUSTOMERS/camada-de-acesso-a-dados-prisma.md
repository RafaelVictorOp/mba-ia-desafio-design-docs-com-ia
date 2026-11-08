# Potencial ADR: Usar Prisma como camada de acesso a dados de clientes

**Módulo:** CUSTOMERS  
**Categoria:** Tecnologia  
**Prioridade:** Documentação obrigatória (pontuação: 100)  
**Data da identificação:** 2026-11-08

---

## O que foi identificado

O repositório do módulo CUSTOMERS recebe `PrismaClient`, usa tipos `Prisma` e `Customer` gerados pelo cliente Prisma e executa operações por meio de `prisma.customer`. `prisma/schema.prisma` define o gerador `prisma-client-js`, e `package.json` declara `@prisma/client` 5.22.0 e `prisma` 5.22.0.

Esta é uma decisão estrutural da Categoria 3 da Etapa 0 (ORM ou camada de acesso a dados). O código demonstra o uso de Prisma, mas não documenta o motivo da escolha ou as alternativas consideradas. O datasource configurado utiliza MySQL; a seleção do banco é tratada como candidato separado.

O histórico Git disponível registra um commit de inclusão (`init repository`, 2026-06-24) nos arquivos consultados, sem alterações posteriores localizadas. Esse registro não permite inferir a motivação nem confirmar uma decisão formal.

## Por que essa decisão pode merecer um ADR

- **Impacto:** operações de leitura, contagem, criação, atualização e exclusão de clientes passam pelo Prisma Client.
- **Trade-offs:** o módulo utiliza tipos e operações específicos de Prisma; nenhuma comparação com outra camada de acesso foi encontrada.
- **Complexidade:** trocar a camada exigiria adaptar o repositório e os tipos relacionados; a pontuação de custo usa o mínimo previsto para decisões estruturais.
- **Conhecimento da equipe:** compreender o cliente, as consultas e os tipos Prisma é necessário para manter persistência de clientes.
- **Implicações futuras:** o modelo de dados e a implementação de persistência do módulo dependem do ORM adotado.

**Pontuação:** base estrutural 75 + abrangência 10 + custo de mudança 10 + conhecimento da equipe 5 = **100/150**. Aplicam-se os pisos da categoria estrutural universal; as parcelas não representam estimativa de esforço medida.

## Evidências encontradas no código

### Arquivos principais

- [`src/modules/customers/customer.repository.ts`](../../../../../src/modules/customers/customer.repository.ts) — linhas 1-3 e 25-55: tipos e operações Prisma para clientes.
- [`prisma/schema.prisma`](../../../../../prisma/schema.prisma) — linhas 1-3 e 40-53: gerador Prisma Client e modelo `Customer`.
- [`package.json`](../../../../../package.json) — linhas 28 e 48: dependências `@prisma/client` e `prisma`.
- [`src/app.ts`](../../../../../src/app.ts) — linhas 22-24 e 34-36: injeção do `PrismaClient` na composição de CUSTOMERS.

### Evidências de código

```typescript
import type { Customer, Prisma, PrismaClient } from '@prisma/client';

export class CustomerRepository {
  constructor(private readonly prisma: PrismaClient) {}
```

Trecho de `src/modules/customers/customer.repository.ts:1-6`.

### Análise de impacto

- **Introdução observada no histórico:** os arquivos consultados aparecem no commit `init repository`, em 2026-06-24; isso não estabelece quando ou por que o Prisma foi escolhido.
- **Modificações:** 1 commit de inclusão encontrado nos arquivos consultados; nenhuma modificação posterior localizada.
- **Última alteração localizada:** 2026-06-24, commit `init repository`.
- **Abrangência:** repositório e composição do módulo CUSTOMERS, com tipos fornecidos pelo Prisma Client.
- **Temas recentes:** não há commits posteriores associados a migração de ORM nos arquivos consultados.

### Alternativas (se observáveis)

Nenhuma alternativa de ORM ou justificativa comparativa foi encontrada nos arquivos e commits examinados.

## Questões para abordar no ADR formal (se criado)

- Quais requisitos motivaram o uso do Prisma?
- Quais alternativas foram consideradas para acesso e mapeamento de dados?
- Como a equipe pretende tratar consultas específicas do banco e evolução do schema?

## Potenciais ADRs relacionados

- A escolha do Prisma também aparece em análises de candidatos nos módulos [DATA](../DATA/orm-prisma.md), [PRODUCTS](../PRODUCTS/prisma-como-camada-de-acesso-a-dados.md) e [ORDERS](../ORDERS/prisma-orm-orders.md). São análises de candidatos, não ADRs formais; avaliar consolidar uma decisão compartilhada antes de formalizar.
- [Persistir os dados de clientes em MySQL](./banco-de-dados-mysql.md)

## Observações adicionais

A evidência valida o uso do Prisma no módulo, não que todas as operações de dados da aplicação tenham sido examinadas nesta análise.
