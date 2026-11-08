# Potencial ADR: Express como framework HTTP no módulo ORDERS

**Módulo:** ORDERS
**Categoria:** Arquitetura
**Prioridade:** Documentação obrigatória (pontuação: 115)
**Data da identificação:** 2026-11-08

---

## Contexto de ADRs existentes

Não foram encontrados ADRs existentes em `docs/adrs/generated/`; não há decisão anterior disponível para comparação lexical, análise de duplicidade ou cronologia.

## O que foi identificado

O módulo ORDERS estrutura o transporte HTTP com Express: `order.routes.ts` cria e retorna um `Router` do Express, e `order.controller.ts` define handlers com `RequestHandler` e a interface de requisição/resposta do framework. Isso demonstra o uso de Express na camada HTTP do módulo, mas não comprova por que foi escolhido ou se outras plataformas foram consideradas.

O histórico Git mostra os arquivos de rotas e controller no commit inicial de 2026-06-24 (`init repository`), sem commits posteriores no módulo que registrem motivação, migração ou avaliação de alternativas. O assunto do commit não fornece justificativa técnica.

Este candidato deriva da Etapa 0 (framework/plataforma principal) e, portanto, não passa pelos sinais de exclusão. A pontuação considera somente o módulo ORDERS, sem extrapolar a análise para a aplicação inteira.

## Por que essa decisão pode merecer um ADR

- **Impacto:** o framework está presente na camada HTTP examinada de ORDERS; impacto em outros módulos não foi avaliado nesta análise.
- **Trade-offs:** o código aproveita roteamento e tipos HTTP do Express, mas as rotas e os handlers dependem das APIs do framework.
- **Complexidade:** substituir o framework exigiria adaptar a construção do roteador e a interface dos handlers do módulo; o escopo fora de ORDERS não foi estimado.
- **Conhecimento da equipe:** familiaridade com roteadores e handlers Express é importante para modificar a camada HTTP de ORDERS.
- **Implicações futuras:** decisões de middleware, roteamento e tratamento de requisições neste módulo permanecem ligadas ao framework observado.
- **Estabilidade observável:** o histórico contém apenas o commit inicial do módulo, em 2026-06-24; esse período não prova intenção de longo prazo.

### Pontuação

- Base Etapa 0 (framework/plataforma principal): 75 pontos.
- Abrangência e impacto: 10 pontos — um módulo, conforme o escopo delimitado.
- Custo da mudança: 15 pontos — estimativa analítica de duas a oito semanas para adaptar a camada HTTP deste módulo a outro framework; não é uma duração comprovada pelo Git.
- Conhecimento da equipe: 15 pontos — conhecimento importante para a área HTTP do módulo.
- **Total: 115/150 — `must-document/`.**

## Evidências encontradas no código

### Arquivos principais

- [`src/modules/orders/order.routes.ts`](../../../../../src/modules/orders/order.routes.ts#L1-L26) — importa `Router` do Express e declara as rotas do módulo.
- [`src/modules/orders/order.controller.ts`](../../../../../src/modules/orders/order.controller.ts#L1-L17) — usa `RequestHandler` e a interface HTTP do Express nos handlers.

### Evidências de código

```ts
// src/modules/orders/order.routes.ts:12-18
export function buildOrderRouter(controller: OrderController): Router {
  const router = Router();
  router.use(authenticate);
  router.get('/', validate({ query: listOrdersQuerySchema }), controller.list);
  router.get('/:id', validate({ params: orderIdParamSchema }), controller.getById);
  router.post('/', validate({ body: createOrderSchema }), controller.create);
```

```ts
// src/modules/orders/order.controller.ts:1,9
import type { RequestHandler } from 'express';

export class OrderController {
  list: RequestHandler = async (req, res, next) => {
```

### Análise de impacto

- Introdução: 2026-06-24, no commit `init repository`.
- Modificações: 1 commit no histórico disponível para `src/modules/orders`; não há commits subsequentes registrados.
- Última alteração: 2026-06-24, no mesmo commit inicial; o assunto não documenta motivação técnica.
- Abrangência: rotas e controller de ORDERS; não foram analisados outros módulos.
- Temas recentes: indisponíveis; não há commits posteriores ao inicial para esse módulo.

### Alternativas (se observáveis)

Nenhuma alternativa de framework foi encontrada nos arquivos examinados ou no assunto do commit. Não é possível concluir que outras opções tenham sido consideradas.

## Questões para abordar no ADR formal (se criado)

- Quais requisitos levaram ao uso de Express na camada HTTP?
- Quais alternativas foram avaliadas?
- Quais são os custos de manter os handlers e rotas acoplados ao Express?
- A escolha é compartilhada com outros módulos da aplicação?

## Potenciais ADRs relacionados

- [Prisma como ORM no módulo ORDERS](./prisma-orm-orders.md)
- [API HTTP em estilo REST no módulo ORDERS](./api-rest-orders.md)

## Observações adicionais

Esta análise identifica tecnologia observada no código, não uma decisão formal aprovada nem a intenção de seus autores. A inicialização da aplicação e a composição de rotas fora de ORDERS não foram analisadas.
