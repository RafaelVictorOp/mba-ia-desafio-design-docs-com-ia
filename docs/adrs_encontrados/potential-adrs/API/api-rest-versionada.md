# Potencial ADR: API HTTP REST versionada sob `/api/v1`

**Módulo:** API  
**Categoria:** Arquitetura  
**Prioridade:** Documentação obrigatória (pontuação: 130)  
**Data da identificação:** 2026-11-08

---

## O que foi identificado

O projeto se descreve como uma API REST, e a aplicação Express publica seus routers de domínio sob o prefixo `/api/v1`, mantendo `/health` como endpoint separado. `buildApiRouter` agrega os routers de autenticação, usuários, clientes, produtos e pedidos. O código do módulo API evidencia uma interface HTTP com agrupamento por domínio e prefixo versionado.

Esta é uma escolha estrutural da Etapa 0 (protocolo/arquitetura de API). A análise não examinou a semântica de cada operação nos routers de domínio, pois o escopo solicitado foi somente o módulo API; portanto, não conclui se todos os endpoints seguem integralmente convenções REST. O histórico disponível dos arquivos centrais começa em 2026-06-24 e contém somente o commit inicial, sem mensagem que documente alternativas ou motivação.

## Por que essa decisão pode merecer um ADR

- **Impacto:** o prefixo e o estilo de interface alcançam os cinco routers agregados pelo módulo API.
- **Trade-offs:** a interface fica organizada sob uma versão explícita no caminho; regras de compatibilidade e evolução entre versões não estão registradas nos arquivos analisados.
- **Complexidade:** alterar o estilo ou a convenção de versionamento pode exigir mudanças distribuídas nas rotas e compatibilidade com consumidores. A base não contém evidência para estimar o esforço real ou identificar esses consumidores.
- **Conhecimento da equipe:** quem implementa ou integra funcionalidades HTTP precisa conhecer o agrupamento e o prefixo versionado comuns.
- **Implicações futuras:** decisões de compatibilidade, descontinuação e evolução de contratos podem depender desta convenção.
- **Pontuação:** base estrutural 75 + abrangência/impacto 20 + custo de mudança estimado 15 + conhecimento da equipe 20 = **130/150**. O custo é uma estimativa arquitetural (duas a oito semanas), não um prazo observado no histórico.
- **Estabilidade observável:** os arquivos centrais constam desde 2026-06-24, cerca de quatro meses e meio antes da identificação; isso não comprova intenção de permanência.

## Evidências encontradas no código

### Arquivos principais

- [`package.json`](../../../../../package.json) — linha 4 descreve o produto como `REST API`.
- [`src/app.ts`](../../../../../src/app.ts) — linhas 62-67 registram `/health` e montam o router sob `/api/v1`.
- [`src/routes/index.ts`](../../../../../src/routes/index.ts) — linhas 21-30 agregam os cinco routers por domínio.

### Evidências de código

```typescript
// src/app.ts:62-67
app.get('/health', (_req, res) => {
  res.status(200).json({ status: 'ok' });
});

const controllers = buildControllers(deps.prisma);
app.use('/api/v1', buildApiRouter(controllers));
```

```typescript
// src/routes/index.ts:21-30
export function buildApiRouter(controllers: Controllers): Router {
  const router = Router();

  router.use('/auth', buildAuthRouter(controllers.auth));
  router.use('/users', buildUserRouter(controllers.users));
  router.use('/customers', buildCustomerRouter(controllers.customers));
  router.use('/products', buildProductRouter(controllers.products));
  router.use('/orders', buildOrderRouter(controllers.orders));

  return router;
}
```

### Análise de impacto

- **Introdução:** 2026-06-24, data do primeiro commit disponível que inclui os arquivos centrais; não necessariamente a data da decisão.
- **Modificações:** um commit no histórico disponível de `src/app.ts`, `src/server.ts`, `src/routes/index.ts` e `package.json`.
- **Última alteração:** 2026-06-24 (`init repository`).
- **Abrangência:** três arquivos principais do módulo API e cinco routers agregados.
- **Temas recentes:** não foram encontrados commits posteriores ou mensagens relacionadas a versionamento, migração ou evolução de protocolo nesses arquivos.

## Questões para abordar no ADR formal (se criado)

- Quais requisitos justificaram o estilo REST e o prefixo de versão no caminho?
- Como a compatibilidade de consumidores e a evolução de versões devem ser tratadas?
- Quais alternativas de protocolo ou versionamento foram consideradas?
- Quais convenções REST são obrigatórias para os routers de domínio?

## Potenciais ADRs relacionados

- [Express como framework principal da aplicação HTTP](./express-como-framework-principal.md) — escolha relacionada de implementação, não duplicada por esta análise.
- Há análises candidatas de API REST nos módulos [PRODUCTS](../PRODUCTS/api-rest-para-catalogo-produtos.md), [ORDERS](../ORDERS/api-rest-orders.md) e [CUSTOMERS](../CUSTOMERS/api-rest-clientes.md). São candidatos existentes, não ADRs aprovados; recomenda-se consolidar o estilo e o versionamento da API em um único ADR, usando as análises dos módulos como evidências específicas.

## Observações adicionais

- O código consultado comprova a descrição REST no manifesto e o prefixo/agrupamento HTTP; a conformidade semântica dos endpoints não foi avaliada.
- Não foram encontrados ADRs formais na pasta `docs/adrs/generated/`; a pasta não está presente.
- A análise ficou restrita aos arquivos e responsabilidades do módulo API definidos em `mapping.md`.
