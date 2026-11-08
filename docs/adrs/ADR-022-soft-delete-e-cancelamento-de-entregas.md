---
status: proposed
date: 2026-11-08
---

# ADR-022: Preservar webhooks excluídos e cancelar entregas ainda não iniciadas

## Contexto e problema

A exclusão física de um endpoint pode eliminar a referência necessária para auditar entregas e eventos pendentes. O RFC propõe soft delete, cancelamento de eventos ainda não despachados e preservação de entregas concluídas e DLQ. O estado atual não possui modelos ou worker de webhook (`prisma/schema.prisma:1-53`; `package.json:1-20`).

O RFC também propõe tratar `DISPATCHING` como ponto de linearização entre exclusão e despacho, mas registra uma janela entre reservar a entrega e abrir a conexão HTTP. A escolha permanece sujeita a validação.

## Fatores de decisão

- A exclusão deve impedir novas inserções e envios ainda não iniciados.
- O histórico de entregas e a DLQ devem permanecer consultáveis para auditoria.
- Replay não deve ocorrer para endpoint excluído.
- A concorrência entre exclusão e início de HTTP precisa ter uma regra observável e testável.

## Opções consideradas

- **Soft delete com cancelamento:** marcar o endpoint como excluído, impedir novas inserções, cancelar eventos pendentes ou em retry e preservar histórico e DLQ.
- **Hard delete:** remover fisicamente o endpoint, alternativa documentada no RFC, com perda de referência de auditoria e maior complexidade para eventos relacionados.

## Resultado da decisão

**Opção proposta:** usar soft delete e cancelar entregas que ainda não atingiram o estado de despacho, porque preserva a rastreabilidade e evita iniciar novas chamadas após a exclusão. O estado `DISPATCHING` é a fronteira operacional proposta, não uma decisão já validada.

### Consequências

- **Positiva:** mantém referência para entregas concluídas e DLQ, enquanto impede novos eventos para o endpoint excluído.
- **Negativa:** requer estados adicionais, coordenação transacional entre worker e exclusão e regras de retenção.
- **Negativa:** tratar a reserva `DISPATCHING` como início do request pode permitir que a exclusão ocorra antes do primeiro byte HTTP.

### Confirmação

Validar a fronteira entre claim e abertura da conexão com Product/Security. Testes concorrentes devem provar que uma exclusão confirmada antes do ponto de início impede o envio, que itens ainda pendentes ou em retry são cancelados e que entregas concluídas e DLQ são preservadas.

## Prós e contras das opções

### Soft delete com cancelamento

- **Pró:** preserva rastreabilidade e permite cancelar entregas que ainda não foram despachadas.
- **Contra:** adiciona estados e coordenação concorrente entre API e worker.

### Hard delete

- **Pró:** simplifica a representação de existência e remoção do endpoint.
- **Contra:** perde referência de auditoria e complica chaves estrangeiras, histórico e eventos pendentes.

## Mais informações

- RFC: [`docs/RFC.md`](../../../RFC.md), seções “Requisitos e restrições”, “Exclusão” e “Exclusão versus worker concorrente”.
- Mapeamento de escopo: [`mapping.md`](../../../adrs/adrs_sistema/mapping.md), módulos ORDERS e DATA.
- Evidências do estado atual: [`schema.prisma`](../../../../prisma/schema.prisma#L1-L53) não contém entidades de webhook; [`package.json`](../../../../package.json#L1-L20) não declara processo worker.
- **PENDENTE DE VALIDAÇÃO:** Product/Security devem confirmar se `DISPATCHING` pode representar “requisição iniciada” apesar da janela entre claim e conexão HTTP, ou definir outra semântica observável.
