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
| PRD-FR-001 | docs/PRD.md | Requisito Funcional | Os webhooks são outbound: o OMS envia notificações aos clientes, sem receber webhooks deles. | TRANSCRICAO | [09:02] Sofia |
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
| PRD-SCOPE-005 | docs/PRD.md | Restrição de Escopo | Webhooks inbound, isto é, enviados pelo cliente para a plataforma, ficam fora do escopo; a feature é somente outbound. | TRANSCRICAO | [09:02] Sofia |
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
| PRD-AC-001 | docs/PRD.md | Critério de Aceitação | Mudança com endpoint elegível confirma pedido, histórico, estoque e evento na mesma transação; falha na outbox reverte tudo e nenhum evento é criado sem endpoint elegível. | TRANSCRICAO | [09:40] Bruno |
| PRD-AC-002 | docs/PRD.md | Critério de Aceitação | Endpoint HTTPS e status válidos são cadastrados por usuário autenticado e a plataforma cria a configuração e devolve secret exclusiva. | TRANSCRICAO | [09:31] Marcos |
| PRD-AC-003 | docs/PRD.md | Critério de Aceitação | Worker faz polling a cada 2 segundos, entrega por HTTPS com timeout de 10 segundos e limita payload a 64 KB; menos de 10 segundos é objetivo, não garantia. | TRANSCRICAO | [09:42] Diego |
| PRD-AC-004 | docs/PRD.md | Critério de Aceitação | Após a tentativa inicial, são feitas até cinco retentativas nos intervalos acordados e, após falha final, o evento é preservado na DLQ para replay. | TRANSCRICAO | [09:17] Diego |
| PRD-AC-005 | docs/PRD.md | Critério de Aceitação | Endpoint usa secret exclusiva, HMAC-SHA256 e rotação com secret anterior válida por 24 horas. | TRANSCRICAO | [09:22] Sofia |
| PRD-AC-006 | docs/PRD.md | Critério de Aceitação | Usuários autenticados listam, editam e removem configurações; URLs usam HTTPS e remoção não gera novos eventos. | TRANSCRICAO | [09:33] Bruno |
| PRD-AC-007 | docs/PRD.md | Critério de Aceitação | GET /webhooks/:id/deliveries retorna até 100 entregas com diagnóstico e restringe o acesso ao cliente autorizado. | TRANSCRICAO | [09:34] Marcos |
| PRD-AC-008 | docs/PRD.md | Critério de Aceitação | POST /admin/webhooks/dead-letter/:id/replay exige ADMIN, recoloca o evento na outbox e registra o administrador solicitante. | TRANSCRICAO | [09:36] Sofia |
| PRD-AC-009 | docs/PRD.md | Critério de Aceitação | Worker executa em processo distinto da API, usa PrismaClient próprio e opera como instância única na primeira versão. | TRANSCRICAO | [09:11] Diego |
| PRD-AC-010 | docs/PRD.md | Critério de Aceitação | Somente endpoints ativos inscritos no status de destino recebem evento na outbox. | TRANSCRICAO | [09:34] Diego |
| PRD-AC-011 | docs/PRD.md | Critério de Aceitação | Payload JSON contém event_id, event_type, timestamp ISO 8601, dados do pedido e customer_id, sem itens; headers identificam evento, webhook, assinatura e timestamp. | TRANSCRICAO | [09:43] Diego |
| PRD-AC-012 | docs/PRD.md | Critério de Aceitação | A entrega é at-least-once e usa X-Event-Id estável para deduplicação pelo consumidor. | TRANSCRICAO | [09:25] Diego |
| PRD-AC-013 | docs/PRD.md | Critério de Aceitação | Payload acima de 64 KB é reconhecido como erro, não enviado e não truncado. | TRANSCRICAO | [09:24] Larissa |
| PRD-AC-014 | docs/PRD.md | Critério de Aceitação | Worker único processa eventos pela ordem de criação, sem garantia absoluta de ordenação por pedido ou global. | TRANSCRICAO | [09:13] Larissa |
| RFC-CTX-001 | docs/RFC.md | Problema | Clientes B2B consultam repetidamente GET /orders para detectar mudanças de status. | TRANSCRICAO | [09:00] Marcos |
| RFC-ARCH-001 | docs/RFC.md | Decisão | Usar Transactional Outbox no MySQL existente em vez de adicionar broker nesta versão. | TRANSCRICAO | [09:07] Diego |
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
| RFC-SCOPE-001 | docs/RFC.md | Restrição de Escopo | Receber webhooks inbound de clientes não faz parte do escopo; o projeto é outbound. | TRANSCRICAO | [09:02] Sofia |
| RFC-SCOPE-002 | docs/RFC.md | Restrição de Escopo | E-mail para alertar clientes sobre falhas fica fora desta fase e pode ser avaliado após medir impacto. | TRANSCRICAO | [09:37] Larissa |
| RFC-SCOPE-003 | docs/RFC.md | Restrição de Escopo | Rate limiting de saída não será implementado nesta fase; o volume deve ser observado. | TRANSCRICAO | [09:38] Diego |
| RFC-SCOPE-004 | docs/RFC.md | Restrição de Escopo | Dashboard ou painel visual para clientes fica fora desta fase; o frontend é projeto separado e o recurso será exposto por API. | TRANSCRICAO | [09:39] Marcos |
| RFC-SCOPE-005 | docs/RFC.md | Restrição de Escopo | Arquivamento de registros entregues após 30 dias fica fora desta feature. | TRANSCRICAO | [09:08] Diego |
| RFC-SCOPE-006 | docs/RFC.md | Restrição de Escopo | Múltiplos workers paralelos, ordenação global e exactly-once ficam fora da primeira versão. | TRANSCRICAO | [09:12] Diego |
| RFC-AUTH-001 | docs/RFC.md | Lacuna de Autorização | A transcrição não define relação ou autorização usuário-cliente; customer_id vem da requisição e não é inferido do JWT. | TRANSCRICAO | [09:32] Larissa |
| RFC-ORDER-001 | docs/RFC.md | Requisito Não Funcional | O worker único processa eventos por created_at, mas falhas e retries podem alterar a ordem efetiva e não há garantia global. | TRANSCRICAO | [09:12] Diego |
| RFC-PAYLOAD-001 | docs/RFC.md | Contrato | Payload acima de 64 KB deve ser rejeitado sem envio, truncamento ou fragmentação. | TRANSCRICAO | [09:24] Larissa |
| RFC-CLIENT-001 | docs/RFC.md | Dependência Externa | Endpoint HTTPS fornecido pelo cliente é variável em disponibilidade e pode ficar lento ou indisponível. | TRANSCRICAO | [09:04] Bruno |
| RFC-PLAN-001 | docs/RFC.md | Planejamento | A reunião estimou três sprints, incluindo a revisão de segurança antes do deploy. | TRANSCRICAO | [09:46] Larissa |
| RFC-CODE-001 | docs/RFC.md | Evidência de Implementação Atual | changeStatus() usa transação Prisma para atualizar pedido, histórico e efeitos de estoque. | CODIGO | src/modules/orders/order.service.ts |
| RFC-CODE-002 | docs/RFC.md | Evidência de Implementação Atual | Middleware authenticate valida JWT; requireRole() autoriza pelas roles ADMIN e OPERATOR. | CODIGO | src/middlewares/auth.middleware.ts |
| RFC-CODE-003 | docs/RFC.md | Restrição Técnica | O datasource Prisma existente usa MySQL. | CODIGO | prisma/schema.prisma |
| RFC-CODE-004 | docs/RFC.md | Lacuna da Implementação Atual | Os modelos User e Customer não declaram relação de vínculo usuário-cliente no schema atual. | CODIGO | prisma/schema.prisma |
| RFC-CODE-005 | docs/RFC.md | Evidência de Implementação Atual | O logger Pino existente redige campos sensíveis configurados, incluindo authorization, cookie, password e token. | CODIGO | src/shared/logger/index.ts |
| ADR-018 | docs/adrs/ADR-018-outbox-mysql-worker-polling.md | Decisão | Registrar o evento na mesma transação SQL da mudança de status usando Transactional Outbox. | TRANSCRICAO | [09:06] Diego |
| ADR-018 | docs/adrs/ADR-018-outbox-mysql-worker-polling.md | Decisão | Usar o MySQL existente para a outbox em vez de adicionar infraestrutura de mensageria. | TRANSCRICAO | [09:07] Diego |
| ADR-018-NFR-001 | docs/adrs/ADR-018-outbox-mysql-worker-polling.md | Requisito Não Funcional | Consultar eventos pendentes por polling a cada 2 segundos. | TRANSCRICAO | [09:09] Diego |
| ADR-018-DEC-001 | docs/adrs/ADR-018-outbox-mysql-worker-polling.md | Decisão | Executar o worker em processo separado da API, conectado ao mesmo banco. | TRANSCRICAO | [09:11] Diego |
| ADR-019 | docs/adrs/ADR-019-assinatura-hmac-por-endpoint.md | Decisão | Assinar o corpo do request outbound com HMAC-SHA256. | TRANSCRICAO | [09:20] Sofia |
| ADR-019-SEC-001 | docs/adrs/ADR-019-assinatura-hmac-por-endpoint.md | Requisito de Segurança | Usar uma secret exclusiva para cada endpoint de webhook. | TRANSCRICAO | [09:21] Sofia |
| ADR-019-SEC-002 | docs/adrs/ADR-019-assinatura-hmac-por-endpoint.md | Requisito de Segurança | Exigir HTTPS para endpoints de webhook e recusar URLs HTTP. | TRANSCRICAO | [09:23] Sofia |
| ADR-019-TRANS-003 | docs/adrs/ADR-019-assinatura-hmac-por-endpoint.md | Requisito de Segurança | Após a rotação, a secret anterior permanece válida em paralelo por 24 horas. | TRANSCRICAO | [09:21] Sofia |
| ADR-019-TRANS-004 | docs/adrs/ADR-019-assinatura-hmac-por-endpoint.md | Contrato | A plataforma gera uma secret exclusiva por endpoint e a devolve na criação do webhook. | TRANSCRICAO | [09:31] Marcos |
| ADR-019-CODE-001 | docs/adrs/ADR-019-assinatura-hmac-por-endpoint.md | Lacuna da Implementação Atual | O schema Prisma não contém modelos de webhook nem armazenamento de secrets de webhook. | CODIGO | prisma/schema.prisma |
| ADR-019-CODE-002 | docs/adrs/ADR-019-assinatura-hmac-por-endpoint.md | Mecanismo de Autorização | O middleware `requireRole` restringe acesso às roles configuradas para a rota. | CODIGO | src/middlewares/auth.middleware.ts |
| ADR-020 | docs/adrs/ADR-020-entrega-at-least-once-com-event-id.md | Decisão | Usar entrega at-least-once e UUID estável em `X-Event-Id` para deduplicação pelo consumidor. | TRANSCRICAO | [09:24] Diego |
| ADR-020 | docs/adrs/ADR-020-entrega-at-least-once-com-event-id.md | Decisão | Usar entrega at-least-once e UUID estável em `X-Event-Id` para deduplicação pelo consumidor. | TRANSCRICAO | [09:25] Diego |
| ADR-021-CODE-001 | docs/adrs/ADR-021-autorizacao-por-vinculo-usuario-cliente.md | Lacuna da Implementação Atual | O schema atual não implementa uma relação `User`–`Customer` nem `CustomerMembership`; o modelo descrito no ADR é proposta. | CODIGO | prisma/schema.prisma |
| ADR-022-CODE-001 | docs/adrs/ADR-022-soft-delete-e-cancelamento-de-entregas.md | Lacuna da Implementação Atual | O schema Prisma não contém entidades de webhook e os scripts do pacote não declaram comando de worker; a semântica de despacho no ADR é proposta, não comportamento implementado. | CODIGO | prisma/schema.prisma; package.json |
| ADR-023-CODE-001 | docs/adrs/ADR-023-classificar-payload-excedente-na-criacao.md | Lacuna da Implementação Atual | O schema Prisma consultado não contém modelos de webhook, outbox ou DLQ; a classificação de payload excedente e sua persistência descritas no ADR ainda são propostas, não implementação. | CODIGO | prisma/schema.prisma |
| ADR-025-CODE-001 | docs/adrs/ADR-025-identidade-de-evento-e-tentativa-no-replay.md | Lacuna da Implementação Atual | A identidade por tentativa e o replay de webhook descritos no ADR ainda não estão implementados: o schema não define entidades de webhook, o fluxo de mudança de status não grava eventos e não há comando de worker. | CODIGO | prisma/schema.prisma; src/modules/orders/order.service.ts; package.json |
| ADR-019-TRANS-005 | docs/adrs/ADR-019-assinatura-hmac-por-endpoint.md | Precedente de Autorização | Replay manual da DLQ exige role `ADMIN` e auditoria de quem solicitou. | TRANSCRICAO | [09:36] Sofia |
| ADR-018-CODE-001 | docs/adrs/ADR-018-outbox-mysql-worker-polling.md | Estado atual | `changeStatus()` atualiza pedido, estoque e histórico na transação Prisma; o fluxo não grava evento webhook. | CODIGO | src/modules/orders/order.service.ts |
| ADR-018-TRANS-001 | docs/adrs/ADR-018-outbox-mysql-worker-polling.md | Alternativa rejeitada | Chamadas HTTP síncronas foram rejeitadas porque um endpoint lento ou indisponível poderia bloquear a mudança de status do pedido. | TRANSCRICAO | [09:04] Bruno |
| ADR-018-TRANS-002 | docs/adrs/ADR-018-outbox-mysql-worker-polling.md | Trade-off | Redis Streams foi considerada, mas adicionaria infraestrutura de mensageria ao projeto. | TRANSCRICAO | [09:07] Diego |
| ADR-018-TRANS-003 | docs/adrs/ADR-018-outbox-mysql-worker-polling.md | Restrição | A ordenação descrita depende de um único worker; a reunião não estabeleceu garantia global para múltiplos workers. | TRANSCRICAO | [09:13] Larissa |
| ADR-020-TRANS-001 | docs/adrs/ADR-020-entrega-at-least-once-com-event-id.md | Alternativa rejeitada | Exactly-once exigiria coordenação entre remetente e destinatário e foi considerada mais complexa que at-least-once com `event_id`. | TRANSCRICAO | [09:25] Diego |
| ADR-020-TRANS-002 | docs/adrs/ADR-020-entrega-at-least-once-com-event-id.md | Restrição | A reunião não definiu garantia de ordenação global dos eventos. | TRANSCRICAO | [09:13] Larissa |
| ADR-021-TRANS-001 | docs/adrs/ADR-021-autorizacao-por-vinculo-usuario-cliente.md | Contexto | A reunião indicou que o CRUD de configuração poderia ser usado por qualquer role autenticada. | TRANSCRICAO | [09:37] Sofia |
| ADR-021-CODE-002 | docs/adrs/ADR-021-autorizacao-por-vinculo-usuario-cliente.md | Estado atual | O middleware valida JWT, popula `req.user` com id, email e role, e oferece autorização por roles com `requireRole`. | CODIGO | src/middlewares/auth.middleware.ts |
| ADR-023-TRANS-001 | docs/adrs/ADR-023-classificar-payload-excedente-na-criacao.md | Requisito | A reunião definiu limite de payload de 64 KB e erro quando excedido. | TRANSCRICAO | [09:24] Larissa |
| FDD-CTX-001 | docs/FDD.md | Estado atual | `changeStatus()` atualiza pedido, estoque e histórico dentro de uma transação Prisma. | CODIGO | src/modules/orders/order.service.ts |
| FDD-CTX-002 | docs/FDD.md | Lacuna da Implementação Atual | O schema Prisma consultado não contém modelos de webhook, outbox ou DLQ. | CODIGO | prisma/schema.prisma |
| FDD-CTX-003 | docs/FDD.md | Lacuna da Implementação Atual | O pacote atual não declara comando `worker`; o processo worker é um componente novo. | CODIGO | package.json |
| FDD-PATTERN-001 | docs/FDD.md | Dependência Técnica | Reutilizar AppError, Pino, middleware centralizado de erros, schemas Zod e prefixo `WEBHOOK_` nos erros do módulo. | TRANSCRICAO | [09:30] Larissa |
| FDD-OBJ-001 | docs/FDD.md | Requisito de Integridade | Evento elegível e mudança de status devem ser persistidos na mesma transação; falha de persistência reverte a operação. | TRANSCRICAO | [09:40] Bruno |
| FDD-OBJ-002 | docs/FDD.md | Restrição | A chamada HTTP ao endpoint externo não deve ocorrer no fluxo síncrono de mudança de status. | TRANSCRICAO | [09:04] Bruno |
| FDD-OBJ-003 | docs/FDD.md | Objetivo | O objetivo de tempo para primeira entrega é inferior a 10 segundos em operação normal. | TRANSCRICAO | [09:02] Marcos |
| FDD-OBJ-004 | docs/FDD.md | Requisito Não Funcional | O worker consulta a outbox por polling a cada 2 segundos. | TRANSCRICAO | [09:09] Diego |
| FDD-SCOPE-001 | docs/FDD.md | Restrição de Escopo | A feature é outbound; clientes não enviam webhooks para o OMS. | TRANSCRICAO | [09:02] Marcos |
| FDD-SCOPE-002 | docs/FDD.md | Requisito Funcional | Usuários autenticados podem cadastrar, editar, remover e listar configurações de webhook. | TRANSCRICAO | [09:33] Bruno |
| FDD-SCOPE-003 | docs/FDD.md | Requisito de Segurança | `customer_id` é enviado no body ou path e não é inferido diretamente do JWT. | TRANSCRICAO | [09:32] Larissa |
| FDD-SCOPE-004 | docs/FDD.md | Regra de Negócio | Endpoints ativos são filtrados pelas inscrições no status de destino antes da inserção na outbox. | TRANSCRICAO | [09:34] Diego |
| FDD-SCOPE-005 | docs/FDD.md | Restrição de Escopo | Notificação por e-mail de falhas fica fora desta fase. | TRANSCRICAO | [09:37] Larissa |
| FDD-SCOPE-006 | docs/FDD.md | Restrição de Escopo | Dashboard visual para clientes fica fora desta fase; serão oferecidos endpoints de API. | TRANSCRICAO | [09:40] Larissa |
| FDD-SCOPE-007 | docs/FDD.md | Restrição de Escopo | Rate limiting de saída fica fora desta fase; o volume será observado para avaliação futura. | TRANSCRICAO | [09:39] Larissa |
| FDD-SCOPE-008 | docs/FDD.md | Restrição de Escopo | Arquivamento de registros entregues após cerca de 30 dias fica fora da feature. | TRANSCRICAO | [09:08] Diego |
| FDD-SCOPE-009 | docs/FDD.md | Restrição de Escopo | A primeira versão não deve assumir garantia de ordenação global nem múltiplos workers paralelos. | TRANSCRICAO | [09:13] Larissa |
| FDD-FLOW-001 | docs/FDD.md | Requisito de Arquitetura | O worker deve executar em processo separado da API. | TRANSCRICAO | [09:11] Larissa |
| FDD-FLOW-002 | docs/FDD.md | Requisito de Arquitetura | O worker conecta ao mesmo banco e usa instância Prisma própria por processo. | TRANSCRICAO | [09:30] Bruno |
| FDD-FLOW-003 | docs/FDD.md | Decisão | O evento usa UUID e guarda snapshot do payload no momento da inserção. | TRANSCRICAO | [09:52] Larissa |
| FDD-CONTRACT-001 | docs/FDD.md | Contrato | Cadastro recebe URL e status de interesse; a plataforma gera a secret e a devolve na criação. | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRACT-002 | docs/FDD.md | Contrato | O histórico deve expor até 100 entregas recentes com status, payload, resposta e tempo de resposta. | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRACT-003 | docs/FDD.md | Contrato | A consulta de histórico foi definida como `GET /webhooks/:id/deliveries`. | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRACT-004 | docs/FDD.md | Contrato | O replay manual da DLQ foi definido como `POST /admin/webhooks/dead-letter/:id/replay`. | TRANSCRICAO | [09:35] Diego |
| FDD-CONTRACT-005 | docs/FDD.md | Contrato | O payload descreve mudança de status com `event_id`, `event_type`, timestamp ISO 8601, dados do pedido e `customer_id`, sem itens. | TRANSCRICAO | [09:43] Diego |
| FDD-CONTRACT-006 | docs/FDD.md | Contrato | A entrega inclui `X-Event-Id`, `X-Signature`, `X-Timestamp`, `Content-Type` e `X-Webhook-Id`. | TRANSCRICAO | [09:44] Sofia |
| FDD-SEC-001 | docs/FDD.md | Requisito de Segurança | Assinar o corpo outbound com HMAC-SHA256. | TRANSCRICAO | [09:20] Sofia |
| FDD-SEC-002 | docs/FDD.md | Requisito de Segurança | Cada endpoint deve usar uma secret exclusiva, não uma secret global. | TRANSCRICAO | [09:21] Sofia |
| FDD-SEC-003 | docs/FDD.md | Requisito de Segurança | Após rotação, a secret anterior permanece válida em paralelo por 24 horas. | TRANSCRICAO | [09:21] Sofia |
| FDD-SEC-004 | docs/FDD.md | Requisito de Segurança | Recusar URL HTTP e exigir HTTPS para destinos de webhook. | TRANSCRICAO | [09:23] Sofia |
| FDD-ERR-004 | docs/FDD.md | Requisito Não Funcional | Payload acima de 64 KB não deve ser enviado nem truncado; a destinação posterior ainda precisa ser definida. | TRANSCRICAO | [09:24] Larissa |
| FDD-ERR-001 | docs/FDD.md | Requisito Não Funcional | Chamada HTTP que excede 10 segundos é tratada como falha de entrega. | TRANSCRICAO | [09:42] Diego |
| FDD-ERR-002 | docs/FDD.md | Política de Retentativa | Os intervalos registrados para as cinco tentativas são 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas. | TRANSCRICAO | [09:17] Diego |
| FDD-ERR-003 | docs/FDD.md | Decisão | Persistir falhas permanentes em tabela DLQ separada com payload, motivo e timestamp. | TRANSCRICAO | [09:18] Diego |
| FDD-AUTH-001 | docs/FDD.md | Requisito de Segurança | Replay da DLQ exige role `ADMIN` e registra quem solicitou. | TRANSCRICAO | [09:36] Sofia |
| FDD-AUTH-003 | docs/FDD.md | Requisito de Segurança | Operações de gerenciamento de webhook usam JWT; o middleware atual valida JWT e popula a identidade autenticada. | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-AUTH-002 | docs/FDD.md | Lacuna da Implementação Atual | O schema atual não declara vínculo entre `User` e `Customer`; o mecanismo de autorização por vínculo não está implementado. | CODIGO | prisma/schema.prisma |
| FDD-DELIVERY-001 | docs/FDD.md | Semântica de Entrega | Entrega at-least-once permite duplicatas; consumidor deduplica pelo UUID estável em `X-Event-Id`. | TRANSCRICAO | [09:25] Diego |
| FDD-OBS-001 | docs/FDD.md | Dependência Técnica | O projeto já configura logger Pino com redação de campos sensíveis selecionados. | CODIGO | src/shared/logger/index.ts |
| FDD-DEP-001 | docs/FDD.md | Dependência Técnica | O projeto declara Node.js `>=20`. | CODIGO | package.json |
| FDD-DEP-002 | docs/FDD.md | Dependência Técnica | O Prisma existente usa provider MySQL. | CODIGO | prisma/schema.prisma |
| FDD-DEP-003 | docs/FDD.md | Dependência Técnica | A dependência `@prisma/client` existente está fixada em 5.22.0. | CODIGO | package.json |
| FDD-DEP-004 | docs/FDD.md | Dependência Organizacional | A revisão de segurança deve ocorrer antes do deploy, reservando pelo menos dois dias úteis. | TRANSCRICAO | [09:46] Sofia |
| FDD-DEP-005 | docs/FDD.md | Dependência Técnica | O projeto usa Express 4.21.1. | CODIGO | package.json |
| FDD-DEP-006 | docs/FDD.md | Dependência Técnica | O projeto usa Zod 3.23.8 para validação de schemas. | CODIGO | package.json |
