---
status: proposed
date: 2026-11-08
---

# ADR-024: Ativar imediatamente a nova secret de webhook

## Contexto e problema

A assinatura HMAC depende de uma secret compartilhada com o destinatário. O RFC propõe que a nova secret se torne ativa imediatamente para todos os envios, inclusive retries, enquanto o destinatário pode verificar a secret anterior por até 24 horas. Para comprometimento, propõe revogação imediata sem período de graça. A transcrição confirma a rotação e a janela de 24 horas, mas não registra a política de ativação imediata do lado remetente; essa extensão permanece proposta.

O código atual não contém modelos de webhook nem mecanismo de proteção de secrets (`prisma/schema.prisma:1-53`).

## Fatores de decisão

- Uma secret potencialmente comprometida não deve continuar assinando novos requests.
- Retries devem usar a credencial ativa no momento do envio, não uma credencial congelada no snapshot.
- A plataforma não controla quando o destinatário atualiza sua configuração.
- O mecanismo de armazenamento, entrega única e revogação de secrets não está definido no workspace.

## Opções consideradas

- **Ativação imediata da nova secret:** novos requests e retries usam a nova secret assim que a rotação é concluída; o destinatário mantém a anterior disponível para verificação por até 24 horas.
- **Ativação após confirmação do receptor ou período de preparação:** reduz falhas de configuração do destinatário, mas mantém a chave anterior como signatária por mais tempo. O RFC registra que esta opção não foi escolhida para a proposta.

## Resultado da decisão

**Opção proposta:** ativar imediatamente a nova secret para todas as novas tentativas de envio e limitar a validade de verificação da secret anterior no destinatário a 24 horas, porque reduz o período em que a chave anterior assina eventos. Esta proposta não é uma aprovação formal do RFC nem define armazenamento ou distribuição segura das secrets.

### Consequências

- **Positiva:** novos requests deixam de depender da secret anterior logo após a rotação.
- **Negativa:** entregas podem falhar se o destinatário ainda não estiver configurado para validar a nova secret; isso pode levar a retries ou DLQ.
- **Negativa:** a API precisa entregar a nova secret de forma controlada e definir proteção em repouso, revogação e tratamento de requests já iniciados.

### Confirmação

Verificar que, após a rotação, novos envios e retries usam somente a secret ativa; que o consumidor consegue validar as secrets permitidas durante a janela de transição; e que revogação emergencial não permite uso futuro da secret comprometida. Não registrar secrets em logs ou respostas não autorizadas.

## Prós e contras das opções

### Ativação imediata da nova secret

- **Pró:** reduz imediatamente o uso da chave anterior pelo remetente.
- **Contra:** pode causar falha de validação no consumidor até que ele instale a nova secret.

### Ativação após confirmação ou preparação do receptor

- **Pró:** oferece ao destinatário tempo para preparar a troca e reduz a chance de quebra imediata de entrega.
- **Contra:** prolonga o período em que a secret anterior pode assinar novos requests.

## Mais informações

- RFC: [`docs/RFC.md`](../../../RFC.md), seções “Requisitos e restrições”, “Rotação e entrega”, “Riscos, premissas e mitigação” e “Perguntas abertas e decisões pendentes”.
- Transcrição: [`TRANSCRICAO.md`](../../../../TRANSCRICAO.md), [09:20]-[09:22], decisão de rotação de secret e janela de 24 horas.
- Mapeamento de escopo: [`mapping.md`](../../../adrs/adrs_sistema/mapping.md), módulos AUTH, PLATFORM e DATA.
- Evidência do estado atual: [`schema.prisma`](../../../../prisma/schema.prisma#L1-L53) não define configuração ou armazenamento de secrets de webhook.
- **PENDENTE DE VALIDAÇÃO:** qual mecanismo disponível protege a secret em repouso e a entrega apenas uma vez ao cliente? Qual contrato autoriza revogação emergencial e como tratar requests já iniciados?
- ADR relacionado: [ADR-019](./ADR-019-assinatura-hmac-por-endpoint.md), que registra o mecanismo de assinatura sem aprovar esta política de rotação imediata.
