---
status: accepted
date: 2026-11-08
---

# ADR-018: Registrar eventos em outbox MySQL e entregá-los por worker separado

## Contexto e problema

O fluxo atual de mudança de status atualiza pedido, estoque e histórico em uma transação Prisma, mas não publica eventos de webhook (`src/modules/orders/order.service.ts:126-171`). O RFC propõe registrar eventos elegíveis de forma atômica e não realizar chamadas a endpoints externos dentro do fluxo síncrono. A transcrição registra a escolha explícita de Transactional Outbox no MySQL, polling a cada dois segundos e processo worker separado.

Esta decisão limita-se ao transporte e ao processamento assíncrono dos eventos. A aceitação desta escolha registrada na reunião não aprova, por si só, todo o RFC, que permanece em `Draft`.

## Fatores de decisão

- O evento precisa ser confirmado ou revertido junto da mudança de status.
- A indisponibilidade ou latência do endpoint externo não deve bloquear a operação de pedidos.
- O MySQL e as transações Prisma já fazem parte da aplicação.
- O objetivo de iniciar a entrega em menos de dez segundos é operacional, não uma garantia absoluta.

## Opções consideradas

- **Transactional Outbox no MySQL com worker separado e polling:** grava o evento na transação do pedido e delega o envio ao processo worker.
- **Broker dedicado, como Redis Streams:** alternativa discutida, com infraestrutura e operação adicionais.
- **Envio HTTP síncrono:** rejeitado na reunião por acoplar o fluxo de pedidos a endpoints externos.
- **Manter o comportamento atual:** não notifica os clientes e não atende à necessidade descrita.

## Resultado da decisão

**Opção escolhida:** Transactional Outbox no MySQL com processo worker separado, porque a gravação no mesmo banco preserva a atomicidade do evento com a mudança de status sem incluir a chamada HTTP externa na transação. O worker consulta os pendentes por polling a cada dois segundos.

### Consequências

- **Positiva:** uma mudança de status confirmada tem registro durável correspondente; falha da transação reverte também a gravação do evento.
- **Positiva:** falhas ou lentidão dos destinatários não participam da requisição síncrona de mudança de status.
- **Negativa:** é necessário operar e monitorar um worker, polling e as tabelas de eventos; backlog pode exceder o objetivo de latência.
- **Negativa:** o MySQL passa a armazenar estado operacional de entrega, além dos dados de domínio.

### Confirmação

Verificar na implementação que o evento elegível é gravado pelo mesmo cliente transacional Prisma que atualiza pedido e histórico; que a chamada externa ocorre somente no processo worker separado; e que a consulta usa polling configurado para dois segundos.

## Prós e contras das opções

### Transactional Outbox no MySQL com worker separado

- **Pró:** usa a transação MySQL existente e desacopla a disponibilidade do destinatário do fluxo de pedidos.
- **Contra:** adiciona um processo, polling e operação de tabelas de eventos; polling e backlog não garantem início da entrega em menos de dez segundos.

### Broker dedicado

- **Pró:** oferece uma infraestrutura de mensageria dedicada.
- **Contra:** adiciona serviço e operação; a transação do pedido e a publicação precisam de coordenação para evitar dual-write.

### Envio HTTP síncrono

- **Pró:** evita a introdução inicial de um processo assíncrono.
- **Contra:** a latência e indisponibilidade do endpoint externo afetariam a mudança de status.

### Manter o comportamento atual

- **Pró:** não adiciona componentes nem altera o fluxo atual.
- **Contra:** mantém a consulta repetitiva de pedidos e não fornece as notificações solicitadas.

## Mais informações

- RFC: [`docs/RFC.md`](../../../RFC.md), seções “Proposta técnica” e “Alternativas consideradas”.
- Transcrição: [`TRANSCRICAO.md`](../../../../TRANSCRICAO.md), [09:06]-[09:11], decisões de outbox, polling e processo worker.
- Mapeamento de escopo: [`mapping.md`](../../../adrs/adrs_sistema/mapping.md), módulos ORDERS e DATA.
- Evidência do código: [`order.service.ts`](../../../../src/modules/orders/order.service.ts#L126-L171); [`schema.prisma`](../../../../prisma/schema.prisma#L1-L8).
- ADRs relacionados: [ADR-016](../../../adrs/adrs_sistema/generated/DATA/ADR-016-mysql-como-banco-relacional.md) e [ADR-017](../../../adrs/adrs_sistema/generated/DATA/ADR-017-prisma-como-orm.md).
- O status `accepted` aplica-se somente à decisão registrada explicitamente na transcrição; o RFC continua `Draft` e outros detalhes da feature seguem propostos ou pendentes.
