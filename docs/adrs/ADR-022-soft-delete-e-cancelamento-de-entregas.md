---
status: proposed
date: 2026-11-08
---

# ADR-022: Preservar webhooks excluídos e cancelar entregas não reservadas

## Contexto e problema

A exclusão física de um endpoint pode eliminar a referência necessária para auditar entregas e eventos pendentes. O RFC propõe soft delete, cancelamento de eventos ainda não despachados e preservação de entregas concluídas e DLQ. O estado atual não possui modelos ou worker de webhook (`prisma/schema.prisma:1-53`; `package.json:1-20`).

O RFC define uma fronteira concorrente para que exclusão e worker não iniciem/cancelem a mesma entrega de forma ambígua. Existe uma janela entre reservar o despacho no banco e abrir a conexão HTTP: o estado persistido não comprova que bytes já foram enviados.

## Fatores de decisão

- A exclusão deve impedir novas inserções e cancelar entregas que ainda não foram reservadas para despacho.
- O histórico de entregas e a DLQ devem permanecer consultáveis para auditoria.
- Replay não deve ocorrer para endpoint excluído.
- A concorrência entre exclusão e reserva do despacho precisa ter uma regra observável e testável.

## Opções consideradas

- **Soft delete com cancelamento:** marcar o endpoint como excluído, impedir novas inserções, cancelar eventos pendentes ou em retry e preservar histórico e DLQ.
- **Hard delete:** remover fisicamente o endpoint, alternativa documentada no RFC, com perda de referência de auditoria e maior complexidade para eventos relacionados.

## Resultado da decisão

**Opção proposta:** usar soft delete e cancelar entregas que ainda não atingiram o estado de despacho, porque preserva a rastreabilidade e impede que novos claims ocorram depois da exclusão. A transição atômica para `DISPATCHING` é a reserva irrevogável do despacho — não a confirmação de que a conexão foi aberta ou que bytes foram enviados. Se o claim confirmar primeiro, a entrega não será cancelada e poderá iniciar ou chegar ao cliente depois do commit do soft delete.

### Consequências

- **Positiva:** mantém referência para entregas concluídas e DLQ, enquanto impede novos eventos para o endpoint excluído.
- **Negativa:** requer estados adicionais, coordenação transacional entre worker e exclusão e regras de retenção.
- **Negativa aceita na proposta:** a reserva `DISPATCHING` pode preceder o início efetivo do request; portanto, uma entrega reservada antes da exclusão ainda pode ser enviada após o soft delete.

### Semântica concorrente proposta

Worker e exclusão devem serializar o claim e a alteração do endpoint pelo mesmo registro/lock:

1. Se a exclusão confirmar primeiro, PENDING/RETRY são cancelados e o claim condicional falha.
2. Se o claim confirmar primeiro (`PENDING`/`RETRY` → `DISPATCHING`), a exclusão posterior não altera esse estado nem cancela a entrega; o HTTP pode iniciar ou concluir depois do soft delete.
3. `DISPATCHING` significa “despacho reservado e não mais cancelável”, não “requisição iniciada na rede”. Não se mantém transação aberta durante a chamada HTTP.

Testes concorrentes devem provar ambos os ordenamentos e explicitar a possibilidade aceita de envio após a exclusão no segundo caso. A aceitação formal dessa janela por Product/Security continua pendente antes do deploy. A recuperação de um `DISPATCHING` abandonado deve ser detalhada no FDD com lease/timeout ou mecanismo equivalente; por ser at-least-once, o resultado pode ser duplicado e o consumidor deve deduplicar por `event_id`.

## Prós e contras das opções

### Soft delete com cancelamento

- **Pró:** preserva rastreabilidade e permite cancelar entregas ainda não reservadas para despacho.
- **Contra:** adiciona estados e coordenação concorrente entre API e worker; uma entrega reservada pode iniciar após a exclusão.

### Hard delete

- **Pró:** simplifica a representação de existência e remoção do endpoint.
- **Contra:** perde referência de auditoria e complica chaves estrangeiras, histórico e eventos pendentes.

## Mais informações

- RFC: [`docs/RFC.md`](../../../RFC.md), seções “Requisitos e restrições”, “Exclusão” e “Exclusão versus worker concorrente”.
- Mapeamento de escopo: [`mapping.md`](../../../adrs/adrs_sistema/mapping.md), módulos ORDERS e DATA.
- Evidências do estado atual: [`schema.prisma`](../../../../prisma/schema.prisma#L1-L53) não contém entidades de webhook; [`package.json`](../../../../package.json#L1-L20) não declara processo worker.
- **PENDENTE DE APROVAÇÃO FORMAL:** Product/Security devem aceitar a janela operacional descrita acima antes do deploy. A semântica de desenho desta proposta está definida: `DISPATCHING` representa reserva não cancelável, não início literal da requisição na rede.
