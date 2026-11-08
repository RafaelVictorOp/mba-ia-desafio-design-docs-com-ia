# Potencial ADR: Expor operações de clientes por uma API REST

**Módulo:** CUSTOMERS  
**Categoria:** Arquitetura  
**Prioridade:** Documentação obrigatória (pontuação: 100)  
**Data da identificação:** 2026-11-08

---

## O que foi identificado

O módulo expõe operações HTTP de clientes por meio de rotas com `GET`, `POST`, `PATCH` e `DELETE`. Os handlers retornam respostas JSON e códigos HTTP, como `200`, `201` e `204`. Esse conjunto constitui uma API de estilo REST observável no módulo CUSTOMERS.

Esta é uma decisão estrutural da Categoria 4 da Etapa 0 (protocolo ou arquitetura de API). A análise confirma o padrão implementado, mas não encontra registro da motivação, das alternativas avaliadas ou de uma decisão formal. A organização das rotas e handlers usa Express, documentado em candidato separado.

O histórico Git disponível registra um commit de inclusão (`init repository`, 2026-06-24) nos arquivos consultados, sem alterações posteriores localizadas. Isso não permite concluir quando a opção por REST foi deliberada nem qual foi sua justificativa.

## Por que essa decisão pode merecer um ADR

- **Impacto:** define os contratos HTTP de consulta e manutenção de clientes consumidos pela aplicação.
- **Trade-offs:** o módulo expõe operações por verbos, rotas e respostas HTTP; não foi encontrada alternativa documentada, como GraphQL ou gRPC.
- **Complexidade:** alterar o estilo de API exigiria mudanças de contrato e de handlers; a pontuação de custo usa o mínimo previsto para decisões estruturais.
- **Conhecimento da equipe:** manutenção e integração dos endpoints dependem do entendimento dos contratos e códigos HTTP utilizados.
- **Implicações futuras:** consumidores do módulo dependem do estilo de comunicação atualmente exposto.

**Pontuação:** base estrutural 75 + abrangência 10 + custo de mudança 10 + conhecimento da equipe 5 = **100/150**. Aplicam-se os pisos da categoria estrutural universal; as parcelas não representam estimativa de esforço medida.

## Evidências encontradas no código

### Arquivos principais

- [`src/modules/customers/customer.routes.ts`](../../../../../src/modules/customers/customer.routes.ts) — linhas 16-25: rotas de listagem, consulta, criação, atualização parcial e exclusão.
- [`src/modules/customers/customer.controller.ts`](../../../../../src/modules/customers/customer.controller.ts) — linhas 8-53: handlers HTTP e respostas JSON com códigos de status.
- [`src/routes/index.ts`](../../../../../src/routes/index.ts) — linha 26: registro do router de CUSTOMERS no roteamento da API.

### Evidências de código

```typescript
router.get('/', validate({ query: listCustomersQuerySchema }), controller.list);
router.get('/:id', validate({ params: customerIdParamSchema }), controller.getById);
router.post('/', validate({ body: createCustomerSchema }), controller.create);
router.patch(
  '/:id',
  validate({ params: customerIdParamSchema, body: updateCustomerSchema }),
  controller.update,
);
router.delete('/:id', validate({ params: customerIdParamSchema }), controller.delete);
```

Trecho de `src/modules/customers/customer.routes.ts:16-25`.

### Análise de impacto

- **Introdução observada no histórico:** os arquivos consultados aparecem no commit `init repository`, em 2026-06-24; isso não estabelece quando ou por que REST foi escolhido.
- **Modificações:** 1 commit de inclusão encontrado nos arquivos consultados; nenhuma modificação posterior localizada.
- **Última alteração localizada:** 2026-06-24, commit `init repository`.
- **Abrangência:** operações HTTP e contratos expostos pelo módulo CUSTOMERS.
- **Temas recentes:** não há commits posteriores associados a mudança do estilo da API nos arquivos consultados.

### Alternativas (se observáveis)

Nenhuma alternativa de protocolo/estilo de API foi comparada nos arquivos e commits examinados.

## Questões para abordar no ADR formal (se criado)

- Quais necessidades levaram à escolha do estilo REST para a API?
- GraphQL, gRPC ou outro estilo foram considerados?
- Quais princípios de compatibilidade e evolução dos contratos devem ser mantidos?

## Potenciais ADRs relacionados

- O estilo REST também aparece em análises de candidatos nos módulos [API](../API/api-rest-versionada.md), [PRODUCTS](../PRODUCTS/api-rest-para-catalogo-produtos.md) e [ORDERS](../ORDERS/api-rest-orders.md). São análises de candidatos, não ADRs formais; avaliar uma decisão compartilhada em vez de formalizar registros redundantes por módulo.
- [Estruturar a API de clientes com Express](./framework-express.md)

## Observações adicionais

A análise se restringe aos endpoints de CUSTOMERS e não avalia as APIs dos demais módulos.
