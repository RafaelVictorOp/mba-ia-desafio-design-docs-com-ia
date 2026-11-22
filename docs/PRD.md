### PRD: Order Management System Sistema de Webhooks de Notificação de Pedidos

| Campo | Valor |
| --- | --- |
| ID | PRD-webhooks-pedidos |
| Status | Draft |
| Autor | Marcos (Product Manager) |
| Data | 2026-10-31 |
| Versão | Pendente de definição |
| Revisores | A definir; revisão de segurança por Sofia foi informada como necessária antes do deploy |
| Documentos relacionados | [docs/RFC.md](RFC.md), [TRANSCRICAO.md](../TRANSCRICAO.md), [docs/TRACKER.md](TRACKER.md) |

---

### Resumo

O Order Management System (OMS) existente passará a notificar clientes B2B sobre mudanças de status de pedidos por webhooks de saída. A feature atende a solicitações de Atlas Comercial, MaxDistribuição e Nova Cargo, reduzindo a necessidade de consultas periódicas à API. A solução usa Transactional Outbox no MySQL existente e um worker separado para entrega assíncrona, com HMAC-SHA256, retentativas e dead-letter queue (DLQ). O objetivo é iniciar a entrega em menos de 10 segundos em operação normal; esse valor não é uma garantia absoluta.

---

### Contexto e problema

Público-alvo
- Clientes B2B que integram seus sistemas ao OMS, incluindo Atlas Comercial, MaxDistribuição e Nova Cargo.
- Administradores internos responsáveis por reprocessar entregas na DLQ.

Cenários de uso chave
- Um cliente configura um endpoint HTTPS e escolhe os status de pedido que deseja receber.
- Quando um pedido muda para um status assinado, o sistema cria um evento e tenta entregá-lo ao endpoint do cliente.
- O cliente valida a assinatura, identifica o evento e pode deduplicar tentativas repetidas por `X-Event-Id`.
- Um usuário consulta o histórico de entregas e um administrador pode solicitar replay de evento na DLQ.

Onde essa feature será implantada
- No OMS existente, na API Node.js/TypeScript e no MySQL/Prisma existentes. O envio será executado por um novo processo worker separado da API.

Problemas priorizados
- Clientes consultam repetidamente `GET /orders` para descobrir mudanças de status. Isso torna as integrações mais lentas e caras para eles; a frequência e os custos não foram quantificados. Prioridade alta.
- Três clientes B2B solicitaram formalmente notificações automáticas. A Atlas Comercial sinalizou risco de migrar para um concorrente se a necessidade não for atendida até o fim do trimestre. Impacto comercial alto; prioridade alta.
- Chamadas síncronas a endpoints de clientes lentos ou indisponíveis poderiam bloquear mudanças de status de pedidos. Impacto operacional alto; prioridade alta.

---

### Objetivos e métricas

| Objetivo | Métrica | Meta |
| --- | --- | --- |
| Reduzir a espera dos clientes para saber que o status do pedido mudou. | Tempo entre a confirmação da mudança e a primeira tentativa de entrega. | Menos de 10 segundos em operação normal; polling configurado a cada 2 segundos. Não é garantia absoluta, sobretudo durante falhas, retentativas ou acúmulo de eventos. |
| Garantir que mudanças de status elegíveis não sejam confirmadas sem registro do evento correspondente. | Número de mudanças elegíveis confirmadas sem evento correspondente na outbox. | 0; mudança e evento são confirmados ou revertidos juntos na mesma transação. |
| Permitir novas tentativas controladas quando um endpoint estiver indisponível. | Número máximo de retentativas adicionais antes da classificação como falha permanente. | 5 retentativas adicionais à tentativa inicial, com intervalos de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas; esta contagem foi esclarecida durante a entrevista, pois a transcrição não determinava se a tentativa inicial entrava no limite. |

---

### Escopo

Incluso
- Cadastro, edição, remoção e listagem de configurações de webhook por cliente, com filtros de status e secret exclusiva gerada pela plataforma.
- Rotação de secret, mantendo a anterior válida por 24 horas após a rotação.
- Criação de eventos somente para endpoints ativos inscritos no status de destino, com snapshot do payload no momento da mudança.
- Persistência transacional dos eventos em uma outbox MySQL e envio assíncrono por worker separado.
- Assinatura HMAC-SHA256, cabeçalhos de identificação e timestamp, HTTPS obrigatório e limite de payload de 64 KB.
- Retentativas com backoff, persistência de falhas permanentes em tabela DLQ, histórico das últimas 100 entregas por webhook e replay manual restrito a ADMIN.
- Reuso dos padrões existentes do projeto para módulos, validação, erros, logging, autenticação e autorização por role.

Fora de escopo
- Envio de e-mail para avisar clientes sobre falhas (`[09:37] Larissa`, `[09:38] Marcos`).
- Dashboard visual para clientes; nesta fase serão oferecidos endpoints de API (`[09:39] Marcos`, `[09:40] Larissa`).
- Rate limiting de saída por cliente; o volume será observado e a necessidade avaliada posteriormente, sem implementar controle de taxa nesta fase (`[09:38] Diego`, `[09:39] Larissa`).
- Garantia de ordenação global ou paralelismo de múltiplos workers na primeira versão (`[09:12] Diego`, `[09:13] Larissa`).
- Arquivamento de registros entregues após 30 dias (`[09:08] Diego`).
- Webhooks inbound (cliente para a plataforma), pois o escopo é exclusivamente outbound (`[09:02] Sofia`, `[09:02] Marcos`).

---

### Rastreabilidade temporal da transcrição

Os momentos abaixo registram exatamente quando cada assunto foi abordado na reunião. Quando houver mais de um ponto relevante para o mesmo item, todos os timestamps literais da transcrição são preservados.

#### Contexto, objetivos e escopo
- **Proposta dos clientes B2B e motivo do problema:** pedidos da Atlas Comercial, MaxDistribuição e Nova Cargo e consulta repetitiva a `GET /orders` (`[09:00] Marcos`).
- **Risco comercial da Atlas Comercial:** migração para concorrente se a entrega não ocorrer até o fim do trimestre (`[09:00] Marcos`).
- **Critério de tempo real:** menos de 10 segundos é aceitável para os clientes (`[09:02] Marcos`).
- **Escopo outbound:** o OMS envia aos clientes; não recebe webhooks deles (`[09:02] Sofia`, `[09:02] Marcos`, `[09:03] Sofia`).
- **Prazo comercial:** entrega desejada para a Atlas até o fim de novembro (`[09:45] Marcos`, `[09:47] Larissa`).

#### Arquitetura e integridade
- **Descartar chamada síncrona:** não executar HTTP durante a mudança de status, pois clientes lentos ou indisponíveis poderiam travar pedidos (`[09:03] Larissa`, `[09:04] Bruno`, `[09:04] Larissa`, `[09:05] Larissa`).
- **Decisão pelo padrão outbox:** persistir evento na mesma transação da mudança de status (`[09:06] Diego`, `[09:06] Bruno`, `[09:06] Diego`).
- **Descartar Redis Streams:** evitar nova infraestrutura; usar o MySQL existente (`[09:07] Larissa`, `[09:07] Diego`).
- **Estrutura da outbox:** índice por status e `created_at`, batch pequeno e marcação de entregue (`[09:08] Diego`).
- **Polling a cada 2 segundos:** decisão e justificativa sobre reatividade e trigger (`[09:08] Larissa`, `[09:09] Diego`, `[09:09] Bruno`, `[09:09] Diego`, `[09:10] Marcos`, `[09:10] Larissa`).
- **Worker em processo separado:** independência do ciclo de vida da API (`[09:11] Diego`, `[09:11] Larissa`, `[09:11] Bruno`, `[09:11] Diego`).
- **Entry point e comando:** entrada proposta em `src/worker.ts` e script `npm run worker` (`[09:11] Larissa`, `[09:28] Bruno`, `[09:28] Diego`).
- **Single-worker e ordenação:** ordem por `created_at`, ausência de garantia global e escala futura fora da primeira versão (`[09:12] Larissa`, `[09:12] Diego`, `[09:13] Bruno`, `[09:13] Diego`, `[09:13] Larissa`, `[09:14] Marcos`, `[09:14] Larissa`).
- **Atomicidade no fluxo de pedidos:** pedido, histórico, estoque e outbox na mesma transação, com rollback da outbox revertendo tudo (`[09:40] Bruno`, `[09:41] Diego`, `[09:41] Bruno`, `[09:41] Diego`).
- **Snapshot do payload:** o evento guarda o payload renderizado na inserção, não valores posteriores do pedido (`[09:51] Diego`, `[09:51] Larissa`, `[09:51] Diego`, `[09:51] Bruno`, `[09:52] Larissa`, `[09:52] Diego`, `[09:52] Bruno`).
- **UUID da outbox:** confirmação de uso de UUID, no mesmo padrão do restante do projeto (`[09:51] Diego`, `[09:51] Larissa`).

#### Requisitos funcionais
- **FR-001 Gerenciar configurações:** cadastro, edição, remoção e listagem por cliente (`[09:31] Marcos`, `[09:32] Marcos`, `[09:33] Bruno`, `[09:33] Marcos`).
- **FR-001 Campos e geração da secret:** URL, status, secret gerada pela plataforma e devolvida na criação (`[09:31] Marcos`, `[09:32] Bruno`, `[09:32] Marcos`).
- **FR-001 Associação por customer_id:** `customer_id` no body ou path, não inferido diretamente do JWT (`[09:32] Bruno`, `[09:32] Marcos`, `[09:32] Larissa`).
- **FR-002 Filtragem por status:** filtrar na inserção da outbox apenas endpoints ativos inscritos no status de destino (`[09:33] Marcos`, `[09:34] Diego`, `[09:34] Bruno`, `[09:34] Diego`).
- **FR-003 Atomicidade e rollback:** persistir o evento elegível junto com a alteração de status (`[09:06] Diego`, `[09:40] Bruno`, `[09:41] Diego`).
- **FR-004 Worker e entrega:** worker separado, polling, POST HTTPS, batching e registro do resultado (`[09:08] Diego`, `[09:09] Diego`, `[09:11] Diego`, `[09:28] Bruno`).
- **FR-004 Timeout:** chamadas sem resposta em 10 segundos são consideradas falha (`[09:42] Sofia`, `[09:42] Diego`, `[09:42] Sofia`).
- **FR-004 Payload limit:** rejeitar payload acima de 64 KB sem truncamento (`[09:23] Sofia`, `[09:23] Diego`, `[09:24] Larissa`, `[09:24] Diego`).
- **FR-005 HMAC-SHA256:** assinatura do payload com secret compartilhada e header `X-Signature` (`[09:19] Sofia`, `[09:20] Sofia`, `[09:20] Bruno`, `[09:20] Sofia`).
- **FR-005 Secret exclusiva por endpoint:** não usar secret global (`[09:21] Sofia`, `[09:21] Bruno`, `[09:21] Sofia`).
- **FR-005 Rotação:** a secret antiga continua válida por 24 horas após a rotação (`[09:21] Sofia`, `[09:22] Diego`, `[09:22] Sofia`, `[09:22] Larissa`).
- **FR-005 HTTPS obrigatório:** URL HTTP recusada como validação no schema Zod (`[09:23] Sofia`).
- **FR-006 Retentativas:** cinco tentativas com backoff e intervalos de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas (`[09:14] Larissa`, `[09:15] Diego`, `[09:15] Bruno`, `[09:15] Diego`, `[09:16] Bruno`, `[09:16] Diego`, `[09:16] Larissa`, `[09:17] Diego`, `[09:17] Marcos`, `[09:17] Larissa`).
- **Interpretação da contagem:** cinco tentativas adicionais após a tentativa inicial (`[09:48] Larissa`); a transcrição não esclarece diretamente se a inicial integra o teto.
- **FR-006 DLQ:** tabela separada preservando payload, motivo da falha e timestamp (`[09:17] Larissa`, `[09:18] Diego`, `[09:18] Bruno`, `[09:18] Diego`).
- **FR-007 Histórico de entregas:** até 100 entregas com status, payload, resposta e tempo de resposta (`[09:34] Marcos`, `[09:35] Larissa`).
- **FR-007 Contrato de consulta:** `GET /webhooks/:id/deliveries` (`[09:34] Marcos`, `[09:35] Larissa`).
- **FR-008 Replay da DLQ:** `POST /admin/webhooks/dead-letter/:id/replay`, recolocando o evento na outbox como pendente (`[09:18] Diego`, `[09:18] Bruno`, `[09:18] Diego`, `[09:19] Larissa`, `[09:35] Diego`, `[09:35] Larissa`).
- **FR-008 Role administrativa:** replay exige role `ADMIN` e deve registrar quem realizou a ação (`[09:35] Larissa`, `[09:36] Sofia`, `[09:36] Larissa`).
- **FR-009 Worker separado:** entrada própria, mesmo banco e instância Prisma própria por processo (`[09:11] Diego`, `[09:11] Larissa`, `[09:11] Bruno`, `[09:11] Diego`, `[09:28] Bruno`, `[09:29] Diego`, `[09:30] Bruno`).

#### Contrato de entrega e semântica
- **At-least-once:** duplicatas podem ocorrer; exatamente uma vez foi descartado por complexidade (`[09:24] Diego`, `[09:25] Bruno`, `[09:25] Diego`, `[09:25] Sofia`, `[09:25] Diego`, `[09:26] Marcos`, `[09:26] Larissa`).
- **`X-Event-Id`:** UUID gerado na outbox para deduplicação no cliente (`[09:24] Diego`, `[09:25] Bruno`, `[09:25] Diego`, `[09:26] Larissa`).
- **Payload:** `event_id`, `event_type`, timestamp ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id`, `total_cents`; sem itens (`[09:43] Marcos`, `[09:43] Diego`, `[09:44] Bruno`).
- **Headers:** `X-Event-Id`, `X-Signature`, `X-Timestamp`, `Content-Type: application/json` e `X-Webhook-Id` (`[09:44] Diego`, `[09:44] Sofia`, `[09:45] Diego`).
- **Timestamp do request:** permitir ao cliente detectar replay attacks antigos (`[09:44] Diego`).

#### Padrões do projeto e arquitetura de código
- **Padrão de módulos:** `src/modules/webhooks` com controller, service, repository, routes e schemas (`[09:27] Larissa`, `[09:27] Bruno`, `[09:28] Diego`, `[09:28] Bruno`).
- **Padrões compartilhados:** AppError, Pino, middleware centralizado de erros, Zod e códigos prefixados `WEBHOOK_` (`[09:28] Bruno`, `[09:29] Larissa`, `[09:29] Bruno`, `[09:29] Diego`, `[09:30] Larissa`).
- **Prisma separado no worker:** mesmo banco e `DATABASE_URL`, mas nova instância por processo (`[09:29] Diego`, `[09:30] Bruno`, `[09:30] Larissa`).

#### Segurança, autorização e revisão
- **TLS obrigatório:** somente HTTPS para endpoints cadastrados (`[09:23] Sofia`).
- **Não registrar secrets em logs:** compromisso de controle mencionado para revisão de segurança (`[09:49] Sofia`, `[09:49] Larissa`).
- **JWT e associação de cliente:** CRUD autenticado, `customer_id` fornecido na requisição e vínculo usuário-cliente pendente (`[09:31] Marcos`, `[09:32] Bruno`, `[09:32] Marcos`, `[09:32] Larissa`, `[09:36] Marcos`, `[09:36] Sofia`, `[09:37] Sofia`).
- **Roles:** replay restrito a `ADMIN`; demais CRUDs por usuário autenticado, sem endurecimento definido nesta fase (`[09:35] Larissa`, `[09:36] Sofia`, `[09:36] Larissa`, `[09:36] Marcos`, `[09:37] Sofia`).
- **Revisão de segurança:** reservar pelo menos dois dias úteis antes do deploy, inclusive para HMAC e geração de secret (`[09:46] Sofia`, `[09:47] Larissa`, `[09:49] Sofia`).

#### Riscos e limitações
- **Endpoint indisponível ou lento:** timeout, retries, DLQ e replay como mitigação (`[09:14] Larissa`, `[09:15] Diego`, `[09:42] Diego`, `[09:18] Diego`).
- **Duplicidade:** impacto mitigado pela deduplicação do cliente (`[09:24] Diego`, `[09:25] Diego`, `[09:26] Larissa`).
- **Ordenação:** single-worker oferece expectativa por `order_id`, não garantia global (`[09:12] Diego`, `[09:13] Diego`, `[09:13] Larissa`, `[09:14] Larissa`).
- **Acesso entre clientes:** lacuna de autorização a ser definida antes do deploy (`[09:32] Larissa`, `[09:36] Marcos`, `[09:37] Sofia`).
- **Vazamento anterior de secret:** exposição em log de cliente motivou secret por endpoint e rotação com grace period (`[09:22] Diego`, `[09:22] Sofia`).
- **Rate limiting:** exemplo de 50 mudanças em um minuto e decisão de observar o volume antes de implementar controle (`[09:38] Diego`, `[09:39] Larissa`).

#### Planejamento e validação
- **Estimativa:** três sprints, incluindo revisão de segurança (`[09:45] Marcos`, `[09:46] Larissa`, `[09:46] Sofia`, `[09:47] Larissa`, `[09:47] Marcos`).
- **Testes:** unitários e integração para regras de negócio, transações e rollback (`[09:40] Bruno`, `[09:41] Bruno`, `[09:50] Larissa`).
- **Fechamento da reunião:** decisões confirmadas por Diego, Bruno e Sofia (`[09:49] Diego`, `[09:49] Bruno`, `[09:49] Sofia`, `[09:53] Diego`, `[09:53] Bruno`, `[09:53] Larissa`).

---

### Requisitos funcionais

#### FR-001 Gerenciar configurações de webhook
A API deve permitir que usuários autenticados cadastrem, editem, removam e listem configurações de webhook associadas a um cliente. O cadastro inclui URL HTTPS e lista de status de interesse. A plataforma gera uma secret exclusiva para cada endpoint e devolve a secret na criação. O `customer_id` é informado no body ou no path, não inferido diretamente do JWT.

**Fluxo principal**
- O usuário autenticado envia cliente, URL HTTPS e status de interesse.
- A API valida a requisição, cria a configuração e gera uma secret exclusiva.
- A API devolve a configuração e a secret gerada; operações posteriores permitem consultar ou modificar configurações.

**Fluxos alternativos e exceções**
- O cliente pode alterar a configuração e a lista de status de interesse.
- A remoção deve permitir excluir a configuração. O tratamento dos eventos já enfileirados para esse endpoint permanece pendente de definição.
- A autorização do usuário para operar sobre o `customer_id` informado permanece pendente de definição técnica.

**Erros previstos**
- URL inválida ou sem HTTPS: erro de validação, com código `WEBHOOK_INVALID_URL` proposto na reunião.
- Configuração inexistente: erro `WEBHOOK_NOT_FOUND` proposto na reunião.
- Entrada inválida ou secret obrigatória quando aplicável: erro de validação ou código `WEBHOOK_SECRET_REQUIRED`, conforme o caso.

**Prioridade:** alta

---

#### FR-002 Filtrar eventos pelos status inscritos
Cada configuração deve declarar os status de pedido que deseja receber. A plataforma deve selecionar as configurações elegíveis antes de inserir eventos na outbox.

**Fluxo principal**
- Quando o status do pedido muda, o sistema consulta configurações ativas do cliente inscritas no status de destino.
- É criado um evento por endpoint elegível.
- Se nenhum endpoint estiver inscrito, nenhum evento de webhook é criado.

**Fluxos alternativos e exceções**
- Cada endpoint pode assinar somente um subconjunto dos status disponíveis.
- A filtragem ocorre na inserção do evento na outbox, e não no momento do envio.

**Erros previstos**
- Status ou filtro inválido: erro de validação.
- Falha ao persistir evento elegível: a transação da mudança de status deve falhar integralmente.

**Prioridade:** alta

---

#### FR-003 Persistir evento atomicamente e como snapshot
Para cada endpoint elegível, o sistema deve gravar na mesma transação da mudança do pedido um snapshot do evento na outbox MySQL.

**Fluxo principal**
- A transação atualiza o pedido, grava o histórico de status e atualiza o estoque conforme as regras existentes.
- Na mesma transação, grava o evento elegível na outbox com UUID e payload renderizado naquele instante.
- O commit confirma conjuntamente as alterações; qualquer falha de inserção reverte toda a transação.

**Fluxos alternativos e exceções**
- O evento conserva os valores do momento da mudança, mesmo que o pedido mude posteriormente.
- Sem endpoint inscrito no status, não há evento de webhook para persistir.

**Erros previstos**
- Falha de banco ou de escrita na outbox: rollback de todas as alterações da transação, sem mudança parcial de status, histórico ou estoque.

**Prioridade:** alta

---

#### FR-004 Processar e enviar webhooks
Um worker em processo separado deve consultar eventos pendentes, enviar o payload JSON ao endpoint HTTPS e registrar o resultado da entrega.

**Fluxo principal**
- O worker conecta ao mesmo banco usando sua própria instância Prisma e consulta eventos pendentes mais antigos em batches pequenos.
- O worker envia um POST JSON com `X-Event-Id`, `X-Webhook-Id`, `X-Signature`, `X-Timestamp` e `Content-Type: application/json`.
- O sistema registra status, payload, resposta recebida e tempo de resposta para compor o histórico.

**Fluxos alternativos e exceções**
- O worker consulta a outbox por polling a cada 2 segundos e processa eventos por ordem de criação; a ordem efetiva de recebimento não é garantida diante de falhas, retentativas ou tempos de resposta diferentes.
- Chamadas com mais de 10 segundos são tratadas como falha de entrega.
- Payload maior que 64 KB não deve ser enviado nem truncado; o tratamento após a rejeição permanece pendente de definição técnica.
- A entrega é at-least-once, portanto o mesmo evento pode ser enviado mais de uma vez.

**Erros previstos**
- Endpoint indisponível, erro HTTP ou timeout: encaminhar à política de retentativas.
- Payload acima do limite: reconhecer como erro e não enviar; destinação posterior pendente.
- Falha de assinatura ou configuração: registrar falha sem expor secrets nos logs.

**Prioridade:** alta

---

#### FR-005 Assinar eventos e permitir rotação de secrets
Cada endpoint deve ter uma secret exclusiva para assinar o corpo do request com HMAC-SHA256 e permitir sua rotação.

**Fluxo principal**
- A plataforma gera uma secret por endpoint e calcula a assinatura HMAC-SHA256 sobre o corpo enviado.
- A plataforma disponibiliza uma operação de API para solicitar a rotação da secret.
- Após a rotação, a secret anterior permanece válida por 24 horas e deixa de ser aceita após esse período.

**Fluxos alternativos e exceções**
- O cliente pode usar `X-Event-Id` para deduplicar entregas repetidas e `X-Timestamp` para detectar requests antigos.
- Qual secret assina novos eventos durante a janela em que ambas estão válidas permanece pendente de definição técnica.

**Erros previstos**
- Secret ausente ou sem configuração válida: erro `WEBHOOK_SECRET_REQUIRED` proposto na reunião.
- Falha de rotação ou configuração inexistente: código `WEBHOOK_*` correspondente, conforme definido na implementação.

**Prioridade:** alta

---

#### FR-006 Retentar falhas e registrar falhas permanentes
Falhas de entrega devem ser retentadas com backoff. Após o limite, o evento deve ser persistido em tabela DLQ separada com evidências para diagnóstico e replay.

**Fluxo principal**
- Após a tentativa inicial falhar, o sistema executa até cinco retentativas adicionais com os intervalos de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas.
- Após a última falha, o evento é movido para a tabela DLQ com payload, motivo da falha e timestamp.
- O histórico de entregas registra os resultados necessários para consulta e diagnóstico.

**Fluxos alternativos e exceções**
- A contagem de cinco retentativas adicionais foi esclarecida durante a entrevista; a transcrição da reunião definiu cinco tentativas sem especificar se incluía a tentativa inicial.
- Evento na DLQ pode ser reprocessado por solicitação administrativa. O replay não garante que uma nova entrega terá sucesso.

**Erros previstos**
- Falha ao persistir o evento na DLQ: registrar erro e não descartar o evento silenciosamente.
- Evento inexistente na DLQ: erro `WEBHOOK_NOT_FOUND`, conforme padrão proposto na reunião.

**Prioridade:** alta

---

#### FR-007 Consultar histórico de entregas
A API deve oferecer a consulta das até 100 entregas mais recentes de um webhook, incluindo resultado e dados para diagnóstico.

**Fluxo principal**
- O usuário solicita `GET /webhooks/:id/deliveries`.
- A resposta lista até 100 entregas recentes com status de sucesso ou falha, payload, resposta recebida e tempo de resposta.
- O acesso deve ser limitado a usuários autorizados para o cliente correspondente.

**Fluxos alternativos e exceções**
- Retornar lista vazia quando não houver entregas é comportamento complementar proposto durante a entrevista, não explicitamente discutido na reunião.
- O mecanismo de autorização entre usuário e `customer_id` permanece pendente; a validação completa desse requisito depende dessa definição.

**Erros previstos**
- Webhook inexistente: erro `WEBHOOK_NOT_FOUND`, conforme padrão proposto na reunião.
- Usuário não autorizado para o cliente: negar acesso; o mecanismo de autorização está pendente de definição.

**Prioridade:** média

---

#### FR-008 Reprocessar evento da DLQ como administrador
Um usuário com role `ADMIN` deve poder solicitar replay manual de evento na DLQ, com registro de quem solicitou a operação.

**Fluxo principal**
- Um usuário com role `ADMIN` chama `POST /admin/webhooks/dead-letter/:id/replay`.
- O sistema registra qual administrador solicitou o replay e recoloca o evento na outbox como pendente.
- O sistema reaproveita o mecanismo `requireRole` existente.

**Fluxos alternativos e exceções**
- Usuário autenticado sem role `ADMIN` não pode executar replay.
- O comportamento para replay duplicado e a preservação ou geração de um novo `event_id` permanecem pendentes de definição técnica.

**Erros previstos**
- Evento da DLQ não encontrado: erro `WEBHOOK_NOT_FOUND`, conforme padrão proposto na reunião.
- Role insuficiente: negar a operação conforme o padrão de autorização existente.
- O formato, armazenamento e campos adicionais do registro de auditoria permanecem pendentes.

**Prioridade:** alta

---

#### FR-009 Executar worker em processo separado
O worker deve iniciar e encerrar como processo separado da API, conectando-se ao mesmo banco com seu próprio PrismaClient.

**Fluxo principal**
- O projeto fornece uma entry point separada, prevista como `src/worker.ts`, e um comando previsto como `npm run worker`.
- O processo cria sua própria instância Prisma e executa polling periódico da outbox.
- A primeira versão opera com um único worker.

**Fluxos alternativos e exceções**
- O reinício da API não deve depender do ciclo de vida do worker.
- A arquitetura não promete ordenação global nem ordenação absoluta no recebimento de eventos.

**Erros previstos**
- Falha de conexão com o banco deve ser registrada e não pode resultar em confirmação de entrega não realizada.

**Prioridade:** alta

---

### Requisitos não funcionais

Performance
- Polling da outbox a cada 2 segundos.
- Objetivo de iniciar uma tentativa em menos de 10 segundos da mudança, em operação normal; não é garantia absoluta, especialmente em falhas, retentativas ou acúmulo de eventos.
- Timeout de 10 segundos por chamada HTTP.
- Payload máximo de 64 KB; acima desse limite, rejeitar sem enviar e sem truncar. O destino posterior do evento permanece pendente.
- A reunião não definiu meta de latência p95 para endpoints da API.

Disponibilidade
- Meta de disponibilidade mensal não definida na transcrição e permanece pendente; nenhum percentual é assumido.
- A outbox deve registrar o evento na transação do pedido, de modo que o processamento assíncrono não dependa de a chamada ao endpoint do cliente ocorrer durante a mudança de status.

Segurança e autorização
- URL de destino deve usar HTTPS; URL HTTP deve ser recusada.
- Assinar o corpo do request com HMAC-SHA256 e usar secret exclusiva por endpoint.
- Permitir rotação com sobreposição de validade de 24 horas; qual secret assina novos eventos nesse período permanece pendente.
- Replay da DLQ exige role `ADMIN` e deve registrar o administrador responsável; detalhes da auditoria pendentes.
- CRUD e consulta exigem autenticação JWT. A associação do usuário autenticado ao `customer_id` não foi especificada e deve ser definida para impedir acesso entre clientes.
- Não registrar secrets em logs é um controle a observar durante o design e a revisão de segurança; a transcrição não definiu formato de armazenamento das secrets.

Observabilidade
- Reutilizar o logger Pino existente.
- Registrar resultado, resposta e tempo de resposta das entregas no histórico.
- Registrar quem solicitou replay administrativo.
- Métricas e tracing não foram especificados na transcrição. A orientação de observar volume para avaliar futuramente rate limiting não define uma implementação de observabilidade para esta fase.

Confiabilidade e integridade de dados
- A mudança de status, histórico, atualização de estoque e inserção dos eventos elegíveis devem participar da mesma transação.
- Entrega at-least-once: duplicatas são possíveis. `X-Event-Id` fornece identificador para deduplicação no consumidor, cuja responsabilidade foi discutida na reunião.
- Aplicar cinco retentativas adicionais à tentativa inicial conforme interpretação confirmada durante a entrevista; após falhas, persistir na DLQ separada. Replay permite nova tentativa, mas não garante sucesso.
- A DLQ preserva payload, motivo e timestamp; formato ou campos adicionais de auditoria não foram definidos.
- Um único worker processa eventos por ordem de criação, como propriedade esperada da arquitetura inicial, não garantia absoluta de ordenação por pedido ou global. Falhas, retentativas e tempos de resposta podem mudar a ordem efetiva de recebimento.

Compatibilidade e portabilidade
- Reutilizar Node.js/TypeScript, MySQL e Prisma existentes no OMS. O Prisma schema e o Docker Compose do workspace confirmam o provider MySQL.
- Nenhum requisito adicional de compatibilidade de versão ou portabilidade para outros bancos foi definido na transcrição.
- O worker usa o mesmo banco e uma instância Prisma própria por processo.

Compliance
- Nenhuma norma regulatória ou requisito de compliance específico foi definido na transcrição.

Acessibilidade no frontend consumidor
- Não se aplica nesta fase: o escopo contempla endpoints de API, sem frontend ou dashboard.

---

### Arquitetura e abordagem

Abordagem
- Usar Transactional Outbox no MySQL existente. A API grava o evento elegível junto com a mudança do pedido na mesma transação. Um worker Node.js separado consulta a outbox por polling a cada 2 segundos e entrega eventos por HTTPS. Redis Streams foi considerada e descartada na reunião para evitar infraestrutura adicional.

Componentes
- Novo módulo de webhooks em `src/modules/webhooks`, seguindo o padrão existente de módulos, controllers, services, repositories, routes e schemas.
- Nova outbox MySQL e novas tabelas de configuração, histórico de entregas e DLQ; esses componentes fazem parte da feature e não são capacidades preexistentes.
- Nova entry point `src/worker.ts`, com PrismaClient próprio, e comando de inicialização separado, ambos propostos na reunião.
- Integração no fluxo de mudança de status do serviço de pedidos, usando o cliente transacional para publicar evento com snapshot.
- Reuso de AppError, logger Pino, middleware de erros, schemas Zod e `requireRole`, com códigos de erro do módulo prefixados por `WEBHOOK_`.

Integrações
- Integração interna com a mudança de status do pedido, histórico, estoque, MySQL, Prisma e autenticação JWT já existentes.
- Integração de saída via POST HTTPS com os endpoints fornecidos pelos clientes B2B.
- A documentação para clientes no portal de desenvolvedor foi mencionada como ação do Product Manager; o portal visual não faz parte da implementação da feature.

### Decisões e trade-offs

#### Decisão: usar Transactional Outbox no MySQL existente em vez de broker dedicado
- **Justificativa:** manter evento e mudança de status atomicamente consistentes sem adicionar Redis Cluster ou outra infraestrutura de mensageria nesta versão.
- **Trade-off:** polling exige consultas periódicas; o custo dessas consultas é consequência técnica inferida, não quantificada na reunião. O intervalo de 2 segundos, por si só, não garante entrega em menos de 10 segundos.

#### Decisão: enviar por worker separado e assíncrono, não durante a transação do pedido
- **Justificativa:** endpoints de clientes lentos ou indisponíveis não devem bloquear mudanças de status de pedidos.
- **Trade-off:** exige um processo adicional e a entrega não é síncrona com a operação do pedido.

#### Decisão: usar polling de 2 segundos em vez de triggers para notificar o worker
- **Justificativa:** a reunião registrou que triggers MySQL não notificam diretamente um processo externo; polling atende ao objetivo sem mecanismo improvisado.
- **Alternativa descartada:** usar triggers para acordar um processo externo.
- **Trade-off:** custo de consultas periódicas é inferido e não foi quantificado; o polling não assegura isoladamente o limite inferior a 10 segundos.

#### Decisão: entrega at-least-once identificada por UUID, em vez de exactly-once
- **Justificativa:** exactly-once exigiria coordenação entre a plataforma e o sistema consumidor; `X-Event-Id` permite ao cliente deduplicar.
- **Trade-off:** duplicatas podem ocorrer e o consumidor precisa deduplicar; frequência e efeitos específicos não foram quantificados.

#### Decisão: usar secret exclusiva por endpoint com HMAC-SHA256 e rotação
- **Justificativa:** permitir verificação de origem e integridade sem uma secret global; a secret anterior permanece válida por 24 horas para facilitar a migração do cliente.
- **Trade-off:** a janela de sobreposição mantém duas secrets válidas. A reunião não definiu qual assina novos eventos durante esse período. A rotação foi definida para facilitar a migração, não como resposta completa a vazamento.

#### Decisão: armazenar snapshot do payload na outbox no momento da mudança
- **Justificativa:** evento deve representar o estado do pedido no instante da mudança, mesmo que ele mude posteriormente.
- **Trade-off:** eventual aumento de armazenamento é consequência técnica inferida, não discutida expressamente na reunião.

#### Decisão: manter tabela DLQ separada
- **Justificativa:** preservar payload, motivo da falha e timestamp como evidência para diagnóstico e replay, mantendo a outbox principal voltada aos eventos ativos.
- **Trade-off:** complexidade adicional de persistência e transferência entre tabelas é consequência técnica inferida, não mencionada diretamente na reunião.

#### Decisão complementar: interpretar o limite como cinco retentativas adicionais à tentativa inicial
- **Justificativa:** esclarecimento confirmado durante a elaboração deste PRD; a reunião afirmou cinco tentativas, mas não especificou se a inicial entrava na contagem.
- **Trade-off:** eventos podem permanecer em retentativa pelos intervalos definidos antes de serem classificados como falha permanente. DLQ e replay não garantem entrega bem-sucedida.

#### Decisão: iniciar com um único worker
- **Justificativa:** processamento dos eventos por ordem de criação e expectativa de ordenação por pedido na arquitetura inicial.
- **Trade-off:** ordenação por pedido é uma propriedade esperada, não garantia absoluta; falhas e retentativas podem afetar recebimento. Não há garantia de ordenação global e o processamento paralelo fica fora desta versão.

---

### Dependências

#### técnica: MySQL e Prisma existentes
São a persistência utilizada pelo OMS e serão reutilizados. Worker, outbox e DLQ são novos componentes da feature, não dependências preexistentes.

#### técnica: autenticação JWT e autorização por role existentes
A transcrição confirma autenticação JWT para os endpoints e role `ADMIN` para replay. O workspace contém middleware JWT e `requireRole`. A associação do usuário autenticado ao `customer_id` precisa ser definida antes do deploy para impedir acesso entre clientes.

#### externa: endpoint HTTPS fornecido pelo cliente
Os clientes cadastram endpoints HTTPS para receber notificações. A classificação como dependência externa é uma interpretação documental da integração; a transcrição não usa essa classificação. A disponibilidade do endpoint é variável e pode falhar mesmo após o cadastro.

#### organizacional: revisão de segurança
A revisão de Sofia deve ocorrer antes do deploy. A reunião reservou pelo menos dois dias úteis para revisar principalmente HMAC e geração de secrets.

---

### Riscos e mitigação

#### RISK-001 Endpoint do cliente indisponível ou lento
- **Probabilidade:** não estimada
- **Impacto:** atraso ou falha permanente na entrega de notificações.
- **Mitigação:**
  - Timeout HTTP de 10 segundos.
  - Cinco retentativas adicionais com backoff de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas. A quantidade adicional foi esclarecida durante a entrevista; a reunião não definiu se a tentativa inicial entrava nas cinco.
  - Persistir falha na DLQ com payload, motivo e timestamp.
- **Plano de contingência:** replay manual por administrador. Replay não garante entrega bem-sucedida.

#### RISK-002 Recebimento ou processamento duplicado de notificações
- **Probabilidade:** não estimada
- **Impacto:** consumidor pode processar mais de uma vez o mesmo evento; efeitos específicos não foram detalhados nem quantificados na reunião.
- **Mitigação:**
  - Enviar o mesmo `X-Event-Id` UUID nas tentativas do mesmo evento.
  - O consumidor deduplica pelo identificador, conforme responsabilidade discutida na reunião.
- **Plano de contingência:** não especificado na transcrição; a deduplicação pelo consumidor é a orientação explicitamente discutida.

#### RISK-003 Recebimento de notificações fora de ordem para um mesmo pedido
- **Probabilidade:** não estimada
- **Impacto:** inferência técnica: o consumidor pode processar estados desatualizados do pedido.
- **Mitigação:**
  - Worker único consultando eventos por ordem de criação, como propriedade esperada da arquitetura inicial.
- **Plano de contingência:** pendente de definição.
- **Risco residual:** falhas, retentativas e tempos de resposta podem comprometer a ordem de recebimento. Essa possibilidade é inferência técnica, não cenário explicitamente analisado na reunião.

#### RISK-004 Acesso não autorizado a configurações ou históricos de outros clientes
- **Probabilidade:** não estimada
- **Impacto:** inferência técnica: possível exposição de informações ou alteração indevida de configurações de outro cliente.
- **Mitigação:** pendente de definição, incluindo validação do vínculo entre usuário e `customer_id`.
- **Plano de contingência:** pendente de definição.
- **Causa e ressalva:** a reunião define JWT e informa que `customer_id` é fornecido na requisição, mas não especifica autorização por cliente. Isso é uma lacuna de especificação, não confirmação de vulnerabilidade existente; resolver antes do deploy.

#### RISK-005 Exposição ou comprometimento da secret de webhook
- **Probabilidade:** não estimada
- **Impacto:** inferência técnica: terceiro com secret comprometida pode produzir notificações com assinaturas aparentemente legítimas para o endpoint afetado.
- **Mitigação:**
  - Usar secret exclusiva por endpoint e assinatura HMAC-SHA256.
  - Disponibilizar rotação com sobreposição de validade da secret anterior por 24 horas.
- **Plano de contingência:** rotação está disponível; revogação emergencial e demais procedimentos de resposta a incidentes permanecem pendentes.
- **Risco residual e rastreabilidade:** foi relatada exposição anterior de secret em logs de cliente. A sobreposição de 24 horas foi definida para facilitar migração, não como resposta completa a vazamento; durante a janela, a secret antiga permanece válida.

#### RISK-006 Sobrecarga do endpoint do cliente por alto volume de notificações
- **Probabilidade:** não estimada
- **Impacto:** inferência técnica: degradação de desempenho, aumento de latência, timeouts e possíveis falhas de entrega.
- **Mitigação:**
  - Observar o comportamento do volume e avaliar posteriormente a necessidade de rate limiting; isso é diretriz futura, não controle implementado nesta versão.
- **Plano de contingência:** pendente de definição.
- **Causa e risco residual:** rate limiting de saída ficou fora do escopo. A reunião mencionou como exemplo 50 mudanças de status em um minuto, sem definir limite ou critério de ativação.

---

### Critérios de aceitação
Checklist objetivo que define se a feature está pronta.

- [ ] **AC-001 Atomicidade:** para mudança com endpoint inscrito, pedido, histórico, estoque e evento elegível são gravados na mesma transação; falha na outbox reverte a transação. Sem endpoint elegível, nenhum evento é criado.
- [ ] **AC-002 Cadastro:** usuário autenticado cadastra endpoint HTTPS e status válidos; a plataforma cria configuração e devolve secret exclusiva. Autorização do usuário para o `customer_id` informado deve ser definida antes da validação completa.
- [ ] **AC-003 Entrega:** worker faz polling a cada 2 segundos e envia por HTTPS com timeout de 10 segundos e payload de até 64 KB. Menos de 10 segundos é objetivo em operação normal, não garantia absoluta.
- [ ] **AC-004 Retentativas e DLQ:** após a tentativa inicial, são feitas até cinco retentativas adicionais nos intervalos confirmados; depois da última falha, evento é preservado em DLQ com payload, motivo e timestamp. Replay não garante sucesso.
- [ ] **AC-005 Assinatura e rotação:** cada endpoint tem secret exclusiva; notificações usam HMAC-SHA256 do corpo; rotação mantém secret anterior válida por 24 horas e depois a invalida. Qual secret assina novos eventos durante a sobreposição permanece pendente; a verificabilidade completa desse aspecto depende do design técnico.
- [ ] **AC-006 Gerenciamento:** usuários autenticados podem listar, editar e remover configurações, e URLs cadastradas ou alteradas devem usar HTTPS. A remoção não deve gerar novos eventos, como comportamento inferido; tratamento de eventos já enfileirados e autorização por cliente permanecem pendentes.
- [ ] **AC-007 Histórico:** `GET /webhooks/:id/deliveries` retorna até 100 entregas com sucesso/falha, payload, resposta e tempo de resposta; acesso deve ser autorizado para o cliente. Lista vazia é comportamento complementar, não explicitamente discutido. Validação completa depende de definir autorização por cliente.
- [ ] **AC-008 Replay administrativo:** `POST /admin/webhooks/dead-letter/:id/replay` exige role `ADMIN` via `requireRole`, recoloca evento pendente na outbox e registra o administrador solicitante. Formato e armazenamento da auditoria, tratamento de replay duplicado e identidade do `event_id` após replay permanecem pendentes.
- [ ] **AC-009 Worker separado:** worker executa em processo distinto da API, usa PrismaClient próprio para o mesmo banco, consulta a outbox e opera como instância única na primeira versão.
- [ ] **AC-010 Filtragem:** apenas endpoints ativos inscritos no status de destino recebem eventos criados na outbox; sem endpoint elegível, não se insere evento.
- [ ] **AC-011 Contrato:** payload JSON contém `event_id`, `event_type` (`order.status_changed`), timestamp ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e campos básicos como `total_cents`; não inclui itens. Headers incluem `X-Event-Id`, `X-Webhook-Id`, `X-Signature`, `X-Timestamp` e `Content-Type: application/json`.
- [ ] **AC-012 At-least-once:** entrega duplicada é possível; o mesmo evento mantém o identificador `X-Event-Id` para deduplicação pelo consumidor. A plataforma não garante exactly-once.
- [ ] **AC-013 Limite de payload:** payload acima de 64 KB é reconhecido como erro, não é enviado nem truncado. Destino do evento após a rejeição permanece pendente de definição técnica.
- [ ] **AC-014 Ordenação:** worker único processa eventos pela ordem de criação; documentar isso apenas como propriedade esperada da arquitetura inicial, sem garantia absoluta por pedido ou global, especialmente diante de falhas e retentativas.

---

### Testes e validação

Tipos de teste obrigatórios
- Testes unitários para regras de negócio, validações, filtros de status, lógica de assinatura e política de retentativas.
- Testes de integração para persistência transacional e integração entre os componentes da feature, incluindo rollback quando a gravação do evento falha.

Estratégia de validação
- Executar testes automatizados unitários e de integração antes da integração e da implantação.
- O workspace já utiliza Vitest e possui testes de integração da API com Supertest e Prisma; essa constatação é evidência do projeto existente, não decisão da reunião.
- Não foi confirmada CI, TDD, QA guiado, teste de carga ou segurança como tipo obrigatório.
