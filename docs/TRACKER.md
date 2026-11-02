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
| RFC-CTX-001 | docs/rfcs/RFC-webhooks-pedidos.md | Problema | Clientes B2B consultam repetidamente GET /orders para detectar mudanças de status. | TRANSCRICAO | [09:00] Marcos |
| RFC-ARCH-001 | docs/rfcs/RFC-webhooks-pedidos.md | Decisão | Usar Transactional Outbox no MySQL existente em vez de adicionar broker nesta versão. | TRANSCRICAO | [09:06] Diego |
| RFC-ARCH-002 | docs/rfcs/RFC-webhooks-pedidos.md | Decisão | Executar o worker em processo separado da API, conectado ao mesmo banco. | TRANSCRICAO | [09:11] Diego |
| RFC-ARCH-003 | docs/rfcs/RFC-webhooks-pedidos.md | Requisito Não Funcional | Consultar eventos pendentes por polling a cada 2 segundos. | TRANSCRICAO | [09:09] Diego |
| RFC-REQ-001 | docs/rfcs/RFC-webhooks-pedidos.md | Requisito Funcional | Filtrar endpoints inscritos no status de destino antes de inserir eventos na outbox. | TRANSCRICAO | [09:33] Marcos |
| RFC-RETRY-001 | docs/rfcs/RFC-webhooks-pedidos.md | Política de Retentativa | Usar intervalos de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas. | TRANSCRICAO | [09:17] Diego |
| RFC-DLQ-001 | docs/rfcs/RFC-webhooks-pedidos.md | Decisão | Manter DLQ separada com payload, motivo da falha e timestamp para diagnóstico/replay. | TRANSCRICAO | [09:18] Diego |
| RFC-SEC-001 | docs/rfcs/RFC-webhooks-pedidos.md | Requisito de Segurança | Assinar o corpo dos requests com HMAC-SHA256. | TRANSCRICAO | [09:20] Sofia |
| RFC-SEC-002 | docs/rfcs/RFC-webhooks-pedidos.md | Requisito de Segurança | Usar secret exclusiva por endpoint e suportar rotação com sobreposição anterior de 24 horas. | TRANSCRICAO | [09:21] Sofia |
| RFC-SEC-003 | docs/rfcs/RFC-webhooks-pedidos.md | Requisito de Segurança | Exigir HTTPS para URLs de webhook e recusar HTTP. | TRANSCRICAO | [09:23] Sofia |
| RFC-DELIVERY-001 | docs/rfcs/RFC-webhooks-pedidos.md | Semântica de Entrega | Entrega at-least-once permite duplicatas; o consumidor pode deduplicar usando UUID em X-Event-Id. | TRANSCRICAO | [09:25] Diego |
| RFC-API-001 | docs/rfcs/RFC-webhooks-pedidos.md | Contrato | As operações da API usam JWT e recebem customer_id no body ou path, não inferido diretamente do token. | TRANSCRICAO | [09:32] Larissa |
| RFC-API-002 | docs/rfcs/RFC-webhooks-pedidos.md | Requisito Funcional | Expor histórico com até 100 entregas, incluindo sucesso/falha, payload, resposta e tempo. | TRANSCRICAO | [09:34] Marcos |
| RFC-API-003 | docs/rfcs/RFC-webhooks-pedidos.md | Requisito de Segurança | Restringir replay da DLQ à role ADMIN e auditar quem solicitou. | TRANSCRICAO | [09:36] Sofia |
| RFC-ATOMIC-001 | docs/rfcs/RFC-webhooks-pedidos.md | Requisito de Integridade | Inserir o evento na mesma transação da mudança de status; falha de persistência na outbox causa rollback. | TRANSCRICAO | [09:40] Bruno |
| RFC-HTTP-001 | docs/rfcs/RFC-webhooks-pedidos.md | Requisito Não Funcional | Tratar chamada HTTP acima de 10 segundos como falha para retry. | TRANSCRICAO | [09:42] Diego |
| RFC-CONTRACT-001 | docs/rfcs/RFC-webhooks-pedidos.md | Contrato | O payload descreve order.status_changed com IDs/número do pedido, status anterior/novo, customer_id e campos básicos, sem items. | TRANSCRICAO | [09:43] Diego |
| RFC-CONTRACT-002 | docs/rfcs/RFC-webhooks-pedidos.md | Contrato | Enviar X-Event-Id, X-Signature, X-Timestamp, Content-Type e X-Webhook-Id. | TRANSCRICAO | [09:44] Sofia |
| RFC-SEC-REVIEW-001 | docs/rfcs/RFC-webhooks-pedidos.md | Dependência Organizacional | Reservar ao menos dois dias úteis para revisão de segurança antes do deploy. | TRANSCRICAO | [09:46] Sofia |
| RFC-SNAPSHOT-001 | docs/rfcs/RFC-webhooks-pedidos.md | Decisão | Gerar UUID e snapshot do payload no momento da inserção para refletir o estado do pedido naquela mudança. | TRANSCRICAO | [09:51] Larissa |
| RFC-CODE-001 | docs/rfcs/RFC-webhooks-pedidos.md | Evidência de Implementação Atual | changeStatus() usa transação Prisma para atualizar pedido, histórico e efeitos de estoque. | CODIGO | src/modules/orders/order.service.ts |
| RFC-CODE-002 | docs/rfcs/RFC-webhooks-pedidos.md | Evidência de Implementação Atual | Middleware authenticate valida JWT; requireRole() autoriza pelas roles ADMIN e OPERATOR. | CODIGO | src/middlewares/auth.middleware.ts |
| RFC-CODE-003 | docs/rfcs/RFC-webhooks-pedidos.md | Restrição Técnica | O datasource Prisma existente usa MySQL. | CODIGO | prisma/schema.prisma |
| RFC-CODE-004 | docs/rfcs/RFC-webhooks-pedidos.md | Lacuna da Implementação Atual | Os modelos User e Customer não declaram relação de vínculo usuário-cliente no schema atual. | CODIGO | prisma/schema.prisma |
| RFC-CODE-005 | docs/rfcs/RFC-webhooks-pedidos.md | Evidência de Implementação Atual | O logger Pino existente redige campos sensíveis configurados, incluindo authorization, cookie, password e token. | CODIGO | src/shared/logger/index.ts |
