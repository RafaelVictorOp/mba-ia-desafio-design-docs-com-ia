| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| -- | --------- | ---- | ----------------- | ----- | ----------- |
| PRD-CTX-01 | docs/PRD.md | Público-alvo | Clientes B2B solicitaram receber notificações sobre mudanças de status de pedidos. | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-02 | docs/PRD.md | Contexto | Atlas Comercial, MaxDistribuição e Nova Cargo são os três clientes solicitantes. | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-03 | docs/PRD.md | Problema | Os clientes consultam repetidamente GET /orders, tornando a integração lenta e cara para eles. | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-04 | docs/PRD.md | Risco de negócio | Atlas Comercial sinalizou possível migração para concorrente se a entrega não ocorrer até o fim do trimestre. | TRANSCRICAO | [09:00] Marcos |
| PRD-GOAL-01 | docs/PRD.md | Objetivo | Para os clientes, menos de 10 segundos é considerado tempo real. | TRANSCRICAO | [09:02] Marcos |
| PRD-GOAL-02 | docs/PRD.md | Objetivo | A mudança de status e o evento elegível devem ser persistidos atomicamente; falha na outbox reverte a transação. | TRANSCRICAO | [09:40] Bruno |
| PRD-GOAL-03 | docs/PRD.md | Métrica | A reunião acordou os intervalos de retry de 1 min, 5 min, 30 min, 2 h e 12 h. | TRANSCRICAO | [09:17] Diego |
| PRD-FR-001 | docs/PRD.md | Requisito Funcional | Os webhooks são outbound: o OMS envia notificações aos clientes, sem receber webhooks deles. | TRANSCRICAO | [09:02] Marcos |
| PRD-FR-002 | docs/PRD.md | Requisito Funcional | Usuários autenticados podem cadastrar, editar, remover e listar configurações de webhook. | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-003 | docs/PRD.md | Requisito Funcional | O customer_id deve ser informado no body ou path; não é inferido diretamente do JWT. | TRANSCRICAO | [09:32] Larissa |
| PRD-FR-004 | docs/PRD.md | Requisito Funcional | Cadastro de webhook inclui URL, status de interesse e secret gerada pela plataforma, devolvida na criação. | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-005 | docs/PRD.md | Requisito Não Funcional | URL do webhook deve usar HTTPS; HTTP deve ser recusado por validação. | TRANSCRICAO | [09:23] Sofia |
| PRD-FR-006 | docs/PRD.md | Requisito Funcional | Cada endpoint pode escolher os status que deseja receber; filtrar na inserção evita eventos não elegíveis na outbox. | TRANSCRICAO | [09:34] Bruno |
| PRD-FR-007 | docs/PRD.md | Requisito Funcional | Evento elegível deve ser escrito na mesma transação SQL da mudança de status; falha de escrita causa rollback. | TRANSCRICAO | [09:40] Bruno |
| PRD-FR-008 | docs/PRD.md | Requisito Funcional | O evento usa UUID e payload renderizado como snapshot no momento da inserção. | TRANSCRICAO | [09:52] Larissa |
| PRD-FR-009 | docs/PRD.md | Contrato | Payload inclui event_id, event_type, timestamp ISO 8601, IDs/número do pedido, status anterior/novo, customer_id e campos básicos como total_cents; não inclui items. | TRANSCRICAO | [09:43] Diego |
| PRD-FR-010 | docs/PRD.md | Requisito Funcional | Worker separado consulta eventos pendentes por polling a cada 2 segundos. | TRANSCRICAO | [09:09] Diego |
| PRD-FR-011 | docs/PRD.md | Requisito Funcional | O worker deve ser processo separado da API e conectar-se ao mesmo banco. | TRANSCRICAO | [09:11] Diego |
| PRD-FR-012 | docs/PRD.md | Requisito Funcional | Processo worker usa uma instância Prisma própria, por ser outro processo Node. | TRANSCRICAO | [09:30] Bruno |
| PRD-FR-013 | docs/PRD.md | Requisito Não Funcional | Timeout da chamada HTTP do worker é 10 segundos; timeout é tratado como falha para retry. | TRANSCRICAO | [09:42] Diego |
| PRD-FR-014 | docs/PRD.md | Contrato | Headers incluem X-Event-Id, X-Signature, X-Timestamp e Content-Type application/json; X-Webhook-Id identifica o endpoint. | TRANSCRICAO | [09:44] Diego |
| PRD-FR-015 | docs/PRD.md | Requisito Não Funcional | Limite máximo do payload é 64 KB; acima disso, erro e não envio, sem truncamento. | TRANSCRICAO | [09:24] Larissa |
| PRD-FR-016 | docs/PRD.md | Requisito Funcional | Falhas de entrega seguem backoff de 1 min, 5 min, 30 min, 2 h e 12 h, com cinco tentativas acordadas na reunião. | TRANSCRICAO | [09:17] Diego |
| PRD-FR-017 | docs/PRD.md | Requisito Funcional | Após o limite de falhas, evento vai para tabela DLQ separada com payload, motivo e timestamp. | TRANSCRICAO | [09:18] Diego |
| PRD-FR-018 | docs/PRD.md | Requisito Funcional | Replay manual da DLQ usa POST /admin/webhooks/dead-letter/:id/replay e recoloca o evento na outbox pendente. | TRANSCRICAO | [09:18] Diego |
| PRD-FR-019 | docs/PRD.md | Requisito de Segurança | Replay exige role ADMIN e deve auditar quem o solicitou. | TRANSCRICAO | [09:36] Sofia |
| PRD-FR-020 | docs/PRD.md | Requisito de Segurança | Assinatura dos requests usa HMAC-SHA256 sobre o corpo. | TRANSCRICAO | [09:22] Sofia |
| PRD-FR-021 | docs/PRD.md | Requisito de Segurança | Cada endpoint de webhook deve possuir secret exclusiva, não uma secret global. | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-022 | docs/PRD.md | Requisito de Segurança | A rotação mantém a secret antiga válida em paralelo por 24 horas, após as quais expira. | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-023 | docs/PRD.md | Requisito de Entrega | A garantia é at-least-once; duplicatas podem ocorrer e cliente pode deduplicar com UUID em X-Event-Id. | TRANSCRICAO | [09:25] Diego |
| PRD-FR-024 | docs/PRD.md | Requisito Funcional | Cliente pode consultar GET /webhooks/:id/deliveries, com até 100 entregas, status, payload, resposta e tempo de resposta. | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-025 | docs/PRD.md | Requisito de Segurança | CRUD é acessível a usuário autenticado; a associação do usuário ao customer_id não vem diretamente do JWT. | TRANSCRICAO | [09:32] Larissa |
| PRD-FR-026 | docs/PRD.md | Requisito de Segurança | A reunião indica que o CRUD pode ser usado por qualquer role autenticada; a regra de vínculo entre usuário e cliente não é detalhada. | TRANSCRICAO | [09:37] Sofia |
| PRD-NFR-001 | docs/PRD.md | Requisito Não Funcional | Espera-se polling a cada 2 segundos para atender ao objetivo de tempo inferior a 10 segundos. | TRANSCRICAO | [09:09] Diego |
| PRD-NFR-002 | docs/PRD.md | Requisito Não Funcional | Padrões existentes incluem módulos, AppError, logger Pino, middleware de erros, schemas Zod e prefixo WEBHOOK_ nos erros. | TRANSCRICAO | [09:30] Larissa |
| PRD-NFR-003 | docs/PRD.md | Requisito Não Funcional | O schema Prisma existente declara MySQL como provider. | CODIGO | prisma/schema.prisma |
| PRD-NFR-004 | docs/PRD.md | Requisito Não Funcional | O logger do workspace usa Pino e configuração de redação de campos sensíveis. | CODIGO | src/shared/logger/index.ts |
| PRD-DEC-001 | docs/PRD.md | Decisão | Usar Transactional Outbox no MySQL existente em vez de Redis Streams para não adicionar infraestrutura. | TRANSCRICAO | [09:07] Diego |
| PRD-DEC-002 | docs/PRD.md | Decisão | Evitar chamada HTTP síncrona durante a mudança de status, para não bloquear pedidos por endpoint lento/indisponível. | TRANSCRICAO | [09:04] Bruno |
| PRD-DEC-003 | docs/PRD.md | Decisão | Usar polling no worker em vez de trigger para notificação de processo externo. | TRANSCRICAO | [09:09] Diego |
| PRD-DEC-004 | docs/PRD.md | Decisão | Iniciar com um worker que busca eventos pela ordem de criação; reunião ressalva ausência de ordering global. | TRANSCRICAO | [09:12] Diego |
| PRD-DEC-005 | docs/PRD.md | Decisão | Usar at-least-once e X-Event-Id em vez de exigir exactly-once. | TRANSCRICAO | [09:25] Diego |
| PRD-DEC-006 | docs/PRD.md | Decisão | Manter DLQ em tabela separada para preservar evidência de falha e apoiar diagnóstico/reprocessamento. | TRANSCRICAO | [09:18] Diego |
| PRD-DEC-007 | docs/PRD.md | Decisão | Snapshot do payload é renderizado na inserção, para refletir o estado do pedido naquele momento. | TRANSCRICAO | [09:52] Larissa |
| PRD-DEC-008 | docs/PRD.md | Decisão | Persistência do evento deve participar da mesma transação de mudança de status. | TRANSCRICAO | [09:41] Diego |
| PRD-SCOPE-001 | docs/PRD.md | Restrição de Escopo | E-mail para alertar cliente sobre falhas fica fora desta fase. | TRANSCRICAO | [09:37] Larissa |
| PRD-SCOPE-002 | docs/PRD.md | Restrição de Escopo | Dashboard visual para clientes fica fora desta fase; serão oferecidos endpoints de API. | TRANSCRICAO | [09:40] Larissa |
| PRD-SCOPE-003 | docs/PRD.md | Restrição de Escopo | Rate limiting de saída fica fora de escopo; volume será observado para avaliação futura. | TRANSCRICAO | [09:39] Larissa |
| PRD-SCOPE-004 | docs/PRD.md | Restrição de Escopo | Arquivamento de registros entregues após cerca de 30 dias fica fora da feature. | TRANSCRICAO | [09:08] Diego |
| PRD-DEP-001 | docs/PRD.md | Dependência Técnica | O projeto declara Node.js >=20 e usa TypeScript/Node no pacote da API. | CODIGO | package.json |
| PRD-DEP-002 | docs/PRD.md | Dependência Técnica | O sistema existente usa Prisma com datasource MySQL. | CODIGO | prisma/schema.prisma |
| PRD-DEP-003 | docs/PRD.md | Dependência Técnica | Middleware existente valida JWT e popula usuário autenticado com id, email e role. | CODIGO | src/middlewares/auth.middleware.ts |
| PRD-DEP-004 | docs/PRD.md | Dependência Técnica | Middleware existente fornece requireRole para autorização por role. | CODIGO | src/middlewares/auth.middleware.ts |
| PRD-DEP-005 | docs/PRD.md | Dependência Organizacional | Revisão de segurança antes do deploy deve ter pelo menos dois dias úteis reservados. | TRANSCRICAO | [09:46] Sofia |
| PRD-DEP-006 | docs/PRD.md | Integração Existente | O OrderService atual já executa mudança de status, histórico e ajuste de estoque em transação Prisma; a gravação webhook ainda é proposta, não implementação existente. | CODIGO | src/modules/orders/order.service.ts |
| PRD-RISK-001 | docs/PRD.md | Risco | Endpoint do cliente pode ficar indisponível; timeout, retry/backoff, DLQ e replay administrativo foram discutidos como manejo. | TRANSCRICAO | [09:15] Diego |
| PRD-RISK-002 | docs/PRD.md | Risco | Entrega at-least-once permite duplicatas; consumidor deve deduplicar pelo event_id. | TRANSCRICAO | [09:25] Diego |
| PRD-RISK-003 | docs/PRD.md | Risco | A reunião descreve ordering apenas no cenário single-worker e registra ausência de garantia global. | TRANSCRICAO | [09:13] Larissa |
| PRD-RISK-004 | docs/PRD.md | Risco | JWT autentica usuário, mas customer_id é informado na requisição; a regra de autorização por cliente não é especificada na discussão de CRUD. | TRANSCRICAO | [09:32] Larissa |
| PRD-RISK-005 | docs/PRD.md | Risco | Foi relatado caso anterior de vazamento de secret em logs de cliente; secret por endpoint e rotação foram discutidas. | TRANSCRICAO | [09:22] Diego |
| PRD-RISK-006 | docs/PRD.md | Risco | A equipe discutiu o cenário de 50 mudanças em um minuto e decidiu observar antes de decidir sobre rate limiting. | TRANSCRICAO | [09:38] Diego |
| PRD-PLAN-001 | docs/PRD.md | Planejamento | Estimativa da feature é de três sprints, incluindo revisão de segurança. | TRANSCRICAO | [09:47] Larissa |
| PRD-TEST-001 | docs/PRD.md | Evidência de Teste Existente | O workspace configura Vitest e possui testes API com Supertest e Prisma. | CODIGO | package.json |
| PRD-TEST-001 | docs/PRD.md | Evidência de Teste Existente | Testes de pedidos usam Supertest, fixtures/factories e consultas Prisma. | CODIGO | tests/orders.test.ts |
| RFC-CTX-001 | docs/RFC.md | Problema | Clientes B2B consultam repetidamente GET /orders para detectar mudanças de status. | TRANSCRICAO | [09:00] Marcos |
| RFC-ARCH-001 | docs/RFC.md | Decisão | Usar Transactional Outbox no MySQL existente em vez de adicionar broker nesta versão. | TRANSCRICAO | [09:06] Diego |
| RFC-ARCH-002 | docs/RFC.md | Decisão | Executar o worker em processo separado da API, conectado ao mesmo banco. | TRANSCRICAO | [09:11] Diego |
| RFC-ARCH-003 | docs/RFC.md | Requisito Não Funcional | Consultar eventos pendentes por polling a cada 2 segundos. | TRANSCRICAO | [09:09] Diego |
| RFC-REQ-001 | docs/RFC.md | Requisito Funcional | Filtrar endpoints inscritos no status de destino antes de inserir eventos na outbox. | TRANSCRICAO | [09:33] Marcos |
| RFC-RETRY-001 | docs/RFC.md | Política de Retentativa | Usar intervalos de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas. | TRANSCRICAO | [09:17] Diego |
| RFC-DLQ-001 | docs/RFC.md | Decisão | Manter DLQ separada com payload, motivo da falha e timestamp para diagnóstico/replay. | TRANSCRICAO | [09:18] Diego |
| RFC-SEC-001 | docs/RFC.md | Requisito de Segurança | Assinar o corpo dos requests com HMAC-SHA256. | TRANSCRICAO | [09:20] Sofia |
| RFC-SEC-002 | docs/RFC.md | Requisito de Segurança | Usar secret exclusiva por endpoint e suportar rotação com sobreposição anterior de 24 horas. | TRANSCRICAO | [09:21] Sofia |
| RFC-SEC-003 | docs/RFC.md | Requisito de Segurança | Exigir HTTPS para URLs de webhook e recusar HTTP. | TRANSCRICAO | [09:23] Sofia |
| RFC-DELIVERY-001 | docs/RFC.md | Semântica de Entrega | Entrega at-least-once permite duplicatas; o consumidor pode deduplicar usando UUID em X-Event-Id. | TRANSCRICAO | [09:25] Diego |
| RFC-API-001 | docs/RFC.md | Contrato | As operações da API usam JWT e recebem customer_id no body ou path, não inferido diretamente do token. | TRANSCRICAO | [09:32] Larissa |
| RFC-API-002 | docs/RFC.md | Requisito Funcional | Expor histórico com até 100 entregas, incluindo sucesso/falha, payload, resposta e tempo. | TRANSCRICAO | [09:34] Marcos |
| RFC-API-003 | docs/RFC.md | Requisito de Segurança | Restringir replay da DLQ à role ADMIN e auditar quem solicitou. | TRANSCRICAO | [09:36] Sofia |
| RFC-ATOMIC-001 | docs/RFC.md | Requisito de Integridade | Inserir o evento na mesma transação da mudança de status; falha de persistência na outbox causa rollback. | TRANSCRICAO | [09:40] Bruno |
| RFC-HTTP-001 | docs/RFC.md | Requisito Não Funcional | Tratar chamada HTTP acima de 10 segundos como falha para retry. | TRANSCRICAO | [09:42] Diego |
| RFC-CONTRACT-001 | docs/RFC.md | Contrato | O payload descreve order.status_changed com IDs/número do pedido, status anterior/novo, customer_id e campos básicos, sem items. | TRANSCRICAO | [09:43] Diego |
| RFC-CONTRACT-002 | docs/RFC.md | Contrato | Enviar X-Event-Id, X-Signature, X-Timestamp, Content-Type e X-Webhook-Id. | TRANSCRICAO | [09:44] Sofia |
| RFC-SEC-REVIEW-001 | docs/RFC.md | Dependência Organizacional | Reservar ao menos dois dias úteis para revisão de segurança antes do deploy. | TRANSCRICAO | [09:46] Sofia |
| RFC-SNAPSHOT-001 | docs/RFC.md | Decisão | Gerar UUID e snapshot do payload no momento da inserção para refletir o estado do pedido naquela mudança. | TRANSCRICAO | [09:51] Larissa |
| RFC-CODE-001 | docs/RFC.md | Evidência de Implementação Atual | changeStatus() usa transação Prisma para atualizar pedido, histórico e efeitos de estoque. | CODIGO | src/modules/orders/order.service.ts |
| RFC-CODE-002 | docs/RFC.md | Evidência de Implementação Atual | Middleware authenticate valida JWT; requireRole() autoriza pelas roles ADMIN e OPERATOR. | CODIGO | src/middlewares/auth.middleware.ts |
| RFC-CODE-003 | docs/RFC.md | Restrição Técnica | O datasource Prisma existente usa MySQL. | CODIGO | prisma/schema.prisma |
| RFC-CODE-004 | docs/RFC.md | Lacuna da Implementação Atual | Os modelos User e Customer não declaram relação de vínculo usuário-cliente no schema atual. | CODIGO | prisma/schema.prisma |
| RFC-CODE-005 | docs/RFC.md | Evidência de Implementação Atual | O logger Pino existente redige campos sensíveis configurados, incluindo authorization, cookie, password e token. | CODIGO | src/shared/logger/index.ts |
| PLATFORM-ADR-CAND-01 | docs/adrs/potential-adrs/done/PLATFORM/autenticacao-jwt-bearer.md | Decisão candidata | O middleware autentica requisições com JWT Bearer, valida usando JWT_SECRET e disponibiliza identidade/role; a configuração exige segredo mínimo de 16 caracteres e define expiração padrão de 8 horas. | CODIGO | src/middlewares/auth.middleware.ts |
| PLATFORM-ADR-CAND-01 | docs/adrs/potential-adrs/done/PLATFORM/autenticacao-jwt-bearer.md | Decisão candidata | O segredo JWT tem mínimo de 16 caracteres e JWT_EXPIRES_IN tem padrão de 8 horas. | CODIGO | src/config/env.ts |
| USERS-ADR-CAND-01 | docs/adrs/potential-adrs/done/USERS/hash-de-senhas-com-bcrypt.md | Decisão candidata | O ciclo de vida das senhas de usuários usa bcrypt para gerar hashes com fator de custo 10 antes de persistir passwordHash. | CODIGO | src/modules/users/user.service.ts |
| USERS-ADR-CAND-01 | docs/adrs/potential-adrs/done/USERS/hash-de-senhas-com-bcrypt.md | Decisão candidata | O login compara a senha informada com o hash persistido por meio de bcrypt.compare. | CODIGO | src/modules/auth/auth.service.ts |
| USERS-ADR-CAND-01 | docs/adrs/potential-adrs/done/USERS/hash-de-senhas-com-bcrypt.md | Decisão candidata | O modelo de usuário persiste o campo passwordHash. | CODIGO | prisma/schema.prisma |
| TESTS-OBS-01 | docs/adrs/potential-adrs-index.md | Evidência de código | Os testes de pedidos exercitam rotas HTTP com Supertest e consultam o Prisma para verificar efeitos persistidos. | CODIGO | tests/orders.test.ts |
| TESTS-OBS-02 | docs/adrs/potential-adrs-index.md | Evidência de código | A configuração Vitest usa setup global, desativa paralelismo entre arquivos e define um único fork. | CODIGO | vitest.config.ts |
| TESTS-OBS-03 | docs/adrs/potential-adrs-index.md | Evidência de código | O setup conecta ao Prisma e limpa dados de teste antes de cada teste. | CODIGO | tests/setup.ts |
| POT-ADR-PRODUCTS-EXPRESS | docs/adrs/potential-adrs/done/PRODUCTS/express-como-framework-http-do-modulo.md | Decisão candidata | O módulo PRODUCTS usa o Express para roteamento HTTP e handlers. | CODIGO | src/modules/products/product.routes.ts; src/modules/products/product.controller.ts |
| MAP-API-001 | docs/adrs/mapping.md | Componente de Arquitetura | O módulo API cria a aplicação Express e compõe controllers, serviços e repositórios na inicialização. | CODIGO | src/app.ts |
| MAP-API-002 | docs/adrs/mapping.md | Convenção de API | Routers de domínio são montados sob `/api/v1`, com rota de saúde separada em `/health`. | CODIGO | src/app.ts; src/routes/index.ts |
| POT-ADR-API-001 | docs/adrs/potential-adrs/done/API/express-como-framework-principal.md | Decisão Candidata | Express é o framework principal da aplicação HTTP e estrutura a composição dos routers. | CODIGO | src/app.ts; src/routes/index.ts |
| POT-ADR-API-002 | docs/adrs/potential-adrs/done/API/api-rest-versionada.md | Decisão Candidata | A interface é descrita como REST e os routers são publicados sob `/api/v1`. | CODIGO | package.json; src/app.ts; src/routes/index.ts |
| POT-ADR-PRODUCTS-REST | docs/adrs/potential-adrs/done/PRODUCTS/api-rest-para-catalogo-produtos.md | Decisão candidata | O módulo PRODUCTS expõe operações do catálogo por rotas HTTP GET, POST, PATCH e DELETE; a classificação REST é inferida dessas rotas. | CODIGO | src/modules/products/product.routes.ts |
| POT-ADR-PRODUCTS-PRISMA | docs/adrs/potential-adrs/done/PRODUCTS/prisma-como-camada-de-acesso-a-dados.md | Decisão candidata | O repositório PRODUCTS usa Prisma Client para operações de persistência de produtos. | CODIGO | src/modules/products/product.repository.ts |
| AUTH-ADR-CAND-01 | docs/adrs/potential-adrs/done/AUTH/estrategia-de-autenticacao-local-jwt.md | Decisão candidata | Registro e login são rotas públicas; /me exige autenticação. | CODIGO | src/modules/auth/auth.routes.ts |
| AUTH-ADR-CAND-02 | docs/adrs/potential-adrs/done/AUTH/estrategia-de-autenticacao-local-jwt.md | Decisão candidata | O serviço de usuários gera hashes bcrypt com 10 rounds. | CODIGO | src/modules/users/user.service.ts |
| AUTH-ADR-CAND-02 | docs/adrs/potential-adrs/done/AUTH/estrategia-de-autenticacao-local-jwt.md | Decisão candidata | O login compara a senha com o passwordHash por bcrypt. | CODIGO | src/modules/auth/auth.service.ts |
| AUTH-ADR-CAND-03 | docs/adrs/potential-adrs/done/AUTH/estrategia-de-autenticacao-local-jwt.md | Decisão candidata | O token é JWT assinado com sub, email e role e usa a expiração configurada. | CODIGO | src/modules/auth/auth.service.ts |
| AUTH-ADR-CAND-04 | docs/adrs/potential-adrs/done/AUTH/estrategia-de-autenticacao-local-jwt.md | Decisão candidata | O middleware exige esquema Bearer, verifica JWT com JWT_SECRET e popula a identidade autenticada. | CODIGO | src/middlewares/auth.middleware.ts |
| AUTH-ADR-CAND-05 | docs/adrs/potential-adrs/done/AUTH/estrategia-de-autenticacao-local-jwt.md | Decisão candidata | A autorização por role reconhece ADMIN e OPERATOR e rejeita roles não permitidas. | CODIGO | src/middlewares/auth.middleware.ts |
| AUTH-ADR-CAND-06 | docs/adrs/potential-adrs/done/AUTH/estrategia-de-autenticacao-local-jwt.md | Configuração de segurança | JWT_SECRET exige pelo menos 16 caracteres e JWT_EXPIRES_IN tem padrão 8h. | CODIGO | src/config/env.ts |
| AUTH-ADR-CAND-07 | docs/adrs/potential-adrs/done/AUTH/estrategia-de-autenticacao-local-jwt.md | Proteção de dados | A representação pública do usuário não inclui passwordHash. | CODIGO | src/modules/users/user.service.ts |
| AUTH-ADR-CAND-07 | docs/adrs/potential-adrs/done/AUTH/estrategia-de-autenticacao-local-jwt.md | Proteção de dados | O teste de registro verifica que a resposta não contém passwordHash nem password. | CODIGO | tests/auth.test.ts |
| PRODUCTS-OBS-01 | docs/adrs/potential-adrs-index.md | Decisão avaliada e descartada | O preço de produto é aceito como inteiro não negativo em centavos e enviado à persistência pelo campo priceCents. | CODIGO | src/modules/products/product.schemas.ts; src/modules/products/product.service.ts |
| ADR-CAND-ORDERS-PRISMA | docs/adrs/potential-adrs/done/ORDERS/prisma-orm-orders.md | Decisão candidata | O módulo ORDERS usa Prisma como ORM e camada de acesso a dados. | CODIGO | src/modules/orders/order.repository.ts |
| ADR-CAND-ORDERS-EXPRESS | docs/adrs/potential-adrs/done/ORDERS/express-framework-orders.md | Decisão candidata | O módulo ORDERS define rotas e handlers HTTP com Express. | CODIGO | src/modules/orders/order.routes.ts |
| ADR-CAND-ORDERS-REST | docs/adrs/potential-adrs/done/ORDERS/api-rest-orders.md | Decisão candidata | As rotas de ORDERS expõem operações HTTP em estilo REST, conforme métodos e recursos observados. | CODIGO | src/modules/orders/order.routes.ts |
| DATA-ADR-CAND-001 | docs/adrs/potential-adrs/done/DATA/banco-relacional-mysql.md | Decisão candidata | MySQL 8.0 está configurado como serviço de banco relacional para a persistência do sistema. | CODIGO | docker-compose.yml |
| DATA-ADR-CAND-001 | docs/adrs/potential-adrs/done/DATA/banco-relacional-mysql.md | Decisão candidata | O datasource no schema Prisma declara MySQL como provider. | CODIGO | prisma/schema.prisma |
| DATA-ADR-CAND-002 | docs/adrs/potential-adrs/done/DATA/orm-prisma.md | Decisão candidata | Prisma é usado como ORM, cliente e camada de acesso ao banco. | CODIGO | src/config/database.ts |
| DATA-ADR-CAND-002 | docs/adrs/potential-adrs/done/DATA/orm-prisma.md | Decisão candidata | A aplicação fixa as dependências Prisma e Prisma Client em 5.22.0. | CODIGO | package.json |
| ADR-CUSTOMERS-01 | docs/adrs/potential-adrs/done/CUSTOMERS/banco-de-dados-mysql.md | Decisão candidata | CUSTOMERS usa o serviço MySQL declarado no Compose e datasource MySQL do Prisma para persistência. | CODIGO | docker-compose.yml |
| ADR-CUSTOMERS-02 | docs/adrs/potential-adrs/done/CUSTOMERS/framework-express.md | Decisão candidata | CUSTOMERS registra suas rotas usando Router do Express e handlers HTTP tipados. | CODIGO | src/modules/customers/customer.routes.ts |
| ADR-CUSTOMERS-03 | docs/adrs/potential-adrs/done/CUSTOMERS/camada-de-acesso-a-dados-prisma.md | Decisão candidata | CustomerRepository executa operações de persistência usando PrismaClient e Prisma Customer. | CODIGO | src/modules/customers/customer.repository.ts |
| ADR-CUSTOMERS-04 | docs/adrs/potential-adrs/done/CUSTOMERS/api-rest-clientes.md | Decisão candidata | CUSTOMERS expõe operações HTTP por GET, POST, PATCH e DELETE em rotas de clientes. | CODIGO | src/modules/customers/customer.routes.ts |
| ADR-001 | docs/adrs/generated/PLATFORM/ADR-001-autenticacao-de-requisicoes-com-jwt.md | Decisão | A API autentica requisições com JWT Bearer e aplica autorização por roles. | CODIGO | src/middlewares/auth.middleware.ts |
| ADR-002 | docs/adrs/generated/USERS/ADR-002-hash-de-senhas-com-bcrypt.md | Decisão | Senhas de usuários são protegidas com hashes bcrypt antes da persistência e comparadas no login. | CODIGO | src/modules/users/user.service.ts; src/modules/auth/auth.service.ts |
| ADR-003 | docs/adrs/generated/PRODUCTS/ADR-003-express-como-framework-http.md | Decisão | O módulo PRODUCTS registra suas rotas HTTP com Express. | CODIGO | src/modules/products/product.routes.ts |
| ADR-004 | docs/adrs/generated/PRODUCTS/ADR-004-api-rest-para-catalogo-de-produtos.md | Decisão | O catálogo de produtos expõe operações HTTP orientadas a recursos. | CODIGO | src/modules/products/product.routes.ts |
| ADR-005 | docs/adrs/generated/PRODUCTS/ADR-005-prisma-como-camada-de-dados-de-produtos.md | Decisão | O acesso de dados de produtos usa Prisma. | CODIGO | src/modules/products/product.repository.ts |
| ADR-006 | docs/adrs/generated/AUTH/ADR-006-estrategia-de-autenticacao-local-com-jwt.md | Decisão | A autenticação local verifica credenciais e emite JWT para sessões autenticadas. | CODIGO | src/modules/auth/auth.service.ts |
| ADR-007 | docs/adrs/generated/ORDERS/ADR-007-prisma-como-orm-em-pedidos.md | Decisão | O acesso de dados de pedidos usa Prisma. | CODIGO | src/modules/orders/order.repository.ts |
| ADR-008 | docs/adrs/generated/ORDERS/ADR-008-express-como-framework-em-pedidos.md | Decisão | O módulo ORDERS registra suas rotas HTTP com Express. | CODIGO | src/modules/orders/order.routes.ts |
| ADR-009 | docs/adrs/generated/ORDERS/ADR-009-api-rest-em-pedidos.md | Decisão | O módulo ORDERS expõe operações HTTP orientadas a recursos. | CODIGO | src/modules/orders/order.routes.ts |
| ADR-010 | docs/adrs/generated/API/ADR-010-express-como-framework-principal.md | Decisão | A aplicação HTTP é composta com Express e routers de domínio. | CODIGO | src/app.ts |
| ADR-011 | docs/adrs/generated/API/ADR-011-api-rest-versionada.md | Decisão | A API HTTP publica os routers de domínio sob o prefixo versionado /api/v1. | CODIGO | src/app.ts |
| ADR-012 | docs/adrs/generated/CUSTOMERS/ADR-012-mysql-para-dados-de-clientes.md | Decisão | O datasource da aplicação usa MySQL para os dados relacionais de clientes. | CODIGO | prisma/schema.prisma |
| ADR-013 | docs/adrs/generated/CUSTOMERS/ADR-013-express-para-api-de-clientes.md | Decisão | O módulo CUSTOMERS registra suas rotas HTTP com Express. | CODIGO | src/modules/customers/customer.routes.ts |
| ADR-014 | docs/adrs/generated/CUSTOMERS/ADR-014-prisma-para-dados-de-clientes.md | Decisão | O acesso de dados de clientes usa Prisma. | CODIGO | src/modules/customers/customer.repository.ts |
| ADR-015 | docs/adrs/generated/CUSTOMERS/ADR-015-api-rest-para-clientes.md | Decisão | O módulo CUSTOMERS expõe operações HTTP orientadas a recursos. | CODIGO | src/modules/customers/customer.routes.ts |
| ADR-016 | docs/adrs/generated/DATA/ADR-016-mysql-como-banco-relacional.md | Decisão | O datasource Prisma e o ambiente de desenvolvimento configuram MySQL. | CODIGO | prisma/schema.prisma; docker-compose.yml |
| ADR-017 | docs/adrs/generated/DATA/ADR-017-prisma-como-orm.md | Decisão | A aplicação usa Prisma Client para o acesso ao banco. | CODIGO | src/config/database.ts |
| ADR-018 | docs/adrs/generated/WEBHOOKS/ADR-018-outbox-mysql-worker-polling.md | Decisão | Gravar eventos de status na outbox MySQL na transação do pedido e processá-los por worker separado com polling. | TRANSCRICAO | [09:06] Diego |
| ADR-018 | docs/adrs/generated/WEBHOOKS/ADR-018-outbox-mysql-worker-polling.md | Decisão | Gravar eventos de status na outbox MySQL na transação do pedido e processá-los por worker separado com polling. | TRANSCRICAO | [09:09] Diego |
| ADR-018 | docs/adrs/generated/WEBHOOKS/ADR-018-outbox-mysql-worker-polling.md | Decisão | Gravar eventos de status na outbox MySQL na transação do pedido e processá-los por worker separado com polling. | TRANSCRICAO | [09:11] Diego |
| ADR-019 | docs/adrs/generated/WEBHOOKS/ADR-019-assinatura-hmac-por-endpoint.md | Decisão | Assinar o corpo outbound com HMAC-SHA256, secret exclusiva por endpoint e transporte HTTPS. | TRANSCRICAO | [09:20] Sofia |
| ADR-019 | docs/adrs/generated/WEBHOOKS/ADR-019-assinatura-hmac-por-endpoint.md | Decisão | Assinar o corpo outbound com HMAC-SHA256, secret exclusiva por endpoint e transporte HTTPS. | TRANSCRICAO | [09:21] Sofia |
| ADR-019 | docs/adrs/generated/WEBHOOKS/ADR-019-assinatura-hmac-por-endpoint.md | Decisão | Assinar o corpo outbound com HMAC-SHA256, secret exclusiva por endpoint e transporte HTTPS. | TRANSCRICAO | [09:23] Sofia |
| ADR-020 | docs/adrs/generated/WEBHOOKS/ADR-020-entrega-at-least-once-com-event-id.md | Decisão | Usar entrega at-least-once e `X-Event-Id` para que o consumidor deduplique eventos repetidos. | TRANSCRICAO | [09:24] Diego |
| ADR-020 | docs/adrs/generated/WEBHOOKS/ADR-020-entrega-at-least-once-com-event-id.md | Decisão | Usar entrega at-least-once e `X-Event-Id` para que o consumidor deduplique eventos repetidos. | TRANSCRICAO | [09:25] Diego |
