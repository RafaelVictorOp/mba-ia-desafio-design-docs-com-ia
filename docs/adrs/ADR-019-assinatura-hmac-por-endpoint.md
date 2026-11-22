---
status: accepted
date: 2026-11-08
---

# ADR-019: Assinatura HMAC e ciclo de vida de secrets por endpoint

## Contexto e problema

Os eventos outbound contêm dados de pedidos e são enviados a endpoints controlados pelos clientes. O destinatário precisa verificar a origem e a integridade do corpo. A transcrição registra HMAC-SHA256 sobre o corpo, secret exclusiva por endpoint, HTTPS obrigatório e rotação com validade paralela da secret anterior por 24 horas. A secret é gerada pela plataforma e devolvida na criação; o contrato de devolução durante a rotação não foi definido. Não há implementação de webhook ou armazenamento de secrets no código consultado.

Esta ADR registra a decisão aceita de assinatura e os requisitos confirmados para o ciclo de vida da secret. Não altera a autenticação JWT usada nas requisições à API nem define o armazenamento seguro das secrets. A ativação imediata da nova secret e a revogação emergencial são propostas, não decisões aceitas.

## Fatores de decisão

- O destinatário precisa verificar origem e integridade do request.
- O comprometimento de uma secret não deve expor todos os endpoints.
- O transporte deve proteger a comunicação entre o OMS e o destinatário.
- O mecanismo não pode depender do JWT usado para autenticar usuários da API.
- A rotação deve permitir ao destinatário migrar para uma nova secret dentro da janela de 24 horas acordada.

## Opções consideradas

- **HMAC-SHA256 com secret exclusiva por endpoint e HTTPS:** opção registrada como decisão na transcrição.
- As fontes não registram outras opções de autenticação outbound avaliadas.
- Para o momento de ativação da nova secret, o RFC considera ativação imediata ou ativação após confirmação/preparação do destinatário; a reunião não escolheu entre essas opções.

## Resultado da decisão

**Decisão aceita:** assinar o corpo de cada request com HMAC-SHA256 usando uma secret exclusiva do endpoint destinatário, exigir HTTPS e, após rotação, manter a secret anterior válida em paralelo por 24 horas. A secret é gerada pela plataforma e devolvida na criação do webhook. Essas decisões permitem ao consumidor verificar autenticidade e integridade, isolam o impacto de uma secret exposta e oferecem uma janela de migração.

### Ativação da nova secret — proposta não aprovada

O RFC propõe ativar imediatamente a nova secret para todos os novos envios e retries, usando a credencial ativa no momento de cada tentativa, enquanto o destinatário pode verificar a secret anterior durante até 24 horas. Isso reduz o período em que a secret antiga assina requests, mas pode interromper entregas se o destinatário ainda não estiver configurado. A reunião não definiu o momento de ativação no remetente; portanto, essa proposta não faz parte da decisão aceita desta ADR.

Alternativamente, a ativação poderia aguardar confirmação do destinatário ou um período de preparação. Isso reduz o risco de falhas de configuração, mas prolonga o uso da secret anterior para assinatura. A opção preferida permanece pendente de aprovação explícita.

### Revogação emergencial — proposta e questões em aberto

A existência e o contrato de uma revogação emergencial não foram decididos. Se essa operação for adotada, propõe-se restringi-la a usuário autenticado com role `ADMIN`, usando `authenticate` e `requireRole('ADMIN')`, por analogia à proteção definida para replay manual da DLQ. Essa extensão não deve ser interpretada como decisão da reunião sobre revogação.

Permanecem pendentes: período de graça, gestão e distribuição da secret substituta, divulgação da secret na rotação, armazenamento protegido em repouso e tratamento de requests já iniciados. O código atual não contém modelos de webhook nem mecanismo de proteção de secrets.

### Consequências

- **Positiva:** o destinatário pode verificar a origem e detectar alteração do corpo; o isolamento de secrets limita o alcance de uma exposição.
- **Negativa:** produtor e consumidor precisam compartilhar e proteger a secret correspondente; armazenamento em repouso e divulgação durante a rotação requerem definição.
- **Negativa:** HTTPS é obrigatório para os endpoints cadastrados.
- **Risco da proposta de ativação imediata:** envios podem falhar até que o destinatário esteja configurado para validar a nova secret, levando a retries ou DLQ.
- **Risco da proposta de revogação:** sem política para requests em voo e credencial substituta, a operação pode interromper entregas sem recuperação definida.

### Confirmação

Verificar que a assinatura HMAC-SHA256 é calculada sobre o corpo efetivamente enviado, que cada endpoint usa sua própria secret, que requests a destinos HTTP são recusados, que a secret é devolvida na criação e que valores de secret não são expostos em logs. Confirmar a validade paralela da secret anterior por 24 horas. Antes de implementar ativação imediata ou revogação emergencial, aprovar seus contratos e resolver as questões em aberto acima.

## Prós e contras das opções

### HMAC-SHA256 com secret exclusiva por endpoint e HTTPS

- **Pró:** permite validar a integridade do corpo e restringe o impacto de uma secret comprometida ao endpoint correspondente.
- **Contra:** exige sincronizar a gestão da secret entre a plataforma e o destinatário; não há mecanismo de armazenamento protegido em repouso confirmado no workspace.

### Ativação imediata da nova secret (proposta)

- **Pró:** reduz imediatamente o uso da chave anterior pelo remetente.
- **Contra:** pode causar falha de validação no consumidor até que ele instale a nova secret.

### Ativação após confirmação ou preparação do destinatário (alternativa)

- **Pró:** oferece ao destinatário tempo para preparar a troca e reduz a chance de quebra imediata de entrega.
- **Contra:** prolonga o período em que a secret anterior pode assinar novos requests.

## Mais informações

- RFC: [`docs/RFC.md`](../RFC.md), seções “Requisitos e restrições”, “Rotação e entrega”, “Alternativas consideradas” e “Segurança, observabilidade e operação”.
- Transcrição: [`TRANSCRICAO.md`](../../TRANSCRICAO.md), [09:20]-[09:24] para assinatura, rotação e HTTPS; [09:31] para geração e devolução da secret na criação; [09:36] para o precedente de autorização do replay administrativo.
- Mapeamento do estado atual: [`mapping.md`](../adrs_encontrados/mapping.md), módulos AUTH, DATA e PLATFORM.
- Evidências do código atual: [`auth.middleware.ts`](../../src/middlewares/auth.middleware.ts) mostra autenticação de usuários da API e autorização por role, não autenticação de webhooks outbound; [`schema.prisma`](../../prisma/schema.prisma) não contém modelos de webhook nem armazenamento de secrets.
- A decisão `accepted` cobre somente assinatura, transporte e janela de validade paralela de 24 horas confirmados na reunião. Ativação imediata e revogação emergencial permanecem propostas não aprovadas.
