---
status: accepted
date: 2026-11-08
---

# ADR-020: Usar entrega at-least-once com identidade estável do evento

## Contexto e problema

Falhas de rede e respostas perdidas podem fazer uma entrega ser repetida. A transcrição registra a escolha de semântica at-least-once e o envio de um UUID em `X-Event-Id` para que o consumidor deduplique eventos repetidos. O RFC também descreve retries e replay, mas a política de identidade por tentativa é uma proposta adicional e está tratada em [ADR-025](./ADR-025-identidade-de-evento-e-tentativa-no-replay.md).

## Fatores de decisão

- O envio HTTP e a confirmação de resposta não podem formar uma única transação com o banco do destinatário.
- O consumidor precisa identificar o mesmo evento quando a plataforma tenta entregá-lo novamente.
- Exigir exactly-once demandaria coordenação entre remetente e destinatário, não estabelecida nesta proposta.
- A reunião não definiu garantia de ordenação global.

## Opções consideradas

- **Entrega at-least-once com `event_id` estável e deduplicação no consumidor:** opção explicitamente decidida na transcrição.
- **Exactly-once:** alternativa discutida, rejeitada por exigir coordenação entre os dois lados e maior complexidade.

## Resultado da decisão

**Opção escolhida:** entrega at-least-once com UUID de evento transportado em `X-Event-Id`, porque permite retentativas e deixa ao consumidor a deduplicação por identidade sem prometer exactly-once.

### Consequências

- **Positiva:** falhas transitórias podem ser tratadas com novas tentativas sem perder a identidade de negócio do evento.
- **Negativa:** o consumidor pode receber duplicatas e precisa deduplicar usando `X-Event-Id`.
- **Negativa:** esta decisão não garante ordenação global entre eventos.

### Confirmação

Verificar que o UUID é criado para o evento persistido e que novas entregas do mesmo evento carregam o mesmo `X-Event-Id`; documentar a semântica at-least-once para consumidores e não anunciar exactly-once nem ordenação global.

## Prós e contras das opções

### Entrega at-least-once com `event_id` estável

- **Pró:** permite retentativas e oferece uma chave explícita para deduplicação do consumidor.
- **Contra:** consumidores precisam tratar duplicatas e implementar deduplicação.

### Exactly-once

- **Pró:** evitaria duplicatas visíveis ao consumidor, caso fosse possível garantir a semântica entre os dois sistemas.
- **Contra:** a transcrição registra que isso exigiria coordenação de ambos os lados e seria mais complexo; não foi escolhido.

## Mais informações

- RFC: [`docs/RFC.md`](../../../RFC.md), seções “Requisitos e restrições”, “Replay e identidade” e “Riscos, premissas e mitigação”.
- Transcrição: [`TRANSCRICAO.md`](../../../../TRANSCRICAO.md), [09:24]-[09:26], decisão de at-least-once e `X-Event-Id`.
- Mapeamento de escopo: [`mapping.md`](../../../adrs/adrs_sistema/mapping.md), módulos ORDERS e DATA.
- O tratamento de identidade por tentativa e replay é uma proposta separada em [ADR-025](./ADR-025-identidade-de-evento-e-tentativa-no-replay.md).
