# Visão geral do workspace

O workspace contém um projeto de aplicação identificado como `order-management-api`: uma API REST para gestão de usuários, clientes, produtos e pedidos. O código está organizado em módulos dentro de uma única aplicação Express e compartilha uma persistência MySQL acessada por Prisma. O Docker Compose configura o banco de dados, mas não define uma segunda aplicação de negócio. Evidências: [package.json](../package.json), [src/app.ts](../src/app.ts), [src/routes/index.ts](../src/routes/index.ts) e [docker-compose.yml](../docker-compose.yml).

Não foram encontrados projetos de frontend, bibliotecas compartilhadas, gateway, worker ou outros serviços de aplicação separados no código/configuração atual. A documentação descreve uma feature futura de webhooks com worker, mas ela não deve ser confundida com componente implementado; os detalhes estão em [Itens não confirmados](#itens-não-confirmados).

# Projetos

## `order-management-api`

- **Tipo de projeto:** aplicação backend/API REST, em um único pacote Node.js.
- **Tecnologia / framework principal:** Node.js `>=20`, TypeScript, Express 4, Prisma e MySQL. As versões/dependências são declaradas em [package.json](../package.json); o datasource MySQL está em [prisma/schema.prisma](../prisma/schema.prisma).
- **Responsabilidade:** autenticar usuários e expor operações de gestão de usuários, clientes, produtos e pedidos por rotas HTTP. A composição da aplicação, o endpoint `/health` e o prefixo `/api/v1` estão em [src/app.ts](../src/app.ts); as rotas de domínio são agregadas em [src/routes/index.ts](../src/routes/index.ts).
- **Principais módulos:**
  - **auth:** registro, login e consulta do usuário autenticado; as rotas estão em [src/modules/auth/auth.routes.ts](../src/modules/auth/auth.routes.ts).
  - **users:** consulta de usuário, com rota protegida por `ADMIN`; [src/modules/users/user.routes.ts](../src/modules/users/user.routes.ts).
  - **customers:** cadastro, consulta, atualização, remoção e listagem de clientes; [src/modules/customers/customer.routes.ts](../src/modules/customers/customer.routes.ts).
  - **products:** cadastro, consulta, atualização, remoção e listagem do catálogo; [src/modules/products/product.routes.ts](../src/modules/products/product.routes.ts).
  - **orders:** criação, consulta, listagem, exclusão e alteração de status de pedidos; [src/modules/orders/order.routes.ts](../src/modules/orders/order.routes.ts). O serviço de pedidos valida transições, atualiza estoque e registra histórico dentro de transações Prisma, conforme [src/modules/orders/order.service.ts](../src/modules/orders/order.service.ts) e [src/modules/orders/order.status.ts](../src/modules/orders/order.status.ts).
- **Dependências internas:** os módulos usam controllers, services e repositories para separar HTTP, orquestração de domínio e acesso a dados. A composição concreta é feita em `buildControllers` em [src/app.ts](../src/app.ts). O módulo auth utiliza o módulo users para localizar e criar usuários, conforme [src/modules/auth/auth.service.ts](../src/modules/auth/auth.service.ts). Os repositories recebem Prisma; o serviço de pedidos também usa o cliente Prisma para transações.
- **Integrações externas:** recebe chamadas HTTP/JSON de consumidores da API. Não foi identificada implementação de chamadas de saída para serviços externos no código atual. O acesso ao MySQL é a dependência de infraestrutura persistente, não uma integração com outro projeto de aplicação. As interfaces REST e autenticação são comprovadas por [src/app.ts](../src/app.ts), [src/routes/index.ts](../src/routes/index.ts) e pelas rotas de domínio acima.
- **Persistência utilizada:** MySQL, via Prisma Client. O schema declara `provider = "mysql"` e modela `User`, `Customer`, `Product`, `Order`, `OrderItem`, `OrderStatusHistory` e `OrderNumberSequence`; ver [prisma/schema.prisma](../prisma/schema.prisma). A migration inicial cria as tabelas correspondentes em [prisma/migrations/20260519182739_init/migration.sql](../prisma/migrations/20260519182739_init/migration.sql).
- **Forma de comunicação com outros projetos:** não há outro projeto de aplicação identificado com o qual esta API se comunique. Os módulos colaboram por chamadas dentro do mesmo processo; o acesso ao banco ocorre via Prisma/MySQL. Consumidores não identificados acessam a API por HTTP/REST.

## MySQL — componente de infraestrutura, não projeto de aplicação

- **Tipo:** serviço de banco de dados configurado no Docker Compose.
- **Tecnologia principal:** MySQL 8.0, com volume local nomeado e healthcheck; [docker-compose.yml](../docker-compose.yml).
- **Responsabilidade:** persistir os dados relacionais do OMS. O schema Prisma define os modelos e relações; [prisma/schema.prisma](../prisma/schema.prisma).
- **Dependência relacionada:** a API configura o datasource MySQL por `DATABASE_URL`; [src/config/env.ts](../src/config/env.ts) e [src/config/database.ts](../src/config/database.ts).
- **Comunicação:** a aplicação acessa o banco por Prisma Client. O Compose configura somente o serviço MySQL, não uma segunda API ou worker.

# Mapa de responsabilidades

| Responsabilidade | Projeto/componente |
| --- | --- |
| Expor API REST e endpoint de saúde | `order-management-api` |
| Registro, login e identidade JWT | `order-management-api` — módulo auth |
| Registro e consulta de usuários; controle de acesso por papel | `order-management-api` — módulos auth/users e middleware de autenticação |
| Cadastro e consulta de clientes | `order-management-api` — módulo customers |
| Gestão do catálogo e estoque de produtos | `order-management-api` — módulo products |
| Criação e ciclo de vida de pedidos, status, itens e histórico | `order-management-api` — módulo orders |
| Persistência transacional dos dados do OMS | MySQL — componente de infraestrutura |
| Execução de worker de webhooks | Não localizado como componente implementado; ver [Itens não confirmados](#itens-não-confirmados) |
| Interface visual/front-end | Não localizado como projeto no workspace |

# Comunicação entre projetos

Não há comunicação comprovada entre projetos de aplicação distintos: foi identificado apenas `order-management-api`.

- `Consumidores HTTP não identificados -> HTTP/REST -> order-management-api`: a aplicação recebe JSON, agrega rotas sob `/api/v1` e expõe `/health`; evidência em [src/app.ts](../src/app.ts) e [src/routes/index.ts](../src/routes/index.ts). O workspace não identifica os sistemas consumidores concretos.
- `order-management-api -> Prisma Client/driver de banco -> MySQL`: datasource MySQL no schema e cliente Prisma criado pela aplicação; evidências em [prisma/schema.prisma](../prisma/schema.prisma) e [src/config/database.ts](../src/config/database.ts).
- `Módulos da API -> chamadas no mesmo processo -> outros módulos/serviços`: dependências são montadas diretamente pela aplicação; evidência em [src/app.ts](../src/app.ts). Isso é comunicação interna da aplicação, não comunicação entre projetos separados.

Não foi encontrada configuração ou implementação de broker, fila externa ou event bus no código/configuração atual. A outbox e o worker descritos na documentação são proposta de feature, não comunicação já disponível.

# Fluxos macro

- **Requisição de domínio:** `Consumidor HTTP -> Express/middlewares -> rota e controller -> service -> repository/Prisma -> MySQL -> resposta HTTP`. A montagem da API está em [src/app.ts](../src/app.ts); a composição das rotas em [src/routes/index.ts](../src/routes/index.ts); o serviço de pedidos demonstra o uso de transação Prisma em [src/modules/orders/order.service.ts](../src/modules/orders/order.service.ts).
- **Autenticação:** `Cliente -> POST /api/v1/auth/login -> serviço auth/consulta de usuário -> JWT Bearer -> middleware authenticate -> rota protegida`. As rotas estão em [src/modules/auth/auth.routes.ts](../src/modules/auth/auth.routes.ts), a emissão do token em [src/modules/auth/auth.service.ts](../src/modules/auth/auth.service.ts) e sua validação/autorização por papel em [src/middlewares/auth.middleware.ts](../src/middlewares/auth.middleware.ts).
- **Mudança de status do pedido:** `PATCH /api/v1/orders/:id/status -> OrderService -> transação MySQL (pedido, histórico e ajuste de estoque)`. O fluxo atual está em [src/modules/orders/order.routes.ts](../src/modules/orders/order.routes.ts) e [src/modules/orders/order.service.ts](../src/modules/orders/order.service.ts). Não inclui publicação de evento de webhook no código atual.

# Principais módulos de negócio

| Módulo | Projeto | Escopo comprovado |
| --- | --- | --- |
| Auth | `order-management-api` | Registro, login, JWT e consulta do usuário autenticado. |
| Users | `order-management-api` | Consulta de usuário; papéis são definidos no modelo e aplicados pelo middleware de autorização. |
| Customers | `order-management-api` | CRUD e listagem de clientes. |
| Products | `order-management-api` | CRUD e listagem do catálogo; o modelo inclui preço, estado ativo e quantidade em estoque. |
| Orders | `order-management-api` | Criação/listagem/consulta/exclusão, alteração de status, itens, histórico e atualização de estoque transacional. |
| Webhooks de pedidos | Planejado na documentação; implementação não localizada | PRD descreve cadastro, outbox, worker, entrega, tentativas, DLQ e replay. Não há módulo ou worker correspondente no código atual. |

# Integrações externas

| Sistema ou serviço | Projeto responsável | Finalidade | Mecanismo | Evidência / situação |
| --- | --- | --- | --- | --- |
| MySQL 8.0 | `order-management-api` consome; MySQL é infraestrutura | Persistência do OMS | Prisma Client e conexão configurada por `DATABASE_URL` | [prisma/schema.prisma](../prisma/schema.prisma), [src/config/database.ts](../src/config/database.ts), [docker-compose.yml](../docker-compose.yml). |
| Consumidores HTTP da API (não identificados no workspace) | `order-management-api` | Acesso às operações de autenticação, usuários, clientes, produtos e pedidos | HTTP/REST com JSON; JWT Bearer nas rotas protegidas | [src/app.ts](../src/app.ts), [src/routes/index.ts](../src/routes/index.ts) e arquivos `*.routes.ts` dos módulos. Os sistemas consumidores concretos não são informados. |
| Endpoints HTTPS de clientes B2B para receber notificações de pedido | Não há integração implementada identificada | Entrega outbound de eventos de mudança de status, conforme feature proposta | POST HTTPS com assinatura, outbox e worker, segundo a documentação | Previsto em [docs/PRD.md](PRD.md) e [docs/PRD.JSON](PRD.JSON); não localizado em `src/modules`, [src/server.ts](../src/server.ts), [package.json](../package.json) ou [prisma/schema.prisma](../prisma/schema.prisma). Ver [Itens não confirmados](#itens-não-confirmados). |

Não foram incluídos valores de variáveis, credenciais ou segredos neste documento.

# Tecnologias utilizadas

- **Backend:** Node.js `>=20`, TypeScript, Express 4; [package.json](../package.json), [tsconfig.json](../tsconfig.json).
- **Banco de dados:** MySQL 8.0; [docker-compose.yml](../docker-compose.yml), [prisma/schema.prisma](../prisma/schema.prisma).
- **ORM/migrations:** Prisma e Prisma Client; schema, migration e scripts de banco declarados em [prisma/schema.prisma](../prisma/schema.prisma), [prisma/migrations/20260519182739_init/migration.sql](../prisma/migrations/20260519182739_init/migration.sql) e [package.json](../package.json).
- **Autenticação/segurança:** `jsonwebtoken` para JWT e `bcrypt` para verificação/hash de senha; [package.json](../package.json), [src/modules/auth/auth.service.ts](../src/modules/auth/auth.service.ts), [src/middlewares/auth.middleware.ts](../src/middlewares/auth.middleware.ts).
- **Validação:** Zod, usado para validar ambiente e requisições; [src/config/env.ts](../src/config/env.ts), [src/middlewares/validate.middleware.ts](../src/middlewares/validate.middleware.ts).
- **Logging/observabilidade:** Pino e `pino-http`; [package.json](../package.json), [src/shared/logger/index.ts](../src/shared/logger/index.ts), [src/middlewares/request-logger.middleware.ts](../src/middlewares/request-logger.middleware.ts).
- **Infraestrutura local:** Docker Compose para MySQL; [docker-compose.yml](../docker-compose.yml).
- **Testes:** Vitest e Supertest, com configuração em [vitest.config.ts](../vitest.config.ts) e dependências/scripts em [package.json](../package.json); há testes de autenticação e pedidos em [tests/auth.test.ts](../tests/auth.test.ts) e [tests/orders.test.ts](../tests/orders.test.ts).
- **Build e qualidade:** TypeScript, ESLint e Prettier; scripts e dependências em [package.json](../package.json), configuração de build em [tsconfig.build.json](../tsconfig.build.json).

# Estrutura arquitetural

A aplicação é organizada como um monólito modular em camadas: um único pacote e aplicação Express reúne módulos de domínio; cada módulo separa rotas/controllers, services, repositories e schemas conforme sua necessidade. O wiring é feito manualmente em `buildControllers`; evidência em [src/app.ts](../src/app.ts), [src/routes/index.ts](../src/routes/index.ts) e nos arquivos da pasta [src/modules](../src/modules).

O processamento HTTP usa middleware de logging, validação, autenticação/autorização e tratamento de erros. O acesso a dados passa por Prisma e a lógica de pedidos utiliza transações; evidências em [src/app.ts](../src/app.ts), [src/middlewares](../src/middlewares), [src/config/database.ts](../src/config/database.ts) e [src/modules/orders/order.service.ts](../src/modules/orders/order.service.ts).

O workspace não comprova arquitetura de microsserviços, frontend separado, mensageria/event-driven ou worker implantável. Esses termos não são atribuídos à implementação atual.

# Glossário do domínio

| Termo | Descrição baseada nas evidências |
| --- | --- |
| Usuário (`User`) | Identidade com email, hash de senha e papel `ADMIN` ou `OPERATOR`; [prisma/schema.prisma](../prisma/schema.prisma). |
| Cliente (`Customer`) | Entidade de cliente que pode possuir vários pedidos; [prisma/schema.prisma](../prisma/schema.prisma). |
| Produto (`Product`) | Item de catálogo com SKU, preço em centavos, indicador ativo e quantidade em estoque; [prisma/schema.prisma](../prisma/schema.prisma). |
| Pedido (`Order`) | Compra associada a um cliente e a um usuário criador; mantém total, status e itens; [prisma/schema.prisma](../prisma/schema.prisma). |
| Item do pedido (`OrderItem`) | Associação de produto, quantidade, preço unitário e total dentro de um pedido; [prisma/schema.prisma](../prisma/schema.prisma). |
| Status do pedido (`OrderStatus`) | Estado do pedido. Os valores declarados são `PENDING`, `PAID`, `PROCESSING`, `SHIPPED`, `DELIVERED` e `CANCELLED`; [prisma/schema.prisma](../prisma/schema.prisma). |
| Histórico de status (`OrderStatusHistory`) | Registro de status anterior/novo, data, usuário responsável e motivo opcional; [prisma/schema.prisma](../prisma/schema.prisma). |
| Sequência de pedido (`OrderNumberSequence`) | Registro usado pelo serviço de pedidos para reservar o próximo número de pedido; [prisma/schema.prisma](../prisma/schema.prisma), [src/modules/orders/order.service.ts](../src/modules/orders/order.service.ts). |
| Papel (`UserRole`) | Nível de autorização do usuário: `ADMIN` ou `OPERATOR`; [prisma/schema.prisma](../prisma/schema.prisma), [src/middlewares/auth.middleware.ts](../src/middlewares/auth.middleware.ts). |

# Conceitos importantes

- As rotas da API são montadas sob `/api/v1`; `/health` é exposto separadamente. [src/app.ts](../src/app.ts)
- As rotas de domínio exigem JWT conforme a configuração de cada router; `requireRole` limita rotas explicitamente restritas por papel. [src/middlewares/auth.middleware.ts](../src/middlewares/auth.middleware.ts) e arquivos `*.routes.ts`.
- Mudanças de status de pedido verificam transições permitidas e podem debitar/repor estoque, registrando o histórico dentro da transação do pedido. [src/modules/orders/order.service.ts](../src/modules/orders/order.service.ts), [src/modules/orders/order.status.ts](../src/modules/orders/order.status.ts)
- O modelo de dados e a migration inicial são a referência concreta das entidades persistidas atuais; não incluem tabelas de configuração/outbox/DLQ de webhook. [prisma/schema.prisma](../prisma/schema.prisma), [prisma/migrations/20260519182739_init/migration.sql](../prisma/migrations/20260519182739_init/migration.sql)
- A documentação tem níveis distintos de conclusão: `docs/PRD.md` e `docs/PRD.JSON` descrevem a feature de webhooks; `docs/RFC.md` e `docs/FDD.md` contêm marcadores de documento a elaborar; `docs/adrs/README.md` documenta o diretório, sem ADR individual listado. O [README.md](../README.md) da raiz está identificado como modelo para preenchimento, não como documentação operacional completa.

# Itens não confirmados

| Hipótese / informação | Evidências que motivaram a hipótese | O que faltou para confirmar |
| --- | --- | --- |
| Um worker separado processará notificações de mudança de status por webhooks outbound. | A feature é descrita como futura em [docs/PRD.md](PRD.md), [docs/PRD.JSON](PRD.JSON) e [TRANSCRICAO.md](../TRANSCRICAO.md). | Não há `src/worker.ts`, módulo `src/modules/webhooks`, script `worker` em [package.json](../package.json), nem modelos/tabelas de outbox, deliveries ou DLQ em [prisma/schema.prisma](../prisma/schema.prisma). Assim, o comportamento está documentado como proposta, não comprovado como implementação disponível. |
| Atlas Comercial, MaxDistribuição e Nova Cargo são sistemas integrados atualmente. | Esses nomes aparecem como clientes solicitantes no PRD e na transcrição. | Não foram encontrados endpoints/configurações concretas, clientes de integração ou evidência de chamadas em execução no código. A documentação só comprova a solicitação/planejamento. |
| Existem aplicações consumidoras específicas que chamam a API. | A aplicação expõe rotas REST para consumidores HTTP. | O workspace não identifica aplicações, repositórios ou nomes de sistemas consumidores; por isso, os consumidores são descritos apenas como não identificados. |
| O deploy inclui infraestrutura além do banco MySQL local. | O código tem configuração por ambiente; a documentação de arquitetura menciona execução local ou cloud. | O Compose encontrado define apenas MySQL. Não foram localizados manifests de deploy, configuração de cloud ou container da API que comprovem uma topologia adicional. |

# Direcionamento para análise

| Contexto / responsabilidade | Projeto relacionado |
| --- | --- |
| Autenticação, registro, login, JWT e papéis | `order-management-api` — `src/modules/auth`, `src/modules/users`, `src/middlewares/auth.middleware.ts` |
| Clientes e dados cadastrais | `order-management-api` — `src/modules/customers` |
| Produtos, catálogo e estoque | `order-management-api` — `src/modules/products` e `src/modules/orders` |
| Pedidos, status, transições e histórico | `order-management-api` — `src/modules/orders` |
| Rotas HTTP, bootstrap, middleware e configuração de aplicação | `order-management-api` — `src/app.ts`, `src/server.ts`, `src/routes`, `src/middlewares` |
| Persistência, schema, migrations e seed | `order-management-api` + MySQL — `prisma`, `src/config/database.ts`, `docker-compose.yml` |
| Testes de autenticação e pedidos | `order-management-api` — `tests`, `vitest.config.ts` |
| Webhooks de status, outbox, retries, DLQ e worker | Consultar `docs/PRD.md`, `docs/PRD.JSON` e `TRANSCRICAO.md` para o desenho proposto; a implementação correspondente não foi localizada |
| Visão documental e decisões arquiteturais | `docs/PRD.md`, `docs/RFC.md`, `docs/FDD.md`, `docs/TRACKER.md` e `docs/adrs/` |
