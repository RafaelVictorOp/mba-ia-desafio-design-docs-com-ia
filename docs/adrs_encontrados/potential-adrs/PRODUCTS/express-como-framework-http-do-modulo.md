# Potencial ADR: Uso do Express como framework HTTP no módulo PRODUCTS

**Módulo:** PRODUCTS  
**Categoria:** Tecnologia  
**Prioridade:** Documentação obrigatória (pontuação: 100)  
**Data da identificação:** 2026-11-08

---

## Contexto de ADRs existentes

Não foram encontrados ADRs formais no diretório padrão `docs/adrs/generated/`; não há decisão existente para comparar.

## O que foi identificado

O módulo PRODUCTS usa componentes do Express: `product.routes.ts` cria e retorna um `Router`, enquanto `product.controller.ts` tipa seus handlers com `RequestHandler`. As rotas são montadas usando esse mecanismo HTTP e delegam as operações ao controller.

Isso identifica o Express como framework HTTP usado pelo módulo, uma escolha estrutural da aplicação. O código comprova o uso, mas não registra o motivo da seleção nem as alternativas consideradas. Como a evidência está limitada a PRODUCTS, a eventual formalização deve ser coordenada com os demais módulos que compartilham o framework.

## Por que essa decisão pode merecer um ADR

- **Impacto:** determina como as rotas e handlers HTTP do módulo são construídos e integrados.
- **Trade-offs:** o código mostra dependência dos tipos e do roteador do Express; alternativas não são observáveis.
- **Complexidade:** alterar o framework envolveria adaptar rotas e handlers, mas o esforço não pode ser quantificado com evidência disponível.
- **Conhecimento da equipe:** entender o padrão Express usado pelos handlers é necessário para manter o módulo.
- **Implicações futuras:** decisões de middleware, roteamento e integração HTTP devem permanecer compatíveis com o framework ou considerar sua substituição.

### Pontuação

Categoria estrutural universal 2 (framework principal): base 75. Aplicados os mínimos especiais de abrangência (10), custo de mudança (10) e conhecimento da equipe (5), totalizando **100/150** e prioridade `must-document/`. A pontuação é o mínimo garantido pela regra; não representa uma estimativa independente do esforço.

## Evidências encontradas no código

### Arquivos principais

- [`src/modules/products/product.routes.ts`](../../../../../src/modules/products/product.routes.ts), linhas 1 e 12–25
  - Importa `Router` do Express e instancia um roteador para as rotas do módulo.
- [`src/modules/products/product.controller.ts`](../../../../../src/modules/products/product.controller.ts), linhas 1 e 5–6
  - Importa `RequestHandler` do Express e recebe o serviço do módulo por injeção no controller.

### Evidências de código

```typescript
import { Router } from 'express';

export function buildProductRouter(controller: ProductController): Router {
  const router = Router();
  router.use(authenticate);
  router.get('/', validate({ query: listProductsQuerySchema }), controller.list);
```

### Análise de impacto

- Introdução: os arquivos consultados aparecem no histórico em 2026-06-24, no commit `init repository`; isso não comprova quando ou por que a decisão foi tomada.
- Modificações: 1 commit registrado para cada arquivo principal consultado; o histórico disponível não permite inferir evolução arquitetural.
- Última alteração: 2026-06-24, commit `init repository`; sem tema adicional documentado.
- Abrangência: evidência direta em 2 arquivos do módulo PRODUCTS; não foi analisado o código de outros módulos.
- Temas recentes: nenhum além da inclusão inicial dos arquivos.

### Alternativas

Não foram observadas alternativas de framework em comentários, configuração ou commits consultados.

## Questões para abordar no ADR formal (se criado)

- Qual necessidade levou à escolha do Express?
- Quais alternativas foram consideradas?
- Quais consequências e custos de migração devem ser registrados para a aplicação como um todo?

## Potenciais ADRs relacionados

- [API REST para o catálogo de produtos](./api-rest-para-catalogo-produtos.md)
- [Prisma como camada de acesso a dados](./prisma-como-camada-de-acesso-a-dados.md)

## Observações adicionais

A evidência confirma o uso do Express em PRODUCTS, mas a escolha é compartilhada em potencial com a API e outros módulos. Validar o escopo global antes de formalizar, para evitar ADR duplicado por módulo.
