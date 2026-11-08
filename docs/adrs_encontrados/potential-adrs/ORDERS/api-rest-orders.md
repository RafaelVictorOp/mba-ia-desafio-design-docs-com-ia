# Potencial ADR: API HTTP em estilo REST no módulo ORDERS

**Módulo:** ORDERS
**Categoria:** Arquitetura
**Prioridade:** Documentação obrigatória (pontuação: 115)
**Data da identificação:** 2026-11-08

---

## Contexto de ADRs existentes

Não foram encontrados ADRs existentes em `docs/adrs/generated/`; não há decisão anterior disponível para comparação lexical, análise de duplicidade ou cronologia.

## O que foi identificado

O módulo expõe operações HTTP por rotas que usam recursos de pedidos e métodos `GET`, `POST`, `PATCH` e `DELETE`. Os handlers retornam códigos HTTP como `200`, `201` e `204`. Esse conjunto é compatível com uma API HTTP em estilo REST para as operações observadas; a evidência não basta para afirmar conformidade completa com todas as restrições de REST.

O histórico Git mostra as rotas e o controller no commit inicial de 2026-06-24 (`init repository`), sem commits posteriores no módulo que registrem motivação, migração ou avaliação de estilos de API alternativos. O assunto do commit não explicita a justificativa.

Este candidato deriva da Etapa 0 (protocolo/arquitetura de API) e, portanto, não passa pelos sinais de exclusão. A análise está limitada às rotas HTTP de ORDERS e não descreve a API completa do sistema.

## Por que essa decisão pode merecer um ADR

- **Impacto:** define o estilo visível nos contratos HTTP de ORDERS; impacto em outros módulos não foi avaliado nesta análise.
- **Trade-offs:** o uso de métodos e recursos HTTP oferece uma interface direta para as operações observadas, mas mantém os consumidores vinculados a esses endpoints e semânticas de status.
- **Complexidade:** alterar o estilo de API exigiria rever as rotas e os contratos HTTP deste módulo; o impacto em consumidores externos não foi analisado.
- **Conhecimento da equipe:** conhecer os métodos, rotas e códigos HTTP em uso é importante para evoluir os endpoints de ORDERS.
- **Implicações futuras:** clientes e integrações que dependam dessas rotas podem ser afetados por mudanças no contrato; não há inventário de consumidores no escopo examinado.
- **Estabilidade observável:** o histórico contém apenas o commit inicial do módulo, em 2026-06-24; esse período não comprova intenção de longo prazo.

### Pontuação

- Base Etapa 0 (protocolo/arquitetura de API): 75 pontos.
- Abrangência e impacto: 10 pontos — um módulo, conforme o escopo delimitado.
- Custo da mudança: 15 pontos — estimativa analítica de duas a oito semanas para rever os endpoints e contratos de ORDERS; o impacto em consumidores não foi avaliado.
- Conhecimento da equipe: 15 pontos — conhecimento importante para a área de interface HTTP do módulo.
- **Total: 115/150 — `must-document/`.**

## Evidências encontradas no código

### Arquivos principais

- [`src/modules/orders/order.routes.ts`](../../../../../src/modules/orders/order.routes.ts#L12-L24) — associa recursos a métodos HTTP e handlers.
- [`src/modules/orders/order.controller.ts`](../../../../../src/modules/orders/order.controller.ts#L9-L55) — produz respostas HTTP para as operações roteadas.

### Evidências de código

```ts
// src/modules/orders/order.routes.ts:16-24
router.get('/', validate({ query: listOrdersQuerySchema }), controller.list);
router.get('/:id', validate({ params: orderIdParamSchema }), controller.getById);
router.post('/', validate({ body: createOrderSchema }), controller.create);
router.patch(
  '/:id/status',
  validate({ params: orderIdParamSchema, body: updateOrderStatusSchema }),
  controller.changeStatus,
);
router.delete('/:id', validate({ params: orderIdParamSchema }), controller.delete);
```

```ts
// src/modules/orders/order.controller.ts:28-32
create: RequestHandler = async (req, res, next) => {
  try {
    if (!req.user) throw new UnauthorizedError();
    const created = await this.orders.create(req.body, req.user.id);
    res.status(201).json(created);
```

### Análise de impacto

- Introdução: 2026-06-24, no commit `init repository`.
- Modificações: 1 commit no histórico disponível para `src/modules/orders`; não há commits subsequentes registrados.
- Última alteração: 2026-06-24, no mesmo commit inicial; o assunto não documenta motivação técnica.
- Abrangência: rotas e controller de ORDERS; consumidores externos e outros módulos não foram analisados.
- Temas recentes: indisponíveis; não há commits posteriores ao inicial para esse módulo.

### Alternativas (se observáveis)

Nenhum estilo alternativo de API foi encontrado nos arquivos examinados ou no assunto do commit. Não é possível concluir que outras opções tenham sido consideradas.

## Questões para abordar no ADR formal (se criado)

- Quais requisitos levaram ao estilo de API HTTP observado?
- Quais alternativas de protocolo ou organização de API foram consideradas?
- Quais consumidores dependem dos endpoints atuais e quais compromissos de compatibilidade existem?
- O estilo é uma decisão comum a outros módulos?

## Potenciais ADRs relacionados

- [Express como framework HTTP no módulo ORDERS](./express-framework-orders.md)
- [Prisma como ORM no módulo ORDERS](./prisma-orm-orders.md)

## Observações adicionais

“Estilo REST” é uma caracterização limitada às evidências de rotas e métodos observados; não é uma declaração de conformidade completa com REST. Esta análise não incluiu consumidores externos nem endpoints de outros módulos.
