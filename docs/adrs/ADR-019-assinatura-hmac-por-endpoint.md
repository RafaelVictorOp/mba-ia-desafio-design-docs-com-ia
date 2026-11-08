---
status: accepted
date: 2026-11-08
---

# ADR-019: Assinar webhooks outbound com HMAC-SHA256 por endpoint

## Contexto e problema

Os eventos outbound contêm dados de pedidos e são enviados a endpoints controlados pelos clientes. O destinatário precisa verificar a origem e a integridade do corpo. A transcrição registra explicitamente HMAC-SHA256 sobre o corpo, secret exclusiva por endpoint e HTTPS obrigatório. Não há implementação de webhook ou armazenamento de secrets no código consultado.

Esta decisão trata da autenticação e integridade das chamadas outbound; não altera a autenticação JWT usada nas requisições à API nem define o armazenamento seguro das secrets.

## Fatores de decisão

- O destinatário precisa verificar origem e integridade do request.
- O comprometimento de uma secret não deve expor todos os endpoints.
- O transporte deve proteger a comunicação entre o OMS e o destinatário.
- O mecanismo não pode depender do JWT usado para autenticar usuários da API.

## Opções consideradas

- **HMAC-SHA256 com secret exclusiva por endpoint e HTTPS:** opção registrada como decisão na transcrição.
- As fontes não registram outras opções de autenticação outbound avaliadas.

## Resultado da decisão

**Opção escolhida:** assinar o corpo de cada request com HMAC-SHA256 usando uma secret exclusiva do endpoint destinatário e exigir HTTPS, porque o consumidor pode validar autenticidade e integridade sem reutilizar uma credencial global.

### Consequências

- **Positiva:** o destinatário pode verificar a origem e detectar alteração do corpo; o isolamento de secrets limita o alcance de uma exposição.
- **Negativa:** produtor e consumidor precisam compartilhar e proteger a secret correspondente; rotação e armazenamento em repouso requerem decisões adicionais.
- **Negativa:** HTTPS é obrigatório para os endpoints cadastrados.

### Confirmação

Verificar que a assinatura HMAC-SHA256 é calculada sobre o corpo efetivamente enviado, que cada endpoint usa sua própria secret, que requests a destinos HTTP são recusados e que valores de secret não são expostos em logs. A política de armazenamento e rotação deve ser validada separadamente.

## Prós e contras das opções

### HMAC-SHA256 com secret exclusiva por endpoint e HTTPS

- **Pró:** permite validar a integridade do corpo e restringe o impacto de uma secret comprometida ao endpoint correspondente.
- **Contra:** exige sincronizar a gestão da secret entre a plataforma e o destinatário; não há mecanismo de armazenamento protegido em repouso confirmado no workspace.

## Mais informações

- RFC: [`docs/RFC.md`](../../../RFC.md), seções “Requisitos e restrições”, “Alternativas consideradas” e “Segurança, observabilidade e operação”.
- Transcrição: [`TRANSCRICAO.md`](../../../../TRANSCRICAO.md), [09:19]-[09:24], decisão de HMAC-SHA256, secret por endpoint e TLS.
- Mapeamento de escopo: [`mapping.md`](../../../adrs/adrs_sistema/mapping.md), módulos AUTH e PLATFORM.
- Evidências do código atual: [`auth.middleware.ts`](../../../../src/middlewares/auth.middleware.ts#L1-L58) mostra autenticação de usuários da API, não autenticação de webhooks outbound; [`schema.prisma`](../../../../prisma/schema.prisma#L1-L53) não contém modelos de webhook.
- ADR relacionado: [ADR-001](../../../adrs/adrs_sistema/generated/PLATFORM/ADR-001-autenticacao-de-requisicoes-com-jwt.md), referente a requisições autenticadas da API; não substitui esta decisão de autenticação entre sistemas.
- O status `accepted` cobre a decisão de assinatura registrada na transcrição; a política de ativação imediata de secret proposta no RFC não está aprovada por este ADR.
