---
status: proposed
date: 2026-11-08
---

# ADR-025: Preservar a identidade do evento e criar identidade por tentativa no replay

## Contexto e problema

A entrega at-least-once usa um `event_id` estável para permitir deduplicação do consumidor, decisão registrada em [ADR-020](./ADR-020-entrega-at-least-once-com-event-id.md). Para o replay da DLQ, o RFC propõe preservar o payload snapshot e o `event_id` original, mas gerar um novo `attempt_id`/`delivery_id`, atualizar `X-Timestamp` e recalcular a assinatura. A transcrição não registra essa semântica específica de replay; ela permanece proposta.

## Fatores de decisão

- O replay reenvia o mesmo evento de negócio, não uma nova mudança de status.
- O consumidor precisa reconhecer um replay do mesmo evento para aplicar deduplicação.
- A plataforma precisa distinguir e auditar tentativas individuais.
- A secret ativa e o timestamp da tentativa podem diferir daqueles usados na entrega inicial.

## Opções consideradas

- **Preservar payload e `event_id`, com novo identificador de tentativa:** proposta documentada no RFC.
- As fontes não registram uma alternativa específica de identidade para replay; não se presume que criar um novo evento de negócio tenha sido considerado.

## Resultado da decisão

**Opção proposta:** reutilizar o payload snapshot e o `event_id` original no replay, atribuindo um novo `attempt_id`/`delivery_id` e recalculando timestamp e HMAC para a tentativa atual, porque isso preserva a identidade de negócio e permite auditar cada envio separadamente.

### Consequências

- **Positiva:** o consumidor pode deduplicar entregas originais e replays pela mesma identidade de evento.
- **Positiva:** cada execução continua distinguível no histórico e na auditoria.
- **Negativa:** a implementação precisa manter invariantes distintas para identidade de evento e identidade de tentativa, incluindo concorrência de replay.
- **Negativa:** erro de tamanho ou endpoint excluído deve deixar a DLQ preservada, sem mutação destrutiva.

### Confirmação

Verificar que replay mantém byte a byte o snapshot persistido e o `event_id`; cada replay elegível gera um identificador de tentativa novo, timestamp atual e assinatura calculada com a secret ativa. Testes concorrentes devem demonstrar que no máximo uma nova tentativa ativa é criada para o mesmo item DLQ.

## Prós e contras das opções

### Payload e `event_id` originais com nova identidade de tentativa

- **Pró:** mantém a identidade do evento de negócio e possibilita deduplicação consistente pelo consumidor.
- **Contra:** exige controle concorrente e histórico separado por tentativa para evitar que replay altere ou oculte a evidência original.

## Mais informações

- RFC: [`docs/RFC.md`](../../../RFC.md), seções “Requisitos e restrições”, “Replay e identidade” e “Estratégia de validação”.
- ADR relacionado: [ADR-020](./ADR-020-entrega-at-least-once-com-event-id.md), decisão aceita sobre entrega at-least-once e identidade estável do evento.
- Mapeamento de escopo: [`mapping.md`](../../../adrs/adrs_sistema/mapping.md), módulos ORDERS e DATA.
- **PENDENTE DE VALIDAÇÃO:** confirmar com os responsáveis que a identidade de negócio deve permanecer estável entre entrega inicial e replay e definir a retenção do payload, histórico e tentativas.
