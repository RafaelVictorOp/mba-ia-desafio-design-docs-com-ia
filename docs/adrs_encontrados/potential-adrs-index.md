# Índice de potenciais ADRs

## Progresso da análise

### Módulos analisados
- **API**: Composição e transporte HTTP - 2026-11-08 - 2 alta prioridade, 0 média prioridade
- **PLATFORM**: Capacidades transversais - 2026-11-08 - 1 alta prioridade, 0 média prioridade
- **TESTS**: Verificação automatizada - 2026-11-08 - 0 alta prioridade, 0 média prioridade
- **USERS**: Usuários - 2026-11-08 - 1 alta prioridade, 0 média prioridade
- **CUSTOMERS**: Clientes - 2026-11-08 - 4 alta prioridade, 0 média prioridade
- **PRODUCTS**: Produtos e estoque - 2026-11-08 - 3 alta prioridade, 0 média prioridade
- **AUTH**: Autenticação - 2026-11-08 - 1 alta prioridade, 0 média prioridade
- **DATA**: Persistência - 2026-11-08 - 2 alta prioridade, 0 média prioridade
- **ORDERS**: Pedidos - 2026-11-08 - 3 de alta prioridade, 0 de prioridade média

### Análises pendentes
Nenhum módulo pendente; os nove módulos foram analisados.

## ADRs de alta prioridade (17 potenciais formalizadas)

### Módulo: PLATFORM

| Título | Categoria | Arquivo |
|---|---|---|
| Autentica??o de requisi??es com JWT | Seguran?a/Arquitetura | [Potencial arquivada](./potential-adrs/done/PLATFORM/autenticacao-jwt-bearer.md) ? [ADR-001](./generated/PLATFORM/ADR-001-autenticacao-de-requisicoes-com-jwt.md) |

### Módulo: USERS

| Título | Categoria | Arquivo |
|---|---|---|
| Hash de senhas com bcrypt no ciclo de vida de usuários | Segurança | [Potencial arquivada](./potential-adrs/done/USERS/hash-de-senhas-com-bcrypt.md) · [ADR-002](./generated/USERS/ADR-002-hash-de-senhas-com-bcrypt.md) |

### Módulo: PRODUCTS

| Título | Categoria | Arquivo |
|---|---|---|
| Uso do Express como framework HTTP no módulo PRODUCTS | Tecnologia (framework principal) | [Potencial arquivada](./potential-adrs/done/PRODUCTS/express-como-framework-http-do-modulo.md) · [ADR-003](./generated/PRODUCTS/ADR-003-express-como-framework-http.md) |
| API REST para o catálogo de produtos | Arquitetura (protocolo/estilo de API) | [Potencial arquivada](./potential-adrs/done/PRODUCTS/api-rest-para-catalogo-produtos.md) · [ADR-004](./generated/PRODUCTS/ADR-004-api-rest-para-catalogo-de-produtos.md) |
| Prisma como camada de acesso a dados de produtos | Tecnologia (ORM) | [Potencial arquivada](./potential-adrs/done/PRODUCTS/prisma-como-camada-de-acesso-a-dados.md) · [ADR-005](./generated/PRODUCTS/ADR-005-prisma-como-camada-de-dados-de-produtos.md) |

### Módulo: AUTH

| Título | Categoria | Arquivo |
|---|---|---|
| Estrat?gia de autentica??o local com bcrypt e JWT | Seguran?a / Arquitetura | [Potencial arquivada](./potential-adrs/done/AUTH/estrategia-de-autenticacao-local-jwt.md) ? [ADR-006](./generated/AUTH/ADR-006-estrategia-de-autenticacao-local-com-jwt.md) |

### Módulo: ORDERS

| Título | Categoria | Arquivo |
|---|---|---|
| Prisma como ORM no módulo ORDERS | Tecnologia (ORM/camada de acesso a dados) | [Potencial arquivada](./potential-adrs/done/ORDERS/prisma-orm-orders.md) · [ADR-007](./generated/ORDERS/ADR-007-prisma-como-orm-em-pedidos.md) |
| Express como framework HTTP no módulo ORDERS | Arquitetura (framework/plataforma) | [Potencial arquivada](./potential-adrs/done/ORDERS/express-framework-orders.md) · [ADR-008](./generated/ORDERS/ADR-008-express-como-framework-em-pedidos.md) |
| API HTTP em estilo REST no módulo ORDERS | Arquitetura (protocolo/estilo de API) | [Potencial arquivada](./potential-adrs/done/ORDERS/api-rest-orders.md) · [ADR-009](./generated/ORDERS/ADR-009-api-rest-em-pedidos.md) |

### Módulo: DATA — Persistência

| Título | Categoria | Arquivo |
|---|---|---|
| Uso do MySQL como banco relacional da aplicação | Tecnologia (infraestrutura de dados) | [Potencial arquivada](./potential-adrs/done/DATA/banco-relacional-mysql.md) · [ADR-016](./generated/DATA/ADR-016-mysql-como-banco-relacional.md) |
| Uso do Prisma como ORM e camada de acesso a dados | Tecnologia (ORM/camada de acesso a dados) | [Potencial arquivada](./potential-adrs/done/DATA/orm-prisma.md) · [ADR-017](./generated/DATA/ADR-017-prisma-como-orm.md) |

### Módulo: API

| Título | Categoria | Arquivo |
|---|---|---|
| Express como framework principal da aplicação HTTP | Tecnologia | [Potencial arquivada](./potential-adrs/done/API/express-como-framework-principal.md) · [ADR-010](./generated/API/ADR-010-express-como-framework-principal.md) |
| API HTTP REST versionada sob `/api/v1` | Arquitetura | [Potencial arquivada](./potential-adrs/done/API/api-rest-versionada.md) · [ADR-011](./generated/API/ADR-011-api-rest-versionada.md) |

### Módulo: CUSTOMERS

| Título | Categoria | Arquivo |
|---|---|---|
| Persistir os dados de clientes em MySQL | Tecnologia (infraestrutura) | [Potencial arquivada](./potential-adrs/done/CUSTOMERS/banco-de-dados-mysql.md) · [ADR-012](./generated/CUSTOMERS/ADR-012-mysql-para-dados-de-clientes.md) |
| Estruturar a API de clientes com Express | Tecnologia (framework principal) | [Potencial arquivada](./potential-adrs/done/CUSTOMERS/framework-express.md) · [ADR-013](./generated/CUSTOMERS/ADR-013-express-para-api-de-clientes.md) |
| Usar Prisma como camada de acesso a dados de clientes | Tecnologia (ORM) | [Potencial arquivada](./potential-adrs/done/CUSTOMERS/camada-de-acesso-a-dados-prisma.md) · [ADR-014](./generated/CUSTOMERS/ADR-014-prisma-para-dados-de-clientes.md) |
| Expor operações de clientes por uma API REST | Arquitetura (protocolo/estilo de API) | [Potencial arquivada](./potential-adrs/done/CUSTOMERS/api-rest-clientes.md) · [ADR-015](./generated/CUSTOMERS/ADR-015-api-rest-para-clientes.md) |

## ADRs de prioridade média (consider/)

Nenhum candidato nesta prioridade para os nove módulos analisados.

## Decisões avaliadas e descartadas

### Módulo: TESTS

| Decisão avaliada | Resultado | Pontuação | Justificativa |
|---|---|---:|---|
| Estratégia de testes HTTP de integração com persistência MySQL | Descartada; não gerar arquivo individual | 40/150 | A prática é estrutural para TESTS, está explícita na configuração e nos testes e tende a persistir enquanto a suíte for mantida. Abrangência: 10 pontos (um ou dois módulos diretamente envolvidos); custo de mudança: 15 (estimativa de duas a oito semanas para substituir a abordagem nos testes existentes); conhecimento: 15 (importante para quem altera a suíte e sua integração com API e dados). A soma fica abaixo do limiar de 75. |

**Etapa 0:** Vitest é o framework de testes, mas não o framework principal que estrutura a aplicação; a escolha do framework principal da aplicação e do protocolo da API pertence a módulos fora do escopo TESTS. MySQL é a infraestrutura de persistência mapeada em DATA, não uma decisão independente identificada no código de TESTS.

**Regra dos três Es:** a estratégia é estrutural porque define execução e isolamento dos testes; é evidente porque aparece diretamente na configuração, no setup e nas suítes; e tende a ser estável enquanto essas suítes forem mantidas. A estabilidade futura é uma avaliação técnica, não algo comprovado pelo histórico disponível.

**Evidências:** `vitest.config.ts:8-15` configura `tests/setup.ts`, execução sem paralelismo entre arquivos e um único fork; `tests/setup.ts:4-16` conecta ao Prisma e limpa os registros entre testes; `tests/auth.test.ts:8-81` e `tests/orders.test.ts:19-219` exercitam rotas HTTP com Supertest; os testes de pedidos também consultam o Prisma para confirmar efeitos persistidos. `package.json:17-18` define `test` como `vitest run` e `test:watch` como `vitest`.

**Histórico e comparação:** os arquivos relevantes aparecem no commit inicial `init repository`, de 2026-06-24, sem commits posteriores encontrados que registrem evolução ou motivação. A data comprova somente a inclusão no repositório. Não foram encontrados ADRs formais para comparação; não há duplicidade lexical identificável.

**Limite da análise:** as estimativas de custo, abrangência e conhecimento são aplicação heurística da rubrica, não fatos documentados por commits. O resultado não transforma comportamentos ou regras de negócio exercitados pelos testes em decisões arquiteturais.

### Módulo: PRODUCTS

| Decisão avaliada | Resultado | Pontuação | Justificativa |
|---|---|---:|---|
| Representar o preço do produto como inteiro em centavos | Descartada; não gerar arquivo individual | 40/150 | A convenção aparece em `product.schemas.ts`, que valida `priceCents` como inteiro não negativo, e em `product.service.ts`, que entrega esse campo à persistência. Passa pelos critérios estrutural, evidente e estável; a pontuação heurística é 10 para impacto (escopo direto identificado em PRODUCTS), 15 para custo de mudança (estimativa de duas a oito semanas para alterar representação, contratos e dados) e 15 para conhecimento (importante nas áreas que manipulam preços). Total abaixo de 75. A análise não examinou outros módulos para medir a abrangência total. |

**Limite da análise PRODUCTS:** as categorias estruturais universais identificadas foram registradas acima como candidatos, mas devem ser verificadas contra decisões compartilhadas antes de formalização. MySQL não foi analisado como decisão própria do módulo: sua infraestrutura está fora do escopo PRODUCTS e consta no mapeamento. O diretório `docs/adrs/generated/` não existe; a análise não encontrou ADRs formais para comparação. O histórico dos arquivos PRODUCTS mostra apenas a inclusão inicial em 2026-06-24 e não documenta as motivações das escolhas.

## Resumo

- Alta prioridade: 17
- Média prioridade: 0
- Total: 17
- Módulos analisados: 9 de 9

**Limitações:** não havia ADR formal no diretório padrão `docs/adrs/generated/`; o histórico Git dos arquivos examinados contém somente o commit de inclusão. As análises foram executadas por módulo; candidatos semelhantes podem representar decisões transversais e devem ser consolidados antes da formalização para evitar duplicidade.

**Nota de seleção:** logging estruturado com redação de campos sensíveis, tratamento centralizado de erros e validação HTTP foram avaliados como capacidades transversais fora da Etapa 0. Cada um passou pelos critérios estrutural, evidente e estável; as estimativas ficaram abaixo do limiar de 75 pontos (45/150 para cada: impacto 20, custo de mudança 10 e conhecimento da equipe 15), portanto foram descartados como candidatos independentes. O custo é estimativo, não histórico. Detalhes de expiração do token foram mantidos subordinados à estratégia de autenticação.
