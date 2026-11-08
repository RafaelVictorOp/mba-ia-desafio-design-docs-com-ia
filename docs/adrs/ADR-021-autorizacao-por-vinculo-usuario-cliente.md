---
status: proposed
date: 2026-11-08
---

# ADR-021: Autorizar a gestão de webhooks pelo vínculo usuário-cliente

## Contexto e problema

As operações de gestão de webhooks recebem ou identificam um `customer_id`, mas esse identificador não prova que o usuário autenticado pode acessar o cliente. O middleware existente autentica JWT e autoriza roles `ADMIN` e `OPERATOR`; o schema Prisma não contém relação entre `User` e `Customer` (`src/middlewares/auth.middleware.ts:1-58`; `prisma/schema.prisma:25-53`).

O RFC propõe autorização por vínculo explícito usuário-cliente e rejeita o `customer_id` informado como prova de permissão. A fonte autoritativa e o mecanismo concreto desse vínculo continuam indefinidos; portanto, esta decisão é proposta, não aprovada.

## Fatores de decisão

- O controle precisa impedir que um usuário opere sobre webhooks de outro cliente apenas por conhecer seu identificador.
- JWT e role identificam/autorizam o usuário em nível geral, mas não demonstram por si só autorização sobre um cliente.
- O mecanismo proposto precisa se apoiar numa fonte de vínculo verificável e autoritativa.
- A relação usuário-cliente não existe no schema observado.

## Opções consideradas

- **Vínculo explícito usuário-cliente:** autenticar com JWT e autorizar cada operação pelo vínculo com o cliente.
- **Qualquer usuário autenticado:** mais simples, mas não impede acesso cruzado entre clientes.
- **Somente `ADMIN` para todo CRUD:** evita a autorização por cliente, mas centraliza todas as operações e não oferece autonomia aos usuários dos clientes. O replay administrativo permanece restrito a `ADMIN` em qualquer alternativa.

## Resultado da decisão

**Opção proposta:** exigir autorização verificável do usuário sobre o cliente associado a cada operação de gestão de webhook, porque a autenticação e o `customer_id` recebido não são suficientes para provar permissão sobre o recurso.

### Consequências

- **Positiva:** reduz o risco de acesso cruzado entre clientes e separa autenticação geral de autorização por recurso.
- **Negativa:** requer uma fonte autoritativa de vínculo e trabalho de integração ou evolução do modelo de identidade.
- **Negativa:** a feature não pode ser implementada com segurança até que a fonte e a regra de vínculo sejam definidas.

### Confirmação

Antes da implementação, identificar a fonte autoritativa que determina a relação usuário-cliente. Verificar por testes que um usuário sem vínculo não pode consultar nem alterar configurações, entregas ou secrets de outro cliente, mesmo enviando seu `customer_id`.

## Prós e contras das opções

### Vínculo explícito usuário-cliente

- **Pró:** expressa a autorização no nível do recurso e evita confiar somente em um identificador fornecido pela requisição.
- **Contra:** depende de mecanismo de vínculo que não está modelado nem identificado no estado atual.

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
- **PENDENTE DE VALIDAÇÃO:** qual sistema ou relação autoritativa deve determinar o vínculo entre usuário e `customer_id`? O RFC identifica essa lacuna e o schema atual não a resolve.
