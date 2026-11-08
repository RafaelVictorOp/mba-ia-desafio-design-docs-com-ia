---
status: proposed
date: 2026-11-08
---

# ADR-023: Classificar payload excedente na criação do evento

## Contexto e problema

O RFC define um limite de 65.536 bytes para a serialização JSON em UTF-8 e propõe que eventos excedentes sejam classificados como falha permanente na criação do snapshot. O comportamento proposto é persistir o payload original e o diagnóstico diretamente na DLQ, sem HTTP, truncamento, fragmentação ou retry, e sem reverter pedido apenas por causa do tamanho. Não há outbox ou DLQ de webhook no schema atual (`prisma/schema.prisma:1-53`).

O RFC reconhece que uma falha real de banco ao persistir a DLQ pode abortar a transação do pedido; não propõe ignorar falhas de persistência.

## Fatores de decisão

- O limite é medido sobre os bytes do JSON serializado em UTF-8, não sobre número de caracteres.
- O destinatário não deve receber payload acima do limite nem payload truncado.
- Eventos inválidos conhecidos não devem percorrer retries antes de serem classificados.
- O pedido e o diagnóstico de excesso devem manter rastreabilidade, sem converter apenas o tamanho excedente em falha da mudança de status.

## Opções consideradas

- **Validar na criação do snapshot, com nova validação no worker e replay:** opção recomendada no RFC; grava o evento excedente na DLQ junto da transação de status.
- **Validar somente no worker:** evento excedente permanece na outbox até ser processado e classificado.
- **Validar somente no envio ou replay:** não impede que a criação encaminhe evento acima do limite para processamento.

## Resultado da decisão

**Opção proposta:** medir o tamanho do snapshot serializado na criação e persistir diretamente na DLQ quando ultrapassar 65.536 bytes, porque isso permite classificação terminal imediata e evita envio ou retry de um evento conhecido como excedente. A alteração de tamanho não deve, por si só, reverter pedido, histórico ou estoque.

### Consequências

- **Positiva:** eventos excedentes são identificados antes do worker e ficam disponíveis para diagnóstico sem serem enviados.
- **Positiva:** o caminho de sucesso não depende de retries para detectar um tamanho já conhecido na criação.
- **Negativa:** serialização e cálculo de bytes passam a integrar o fluxo transacional; falha ao persistir a DLQ permanece uma falha de persistência e pode abortar a transação.
- **Negativa:** API, worker e replay precisam aplicar a mesma regra de medição.

### Confirmação

Verificar casos de 65.535, 65.536 e 65.537 bytes em UTF-8. Confirmar que o caso acima do limite produz DLQ com payload original e diagnóstico, não gera request HTTP nem retry, e não causa rollback apenas por exceder o tamanho. Também verificar que falha de persistência é propagada conforme a garantia transacional aprovada.

## Prós e contras das opções

### Validar na criação do snapshot e novamente no worker/replay

- **Pró:** classifica o evento excedente imediatamente e impede que percorra a fila de entregas ou retries.
- **Contra:** exige serialização e cálculo consistentes na API e mantém a escrita da DLQ dentro da transação do pedido.

### Validar somente no worker

- **Pró:** evita fazer a classificação completa no produtor.
- **Contra:** atrasa o diagnóstico quando o worker está indisponível e exige mover o evento posteriormente para a DLQ.

### Validar somente no envio ou replay

- **Pró:** concentra a validação perto da chamada HTTP.
- **Contra:** deixa o evento passar sem validação suficiente antes do processamento e não satisfaz a recomendação de classificação antecipada.

## Mais informações

- RFC: [`docs/RFC.md`](../../../RFC.md), seções “Requisitos e restrições”, “Recomendação provisória para payload excedente”, “Alternativas consideradas” e “Perguntas abertas e decisões pendentes”.
- Mapeamento de escopo: [`mapping.md`](../../../adrs/adrs_sistema/mapping.md), módulos ORDERS e DATA.
- Evidência do estado atual: [`schema.prisma`](../../../../prisma/schema.prisma#L1-L53) não define outbox nem DLQ de webhooks.
- **PENDENTE DE VALIDAÇÃO:** confirmar se uma falha de banco ao gravar o payload excedente na DLQ deve continuar abortando a transação do pedido. O RFC registra essa consequência e aponta outbox intermediária com promoção assíncrona como alternativa ainda não escolhida.
