# Potencial ADR: Express como framework principal da aplicação HTTP

**Módulo:** API  
**Categoria:** Tecnologia  
**Prioridade:** Documentação obrigatória (pontuação: 130)  
**Data da identificação:** 2026-11-08

---

## O que foi identificado

O módulo API usa Express para criar a aplicação HTTP, registrar middlewares e montar o router de domínio. `src/server.ts` inicializa a aplicação construída em `src/app.ts`; o pacote também declara `express` como dependência direta. A composição explícita conecta os controllers dos domínios aos routers, mantendo a inicialização do servidor separada da construção da aplicação.

Esta é uma escolha estrutural da Etapa 0 (framework principal). O histórico disponível mostra os arquivos da API desde o commit `init repository`, em 2026-06-24, e não mostra commits posteriores nesses arquivos nem mensagens associadas a migração ou seleção de framework. Isso comprova a presença do padrão desde o primeiro commit disponível, mas não determina quando ou por que a decisão foi tomada.

## Por que essa decisão pode merecer um ADR

- **Impacto:** Express é o framework central para a composição HTTP e o registro dos cinco routers de domínio listados pelo módulo API.
- **Trade-offs:** a aplicação depende do modelo de aplicação, middleware e `Router` do Express. Não foram encontradas alternativas explicitamente comparadas no código ou nas mensagens de commit consultadas.
- **Complexidade:** mudar o framework exigiria adaptar a inicialização e a integração de rotas e middlewares; a extensão exata do trabalho não pode ser determinada apenas por este módulo.
- **Conhecimento da equipe:** entender o fluxo Express é necessário para trabalhar no transporte HTTP centralizado da aplicação.
- **Implicações futuras:** decisões sobre middlewares, tratamento de erros e composição dos routers permanecem vinculadas ao framework adotado.
- **Pontuação:** base estrutural 75 + abrangência/impacto 20 + custo de mudança estimado 15 + conhecimento da equipe 20 = **130/150**. O custo é uma estimativa arquitetural (duas a oito semanas), não um prazo observado no histórico.
- **Estabilidade observável:** arquivos presentes desde 2026-06-24, cerca de quatro meses e meio antes da identificação; esse dado não comprova intenção de permanência.

## Evidências encontradas no código

### Arquivos principais

- [`package.json`](../../../../../package.json) — linha 28 declara `express` como dependência de execução.
- [`src/app.ts`](../../../../../src/app.ts) — linhas 1 e 55-75 importam Express, criam a aplicação e registram middleware, health check, router e handlers finais.
- [`src/server.ts`](../../../../../src/server.ts) — linhas 6-10 constroem a aplicação e iniciam o listener HTTP.
- [`src/routes/index.ts`](../../../../../src/routes/index.ts) — linhas 1 e 21-30 usam `Router` do Express para agregar os routers de domínio.

### Evidências de código

```typescript
// src/app.ts:55-67
export function buildApp(deps: AppDependencies): Express {
  const app = express();

  app.disable('x-powered-by');
  app.use(express.json({ limit: '1mb' }));
  app.use(requestLogger);

  app.get('/health', (_req, res) => {
    res.status(200).json({ status: 'ok' });
  });

  const controllers = buildControllers(deps.prisma);
  app.use('/api/v1', buildApiRouter(controllers));
```

### Análise de impacto

- **Introdução:** 2026-06-24, data do primeiro commit disponível que inclui `src/app.ts`; não necessariamente a data da decisão.
- **Modificações:** um commit no histórico disponível dos arquivos centrais da API; o conteúdo consultado não revela uma série de alterações arquiteturais.
- **Última alteração:** 2026-06-24 (`init repository`).
- **Abrangência:** três arquivos principais no módulo API; a composição registrada conecta cinco routers de domínio.
- **Temas recentes:** não há commits posteriores nem mensagens de commit com temas de migração, desempenho ou segurança nesses arquivos.

## Questões para abordar no ADR formal (se criado)

- Qual necessidade levou à escolha do Express?
- Quais alternativas foram efetivamente avaliadas?
- Quais propriedades de middleware, roteamento e ecossistema foram relevantes?
- Quais consequências de longo prazo devem orientar uma eventual substituição?

## Potenciais ADRs relacionados

- [API HTTP REST versionada em `/api/v1`](./api-rest-versionada.md) — escolha relacionada, mas distinta: framework e estilo/protocolo de interface não são a mesma decisão.
- Há análises candidatas de Express nos módulos [PRODUCTS](../PRODUCTS/express-como-framework-http-do-modulo.md), [ORDERS](../ORDERS/express-framework-orders.md) e [CUSTOMERS](../CUSTOMERS/framework-express.md). São candidatos existentes, não ADRs aprovados; recomenda-se consolidar a decisão de framework em um único ADR transversal da API, evitando formalizações repetidas por módulo.

## Observações adicionais

- Não foram encontrados ADRs formais na pasta `docs/adrs/generated/`; a pasta não está presente. `docs/adrs/README.md` contém apenas instruções gerais de armazenamento e não registra uma decisão.
- A análise ficou restrita aos arquivos e responsabilidades do módulo API definidos em `mapping.md`.
