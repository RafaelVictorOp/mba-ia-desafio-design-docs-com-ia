# Potencial ADR: API REST para o catálogo de produtos

**Módulo:** PRODUCTS  
**Categoria:** Arquitetura  
**Prioridade:** Documentação obrigatória (pontuação: 100)  
**Data da identificação:** 2026-11-08

---

## Contexto de ADRs existentes

Não foram encontrados ADRs formais no diretório padrão `docs/adrs/generated/`; não há decisão existente para comparar.

## O que foi identificado

O módulo expõe operações de catálogo por rotas HTTP com recursos e métodos distintos: consulta da coleção e de um produto por identificador, criação, atualização parcial e exclusão. A organização em rotas `GET`, `POST`, `PATCH` e `DELETE` é compatível com um estilo de API REST.

A classificação como REST é uma inferência baseada nos métodos e caminhos observados; o código não nomeia explicitamente o estilo nem documenta motivação, alternativas ou requisitos de contrato. A análise está restrita ao módulo PRODUCTS.

## Por que essa decisão pode merecer um ADR

- **Impacto:** define o contrato HTTP do catálogo consumido por clientes da API.
- **Trade-offs:** os handlers seguem operações sobre recursos; alternativas de protocolo ou formato não são registradas no módulo.
- **Complexidade:** mudar o estilo afetaria rotas e clientes, mas o esforço não pode ser estimado a partir das evidências disponíveis.
- **Conhecimento da equipe:** quem integra ou mantém o catálogo precisa entender a convenção dos endpoints.
- **Implicações futuras:** alterações de contrato devem considerar compatibilidade com consumidores existentes.

### Pontuação

Categoria estrutural universal 4 (protocolo ou arquitetura de API): base 75. Aplicados os mínimos especiais de abrangência (10), custo de mudança (10) e conhecimento da equipe (5), totalizando **100/150** e prioridade `must-document/`. A pontuação é o mínimo garantido pela regra; não representa uma estimativa independente do esforço.

## Evidências encontradas no código

### Arquivos principais

- [`src/modules/products/product.routes.ts`](../../../../../src/modules/products/product.routes.ts), linhas 12–25
  - Declara endpoints HTTP para listar, consultar, criar, atualizar parcialmente e excluir produtos.

### Evidências de código

```typescript
router.get('/', validate({ query: listProductsQuerySchema }), controller.list);
router.get('/:id', validate({ params: productIdParamSchema }), controller.getById);
router.post('/', validate({ body: createProductSchema }), controller.create);
router.patch(
  '/:id',
  validate({ params: productIdParamSchema, body: updateProductSchema }),
  controller.update,
);
router.delete('/:id', validate({ params: productIdParamSchema }), controller.delete);
```

### Análise de impacto

- Introdução: `product.routes.ts` aparece no histórico em 2026-06-24, no commit `init repository`; isso não comprova a data definitiva nem a motivação da decisão.
- Modificações: 1 commit registrado para o arquivo; não há histórico de evolução além da inclusão inicial.
- Última alteração: 2026-06-24, commit `init repository`; sem tema adicional documentado.
- Abrangência: evidência direta em 1 arquivo do módulo PRODUCTS; não foi analisado o código de outros módulos ou consumidores.
- Temas recentes: nenhum além da inclusão inicial do arquivo.

### Alternativas

Não foram observadas alternativas de protocolo nos arquivos ou commits consultados.

## Questões para abordar no ADR formal (se criado)

- Quais requisitos levaram ao uso de endpoints HTTP orientados a recursos?
- REST foi comparado com outros protocolos ou estilos?
- Quais garantias de compatibilidade do contrato precisam ser preservadas?

## Potenciais ADRs relacionados

- [Uso do Express como framework HTTP no módulo PRODUCTS](./express-como-framework-http-do-modulo.md)
- [Prisma como camada de acesso a dados](./prisma-como-camada-de-acesso-a-dados.md)

## Observações adicionais

O estilo REST é uma classificação inferida das rotas, não uma decisão expressamente documentada no código. Validar se existe uma decisão de API compartilhada em outro módulo antes de formalizar.
