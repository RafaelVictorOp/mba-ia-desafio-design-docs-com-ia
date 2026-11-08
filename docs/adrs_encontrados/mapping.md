# Mapeamento da arquitetura da base de código

## Visão geral do projeto

O projeto é uma API REST para um Order Management System (OMS), implementada em Node.js e TypeScript. O sistema oferece autenticação e operações sobre usuários, clientes, produtos e pedidos. A aplicação HTTP é montada em Express; o acesso relacional é feito por Prisma sobre MySQL.

O código executável analisado representa o estado atual do OMS. Webhooks de saída são uma proposta futura descrita no PRD e no RFC, ambos com status `Draft`; não há módulo, worker ou modelos de persistência de webhook no código consultado.

## Stack tecnológica

- **Runtime e linguagem:** Node.js 20 ou superior, TypeScript 5.6 e módulos ES.
- **API HTTP:** Express 4.21, com routers sob `/api/v1` e endpoint de saúde em `/health`.
- **Persistência:** MySQL 8.0 em Docker Compose; Prisma 5.22 e Prisma Client para schema, migrations e acesso a dados.
- **Autenticação e segurança:** JWT (`jsonwebtoken`) para autenticação Bearer; `bcrypt` para hashes de senha; autorização por roles `ADMIN` e `OPERATOR`.
- **Validação:** Zod, incluindo validação da configuração de ambiente e dos contratos HTTP.
- **Logging:** Pino e `pino-http`, com redação de campos sensíveis configurada no logger.
- **Testes:** Vitest e Supertest; os testes usam Prisma e preparam/limpam dados no banco.
- **Ferramentas de desenvolvimento:** `tsx`, ESLint e Prettier.
- **Infraestrutura observada:** Compose declara apenas o MySQL. Não foram identificados broker, cache, serviço cloud, Kubernetes ou infraestrutura como código nos arquivos consultados.

## Notas de contexto

**Arquivos consultados:** `docs/PRD.md`, `docs/RFC.md`, `docs/FDD.md`, `TRANSCRICAO.md`, `docker-compose.yml`, `package.json`, `tsconfig.json`, `src/app.ts`, `src/server.ts`, `src/routes/index.ts`, `src/config/*`, `src/middlewares/*`, `src/modules/*`, `src/shared/*`, `prisma/schema.prisma`, migration inicial e `tests/*`.

**Principais descobertas:**

- Os documentos de produto e design descrevem webhooks outbound com Transactional Outbox em MySQL e um worker separado. O PRD e o RFC estão em `Draft`; a transcrição registra a conversa de arquitetura. Esses documentos não comprovam que a feature foi implementada.
- O código atual confirma Express, Prisma, MySQL, JWT, validação Zod, erros de aplicação centralizados e logging Pino.
- O schema Prisma contém usuários, clientes, produtos, pedidos, itens, histórico de status e sequência de numeração. Não contém configuração de webhook, outbox, histórico de entregas ou DLQ.
- `package.json` não declara comando de worker; o ponto de entrada da aplicação é `src/server.ts`.
- `docs/FDD.md` e o `README.md` consultado contêm modelos/instruções com campos ainda não preenchidos, e não devem ser tratados como especificação concluída do sistema.
- Não foram consultados documentos em um diretório de contexto separado nem encontrados ADRs formais neste diretório.

## Módulos do sistema

### Índice de módulos

1. **API** - composição da aplicação Express, inicialização HTTP e registro das rotas.
2. **AUTH** - registro, login, consulta da identidade autenticada e emissão/verificação de JWT.
3. **USERS** - administração e persistência de usuários do OMS.
4. **CUSTOMERS** - cadastro e consulta de clientes associados a pedidos.
5. **PRODUCTS** - catálogo de produtos e dados de estoque.
6. **ORDERS** - criação e consulta de pedidos, transições de status, histórico e efeitos no estoque.
7. **DATA** - schema relacional, migrations, seed e cliente Prisma.
8. **PLATFORM** - middlewares e componentes transversais de validação, erros, HTTP e logging.
9. **TESTS** - testes de integração HTTP e dados auxiliares de teste.

### API: Composição e transporte HTTP

**Finalidade:** criar a aplicação Express, compor controllers e serviços com Prisma, registrar routers e iniciar o servidor.

**Localização:** `src/app.ts`, `src/server.ts`, `src/routes/*`

**Componentes principais:** `buildApp`, `buildControllers`, `buildApiRouter` e bootstrap do servidor.

**Tecnologias:** Express, TypeScript, Prisma Client.

**Dependências:** AUTH, USERS, CUSTOMERS, PRODUCTS, ORDERS, DATA e PLATFORM.

**Padrões:** composição explícita das dependências; routers de domínio montados sob `/api/v1`; tratamento de rota não encontrada e erros no final do pipeline.

**Arquivos relevantes:** `src/app.ts`, `src/server.ts`, `src/routes/index.ts`.

**Porte:** pequeno - 3 arquivos principais.

### AUTH: Autenticação

**Finalidade:** registrar e autenticar usuários, fornecer a identidade do usuário corrente e validar tokens Bearer.

**Localização:** `src/modules/auth/*`, `src/middlewares/auth.middleware.ts`

**Componentes principais:** controller, service, schemas e rotas de autenticação; middleware `authenticate` e helper `requireRole`.

**Tecnologias:** Express, JWT, bcrypt, Zod e Prisma via repositório de usuários.

**Dependências:** USERS, DATA e PLATFORM.

**Padrões:** rotas `/register` e `/login` públicas; `/me` protegido por JWT; roles `ADMIN` e `OPERATOR` disponíveis para autorização.

**Arquivos relevantes:** `src/modules/auth/auth.controller.ts`, `src/modules/auth/auth.routes.ts`, `src/modules/auth/auth.schemas.ts`, `src/modules/auth/auth.service.ts`, `src/middlewares/auth.middleware.ts`.

**Porte:** pequeno - 4 arquivos no módulo, além do middleware compartilhado.

### USERS: Usuários

**Finalidade:** persistir e administrar usuários que operam o OMS; é também a base de dados usada pelo fluxo de autenticação.

**Localização:** `src/modules/users/*`

**Componentes principais:** controller, service, repository, routes e schemas.

**Tecnologias:** Express, Prisma e Zod.

**Dependências:** DATA e PLATFORM; AUTH usa o repositório e os serviços de usuário.

**Padrões:** separação controller/service/repository e validação de entrada nas rotas.

**Arquivos relevantes:** `src/modules/users/user.controller.ts`, `src/modules/users/user.service.ts`, `src/modules/users/user.repository.ts`, `src/modules/users/user.routes.ts`, `src/modules/users/user.schemas.ts`.

**Porte:** médio - 5 arquivos.

### CUSTOMERS: Clientes

**Finalidade:** manter os dados dos clientes que possuem pedidos.

**Localização:** `src/modules/customers/*`

**Componentes principais:** controller, service, repository, routes e schemas.

**Tecnologias:** Express, Prisma e Zod.

**Dependências:** DATA e PLATFORM; ORDERS referencia clientes.

**Padrões:** separação controller/service/repository e validação de entrada nas rotas.

**Arquivos relevantes:** `src/modules/customers/customer.controller.ts`, `src/modules/customers/customer.service.ts`, `src/modules/customers/customer.repository.ts`, `src/modules/customers/customer.routes.ts`, `src/modules/customers/customer.schemas.ts`.

**Porte:** médio - 5 arquivos.

### PRODUCTS: Produtos e estoque

**Finalidade:** manter catálogo, preço, estado ativo e quantidade disponível dos produtos.

**Localização:** `src/modules/products/*`

**Componentes principais:** controller, service, repository, routes e schemas.

**Tecnologias:** Express, Prisma e Zod.

**Dependências:** DATA e PLATFORM; ORDERS lê e altera o estoque durante transições de pedido.

**Padrões:** separação controller/service/repository; quantias monetárias representadas em centavos no modelo persistido.

**Arquivos relevantes:** `src/modules/products/product.controller.ts`, `src/modules/products/product.service.ts`, `src/modules/products/product.repository.ts`, `src/modules/products/product.routes.ts`, `src/modules/products/product.schemas.ts`.

**Porte:** médio - 5 arquivos.

### ORDERS: Pedidos

**Finalidade:** criar e consultar pedidos, validar transições de status, manter histórico e aplicar os efeitos de estoque correspondentes.

**Localização:** `src/modules/orders/*`

**Componentes principais:** controller, service, repository, routes, schemas e regras de transição em `order.status.ts`.

**Tecnologias:** Express, Prisma, Zod e UUIDs definidos no modelo.

**Dependências:** CUSTOMERS, PRODUCTS, USERS (autoria e histórico), DATA e PLATFORM.

**Padrões:** operações de criação e mudança de status em transações Prisma; histórico persistido junto com as alterações do pedido; regras de estoque encapsuladas no serviço.

**Arquivos relevantes:** `src/modules/orders/order.controller.ts`, `src/modules/orders/order.service.ts`, `src/modules/orders/order.repository.ts`, `src/modules/orders/order.routes.ts`, `src/modules/orders/order.schemas.ts`, `src/modules/orders/order.status.ts`.

**Porte:** médio - 6 arquivos.

### DATA: Persistência

**Finalidade:** definir o modelo relacional e disponibilizar conexão Prisma para a aplicação.

**Localização:** `prisma/*`, `src/config/database.ts`

**Componentes principais:** schema Prisma, migration inicial, seed e fábrica/instância de `PrismaClient`.

**Tecnologias:** MySQL 8.0, Prisma 5.22.

**Dependências:** usada por todos os módulos de domínio.

**Padrões:** relações e índices declarados no schema; migration versionada; cliente Prisma compartilhado dentro do processo da API.

**Arquivos relevantes:** `prisma/schema.prisma`, `prisma/migrations/20260519182739_init/migration.sql`, `prisma/migrations/migration_lock.toml`, `prisma/seed.ts`, `src/config/database.ts`.

**Porte:** médio - 5 arquivos listados, sem contar configurações de ambiente.

### PLATFORM: Capacidades transversais

**Finalidade:** fornecer validação HTTP, autenticação/autorização reutilizável, tratamento de erros, respostas e logging.

**Localização:** `src/middlewares/*`, `src/shared/*`, `src/config/env.ts`

**Componentes principais:** middleware de validação, autenticação, erros e request logging; `AppError` e erros HTTP; formato de resposta paginada; logger Pino; validação de variáveis de ambiente.

**Tecnologias:** Express, Zod, Pino, JWT.

**Dependências:** usada pela API e pelos módulos de domínio.

**Padrões:** erros de aplicação com código/status/detalhes; middleware centralizado; logging estruturado com redação de credenciais e senhas.

**Arquivos relevantes:** `src/middlewares/auth.middleware.ts`, `src/middlewares/error.middleware.ts`, `src/middlewares/request-logger.middleware.ts`, `src/middlewares/validate.middleware.ts`, `src/shared/errors/*`, `src/shared/http/response.ts`, `src/shared/logger/index.ts`, `src/config/env.ts`.

**Porte:** médio - 10 arquivos principais, contando os três arquivos de erros compartilhados.

### TESTS: Verificação automatizada

**Finalidade:** validar os fluxos de autenticação e pedidos por requisições HTTP e verificar os efeitos persistidos.

**Localização:** `tests/*`

**Componentes principais:** testes de autenticação e pedidos, setup de conexão/limpeza do banco e factories de dados para testes.

**Tecnologias:** Vitest, Supertest e Prisma.

**Dependências:** API, DATA e configuração de teste.

**Padrões:** testes de integração com persistência MySQL; o setup conecta ao banco e limpa as tabelas entre testes.

**Arquivos relevantes:** `tests/auth.test.ts`, `tests/orders.test.ts`, `tests/setup.ts`, `tests/helpers/factories.ts`.

**Porte:** pequeno - 4 arquivos principais.

## Responsabilidades transversais

- **Autenticação e autorização:** JWT é validado em middleware; `requireRole` verifica roles. A autorização é aplicada às rotas de domínio conforme definido em cada router.
- **Validação:** schemas Zod são aplicados por middleware às entradas HTTP; variáveis de ambiente também são validadas na inicialização.
- **Erros HTTP:** erros de domínio/aplicação seguem `AppError` e são convertidos em respostas por middleware centralizado.
- **Logging:** Pino fornece logging estruturado; request logging é instalado na aplicação e o logger mascara campos sensíveis configurados.
- **Persistência:** os módulos usam Prisma Client; mudanças de pedidos e histórico são coordenadas em transações. MySQL é a única infraestrutura externa identificada no Compose.
- **Testes:** testes atuais cobrem autenticação e pedidos, incluindo validação, transições de status e atualização de estoque; dependem de conexão com banco.
- **Webhooks propostos:** o RFC propõe adicionar um módulo de webhooks, tabelas de outbox/DLQ e um processo worker separado. Isso ainda não aparece no `package.json`, em `src/` ou no schema Prisma atual; permanece fora do mapa do sistema implementado.