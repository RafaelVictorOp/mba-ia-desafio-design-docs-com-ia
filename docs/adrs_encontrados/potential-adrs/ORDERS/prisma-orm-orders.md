# Potencial ADR: Prisma como ORM no módulo ORDERS

**Módulo:** ORDERS
**Categoria:** Tecnologia
**Prioridade:** Documentação obrigatória (pontuação: 115)
**Data da identificação:** 2026-11-08

---

## Contexto de ADRs existentes

Não foram encontrados ADRs existentes em `docs/adrs/generated/`; não há decisão anterior disponível para comparação lexical, análise de duplicidade ou cronologia.

## O que foi identificado

O módulo ORDERS utiliza Prisma como camada de persistência. `OrderRepository` recebe um `PrismaClient` e consulta os modelos por meio da API tipada do Prisma; `OrderService` também depende do cliente e utiliza `Prisma.TransactionClient` para operações transacionais. A evidência confirma a tecnologia em uso, mas não registra por que ela foi escolhida nem se alternativas foram avaliadas.

O código emprega transações Prisma para criação do pedido e para alterações de status que também atualizam histórico e estoque. Isso torna o acoplamento com os tipos e mecanismos de transação do ORM visível no módulo. O histórico Git mostra esses arquivos no commit inicial de 2026-06-24 (`init repository`), sem commits posteriores no módulo que expliquem a escolha.

Este candidato deriva da Etapa 0 (ORM/camada de acesso a dados) e, portanto, não passa pelos sinais de exclusão. A pontuação considera apenas a evidência e o escopo do módulo ORDERS; não representa uma aprovação de ADR formal.

## Por que essa decisão pode merecer um ADR

- **Impacto:** afeta os repositórios e os serviços de persistência examinados em ORDERS; impacto em outros módulos não foi avaliado nesta análise.
- **Trade-offs:** o código aproveita consultas e transações tipadas do Prisma, mas usa tipos e APIs específicos do ORM, o que cria acoplamento caso a camada de dados seja substituída.
- **Complexidade:** consultas e transações do módulo estão expressas diretamente na API Prisma.
- **Conhecimento da equipe:** entender o cliente, os tipos de transação e as operações Prisma é importante para evoluir a persistência de ORDERS.
- **Implicações futuras:** uma troca do ORM exigiria rever o acesso a dados e os limites transacionais aqui observados; o impacto fora de ORDERS não foi estimado.
- **Estabilidade observável:** o histórico contém apenas o commit inicial do módulo, em 2026-06-24; esse período, por si só, não comprova intenção de longo prazo.

### Pontuação

- Base Etapa 0 (ORM): 75 pontos.
- Abrangência e impacto: 10 pontos — um módulo, conforme o escopo delimitado.
- Custo da mudança: 15 pontos — estimativa analítica de duas a oito semanas para substituir o ORM e adaptar a persistência do módulo; não é uma duração comprovada pelo Git.
- Conhecimento da equipe: 15 pontos — conhecimento importante para a área de persistência do módulo.
- **Total: 115/150 — `must-document/`.**

## Evidências encontradas no código

### Arquivos principais

- [`src/modules/orders/order.repository.ts`](../../../../../src/modules/orders/order.repository.ts#L1-L19) — importa os tipos do Prisma e injeta `PrismaClient` no repositório.
- [`src/modules/orders/order.service.ts`](../../../../../src/modules/orders/order.service.ts#L1-L6) — importa `Prisma` e `PrismaClient`; o serviço usa a transação do ORM.
- [`src/modules/orders/order.service.ts`](../../../../../src/modules/orders/order.service.ts#L58-L123) — cria o pedido com uma transação Prisma.
- [`src/modules/orders/order.service.ts`](../../../../../src/modules/orders/order.service.ts#L131-L177) — altera status, histórico e efeitos associados dentro de uma transação Prisma.

### Evidências de código

```ts
// src/modules/orders/order.repository.ts:33-44
async list(filters: OrderListFilters): Promise<{ items: Order[]; total: number }> {
  const where = this.buildWhere(filters);
  const [items, total] = await this.prisma.$transaction([
    this.prisma.order.findMany({
      where,
      orderBy: { createdAt: 'desc' },
      skip: filters.skip,
      take: filters.take,
    }),
    this.prisma.order.count({ where }),
  ]);
  return { items, total };
}
});
```

```ts
// src/modules/orders/order.repository.ts:18-19
export class OrderRepository {
  constructor(private readonly prisma: PrismaClient) {}
}
```

### Análise de impacto

- Introdução: 2026-06-24, no commit `init repository`.
- Modificações: 1 commit no histórico disponível para `src/modules/orders`; não há commits subsequentes registrados.
- Última alteração: 2026-06-24, no mesmo commit inicial; o assunto do commit não documenta motivação técnica.
- Abrangência: arquivos de persistência e serviço do módulo ORDERS; impacto fora desse módulo não foi analisado.
- Temas recentes: indisponíveis; não há commits posteriores ao inicial para esse módulo.

### Alternativas (se observáveis)

Nenhuma alternativa de ORM foi encontrada nos arquivos examinados ou no assunto do commit. Não é possível concluir que outras opções tenham sido consideradas.

## Questões para abordar no ADR formal (se criado)

- Qual problema motivou a adoção do Prisma?
- Quais alternativas de persistência foram avaliadas?
- Quais benefícios e custos do uso de tipos e transações específicos do Prisma são aceitos?
- Quais módulos dependem da mesma escolha fora do escopo desta análise?

## Potenciais ADRs relacionados

- [Express como framework HTTP no módulo ORDERS](./express-framework-orders.md)
- [API HTTP em estilo REST no módulo ORDERS](./api-rest-orders.md)

## Observações adicionais

Esta análise identifica tecnologia observada no código, não uma decisão formal aprovada nem a intenção de seus autores. O schema de banco e a configuração de infraestrutura não foram analisados porque o escopo solicitado se limita ao módulo ORDERS.
