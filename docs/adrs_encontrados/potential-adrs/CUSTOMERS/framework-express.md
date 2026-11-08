# Potencial ADR: Estruturar a API de clientes com Express

**Módulo:** CUSTOMERS  
**Categoria:** Tecnologia  
**Prioridade:** Documentação obrigatória (pontuação: 100)  
**Data da identificação:** 2026-11-08

---

## O que foi identificado

As rotas de CUSTOMERS são construídas com `Router` do Express, e os handlers do controller usam o tipo `RequestHandler`. O bootstrap da aplicação importa e instancia Express, e `package.json` declara a dependência `express` na versão 4.21.1. O framework Express é a estrutura principal observada para a aplicação HTTP.

Esta é uma decisão estrutural da Categoria 2 da Etapa 0 (framework ou plataforma principal). O código comprova o framework utilizado, mas não explica por que foi escolhido nem registra comparação com alternativas. A análise está limitada às evidências do CUSTOMERS e à composição central necessária para confirmar o framework.

O histórico Git disponível registra um commit de inclusão (`init repository`, 2026-06-24) nos arquivos consultados, sem alterações posteriores localizadas. Isso demonstra a presença dos arquivos no histórico consultado, não a motivação ou aprovação formal da decisão.

## Por que essa decisão pode merecer um ADR

- **Impacto:** o Express provê o roteador e os tipos HTTP usados para expor as operações de CUSTOMERS.
- **Trade-offs:** o módulo é integrado ao pipeline HTTP do Express e segue suas interfaces de rotas e handlers; alternativa ou justificativa não aparece no código examinado.
- **Complexidade:** substituir o framework exigiria adaptar a composição HTTP, as rotas e os handlers; a pontuação de custo usa o mínimo previsto para decisões estruturais.
- **Conhecimento da equipe:** compreender o framework é necessário para manter os endpoints e integrar controllers ao pipeline HTTP.
- **Implicações futuras:** decisões sobre middlewares, roteamento e tratamento de requisições do módulo dependem da plataforma HTTP escolhida.

**Pontuação:** base estrutural 75 + abrangência 10 + custo de mudança 10 + conhecimento da equipe 5 = **100/150**. Aplicam-se os pisos da categoria estrutural universal; as parcelas não representam estimativa de esforço medida.

## Evidências encontradas no código

### Arquivos principais

- [`src/modules/customers/customer.routes.ts`](../../../../../src/modules/customers/customer.routes.ts) — linhas 1-3 e 11-25: importa `Router` e registra endpoints no router.
- [`src/modules/customers/customer.controller.ts`](../../../../../src/modules/customers/customer.controller.ts) — linhas 1-8: handlers tipados como `RequestHandler`.
- [`src/app.ts`](../../../../../src/app.ts) — linhas 1 e 56-57: importa e instancia Express.
- [`package.json`](../../../../../package.json) — linha 28: declara `express` na versão `4.21.1`.

### Evidências de código

```typescript
import { Router } from 'express';
import { authenticate } from '../../middlewares/auth.middleware.js';
import { validate } from '../../middlewares/validate.middleware.js';

export function buildCustomerRouter(controller: CustomerController): Router {
  const router = Router();
  router.use(authenticate);
```

Trecho de `src/modules/customers/customer.routes.ts:1-7`.

### Análise de impacto

- **Introdução observada no histórico:** os arquivos consultados aparecem no commit `init repository`, em 2026-06-24; isso não estabelece quando ou por que o framework foi escolhido.
- **Modificações:** 1 commit de inclusão encontrado nos arquivos consultados; nenhuma modificação posterior localizada.
- **Última alteração localizada:** 2026-06-24, commit `init repository`.
- **Abrangência:** roteamento e handlers HTTP do módulo CUSTOMERS, integrados à composição da aplicação Express.
- **Temas recentes:** não há commits posteriores associados a substituição ou migração de framework nos arquivos consultados.

### Alternativas (se observáveis)

Nenhuma alternativa de framework ou justificativa comparativa foi encontrada nos arquivos e commits examinados.

## Questões para abordar no ADR formal (se criado)

- Quais requisitos levaram à escolha de Express?
- Quais alternativas foram avaliadas e quais critérios foram usados?
- Quais consequências de longo prazo são esperadas para roteamento e middlewares?

## Potenciais ADRs relacionados

- A escolha do Express também aparece em análises de candidatos nos módulos [API](../API/express-como-framework-principal.md), [PRODUCTS](../PRODUCTS/express-como-framework-http-do-modulo.md) e [ORDERS](../ORDERS/express-framework-orders.md). São análises de candidatos, não ADRs formais; ao formalizar, avaliar uma decisão compartilhada, evitando duplicidade por módulo.
- [Organizar as rotas de clientes como API REST](./api-rest-clientes.md)

## Observações adicionais

O histórico observado é limitado ao commit de inclusão disponível. A análise não atribui intenção à equipe com base apenas no uso do framework.
