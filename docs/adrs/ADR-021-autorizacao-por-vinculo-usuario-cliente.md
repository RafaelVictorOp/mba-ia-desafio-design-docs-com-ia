---
status: proposed
date: 2026-11-08
---

# ADR-021: Autorizar a gestão de webhooks pelo vínculo usuário-cliente

## Contexto e problema

As operações de gestão de webhooks recebem ou identificam um `customer_id`, mas esse identificador não prova que o usuário autenticado pode acessar o cliente. O middleware existente autentica JWT e autoriza roles `ADMIN` e `OPERATOR`; o schema Prisma não contém relação entre `User` e `Customer` (`src/middlewares/auth.middleware.ts:1-58`; `prisma/schema.prisma:25-53`).

O RFC propõe autorização por vínculo explícito usuário-cliente e rejeita o `customer_id` informado como prova de permissão. A fonte de verdade proposta é uma relação `CustomerMembership` persistida no MySQL, com chaves para usuário e cliente, estado ativo/revogado e unicidade do par. Cada operação sobre recurso de cliente consulta essa relação usando a identidade autenticada (`req.user.id`, derivada do JWT) e o `customer_id` do recurso. Concessões e revogações ficam restritas a provisionamento administrativo confiável, sem autoassociação. A tabela e o fluxo de provisionamento ainda não existem; o ADR permanece proposto até revisão/aprovação e implementação.

## Fatores de decisão

- O controle precisa impedir que um usuário opere sobre webhooks de outro cliente apenas por conhecer seu identificador.
- JWT e role identificam/autorizam o usuário em nível geral, mas não demonstram por si só autorização sobre um cliente.
- O mecanismo proposto precisa se apoiar numa fonte de vínculo verificável e autoritativa.
- A relação usuário-cliente não existe no schema observado.
- A associação deve ser revogável e consultada em cada operação para que uma revogação tenha efeito sem aguardar expiração de JWT.
- Concessões e revogações não podem ser controladas pelo próprio usuário que obtém acesso; precisam de provisionamento administrativo confiável e auditável.

## Opções consideradas

- **Vínculo explícito usuário-cliente:** autenticar com JWT e autorizar cada operação pelo vínculo com o cliente.
- **Qualquer usuário autenticado:** mais simples, mas não impede acesso cruzado entre clientes.
- **Somente `ADMIN` para todo CRUD:** evita a autorização por cliente, mas centraliza todas as operações e não oferece autonomia aos usuários dos clientes. O replay administrativo permanece restrito a `ADMIN` em qualquer alternativa.

## Resultado da decisão

**Opção proposta:** manter `CustomerMembership` como fonte de verdade no MySQL e exigir associação ativa entre a identidade autenticada (`req.user.id`, derivada do JWT) e o `customer_id` do recurso em cada operação de gestão, consulta de histórico ou rotação. A API nega por padrão quando a associação estiver ausente ou revogada; não confia isoladamente em `customer_id`, claims de cliente no token ou role geral. Um fluxo administrativo confiável concede/revoga associações e registra ator e horário; não há autoassociação. Replay permanece uma operação global separada, restrita a `ADMIN` e auditada conforme o requisito do RFC.

### Consequências

- **Positiva:** reduz o risco de acesso cruzado entre clientes e separa autenticação geral de autorização por recurso.
- **Negativa:** requer migration/modelo Prisma para a relação e implementação do provisionamento administrativo.
- **Negativa:** a feature não pode ser implementada com segurança até que a relação, provisionamento, checagem por recurso e testes estejam entregues e revisados.

### Confirmação

O ponto de desenho sobre a fonte de autorização está definido nesta proposta como `CustomerMembership` no MySQL. Antes do deploy, implementar o modelo e o provisionamento administrativo e testar que vínculo ausente/revogado impede consulta ou alteração de configurações, entregas e secrets de outro cliente, mesmo quando o usuário envia o `customer_id`; confirmar também que revogação passa a valer em operações subsequentes. O estado `proposed` permanece até aprovação formal.

## Prós e contras das opções

### Vínculo explícito usuário-cliente

- **Pró:** expressa a autorização no nível do recurso, evita confiar somente em identificador fornecido pela requisição e permite revogação consultada em cada operação.
- **Contra:** exige criar e governar uma relação `CustomerMembership` e seu fluxo administrativo, que não existem no estado atual.

### Qualquer usuário autenticado

- **Pró:** reutiliza diretamente o middleware de autenticação existente.
- **Contra:** não comprova que o usuário pode operar sobre o `customer_id` informado e permite acesso cruzado se não houver outra política.

### Somente `ADMIN` para todo CRUD

- **Pró:** reutiliza a autorização por role já disponível.
- **Contra:** centraliza a gestão em administradores e não atende à autonomia prevista para usuários dos clientes.

## Mais informações

- RFC: [`docs/RFC.md`](../../../RFC.md), seções “Requisitos e restrições”, “Alternativas consideradas” e “Perguntas abertas e decisões pendentes”.
- Mapeamento de escopo: [`mapping.md`](../../../adrs/adrs_sistema/mapping.md), módulos AUTH, CUSTOMERS e PLATFORM.
- Evidências do código: [`auth.middleware.ts`](../../../../src/middlewares/auth.middleware.ts#L1-L58); [`schema.prisma`](../../../../prisma/schema.prisma#L25-L53).
- ADRs relacionados: [ADR-001](../../../adrs/adrs_sistema/generated/PLATFORM/ADR-001-autenticacao-de-requisicoes-com-jwt.md) e [ADR-006](../../../adrs/adrs_sistema/generated/AUTH/ADR-006-estrategia-de-autenticacao-local-com-jwt.md) tratam de autenticação JWT e roles; esta proposta complementa essas decisões com autorização por recurso.
- **Fonte e regra de autorização definidas como proposta:** `CustomerMembership` no MySQL, vínculo ativo consultado por operação e concessão/revogação por provisionamento administrativo. Não é uma implementação existente.
- **Validação de implementação pendente:** migration, governança do provisionamento, auditoria de concessão/revogação, testes de isolamento por cliente e aprovação formal do ADR.
