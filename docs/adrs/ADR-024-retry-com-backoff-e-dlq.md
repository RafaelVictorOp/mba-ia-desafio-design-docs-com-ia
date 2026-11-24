---
status: accepted
date: 2026-11-22
---

# ADR-024: Aplicar retry com backoff finito e DLQ separada

## Contexto e problema

Endpoints de clientes podem ficar temporariamente indisponíveis, responder com falha transitória ou exceder o timeout. Descartar a entrega na primeira falha reduz a confiabilidade; mantê-la em retry indefinidamente deixa eventos presos e aumenta o backlog sem um estado terminal claro.

A reunião definiu cinco retentativas após a tentativa inicial, com intervalos de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas. Depois da última falha, o evento deve ser preservado em uma tabela DLQ separada, com payload, motivo da falha e timestamp, permitindo diagnóstico e replay administrativo.

## Fatores de decisão

- Indisponibilidades de clientes podem durar minutos ou horas.
- Eventos não podem permanecer pendentes indefinidamente.
- Falhas permanentes precisam conservar evidências para diagnóstico e replay.
- A outbox principal deve permanecer focada nos eventos entregáveis.
- O sistema oferece entrega at-least-once; uma retentativa pode produzir duplicata e mantém o mesmo `X-Event-Id` conforme a ADR-020.

## Opções consideradas

- **Backoff finito e DLQ separada:** executar cinco retentativas adicionais em 1 min, 5 min, 30 min, 2 h e 12 h; depois persistir a falha permanentemente na DLQ.
- **Retry indefinido com backoff:** continuar reagendando enquanto o endpoint falhar, sem estado terminal automático.
- **Falha permanente após a primeira tentativa:** não reagendar e encaminhar imediatamente o evento à DLQ.

## Resultado da decisão

**Opção escolhida:** aplicar cinco retentativas adicionais após a tentativa inicial, nos intervalos de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas. Se a última retentativa falhar, mover o evento para uma tabela DLQ separada, preservando ao menos o payload, o motivo da falha e o timestamp.

Falhas transitórias devem ser reagendadas sem alterar a identidade do evento. O replay da DLQ é uma operação administrativa separada e não reinicia automaticamente um ciclo infinito de tentativas.

### Consequências

- **Positiva:** cobre indisponibilidades curtas e prolongadas por uma janela de aproximadamente 15 horas.
- **Positiva:** estabelece um limite operacional e impede que eventos permaneçam em retry indefinidamente.
- **Positiva:** a DLQ separada preserva evidências e mantém a leitura da outbox principal concentrada em trabalho entregável.
- **Negativa:** uma recuperação posterior à última retentativa exige replay administrativo.
- **Negativa:** eventos podem permanecer pendentes por muitas horas e aumentar o backlog durante indisponibilidades amplas.
- **Negativa:** retentativas podem gerar entregas duplicadas; consumidores precisam deduplicar por `X-Event-Id`.

### Confirmação

Verificar em testes que uma falha transitória agenda exatamente os cinco intervalos definidos; que sucesso em qualquer tentativa encerra o ciclo; que a falha final cria uma entrada DLQ com os dados mínimos; e que nenhuma falha é descartada silenciosamente se a persistência na DLQ falhar.

## Prós e contras das opções

### Backoff finito e DLQ separada

- **Pró:** equilibra tolerância a indisponibilidade, limite de recursos e capacidade de diagnóstico.
- **Contra:** exige estados persistentes, agendamento e operação de DLQ/replay.

### Retry indefinido com backoff

- **Pró:** maximiza a chance de entrega automática mesmo após indisponibilidades muito longas.
- **Contra:** pode manter eventos pendentes para sempre, ampliar o backlog e ocultar falhas permanentes.

### Falha permanente após a primeira tentativa

- **Pró:** reduz a complexidade do worker e encerra rapidamente cada processamento.
- **Contra:** trata indisponibilidades transitórias como permanentes e aumenta a necessidade de replay manual.

## Mais informações

- Transcrição: [`TRANSCRICAO.md`](../../TRANSCRICAO.md), [09:14]-[09:19] e resumo em [09:48].
- PRD: [`docs/PRD.md`](../PRD.md), requisitos FR-006 e FR-008.
- RFC: [`docs/RFC.md`](../RFC.md), seções “Requisitos e restrições” e “Retentativas, DLQ e replay”.
- ADR relacionada: [ADR-020](./ADR-020-entrega-at-least-once-com-event-id.md), identidade estável e deduplicação na entrega at-least-once.
