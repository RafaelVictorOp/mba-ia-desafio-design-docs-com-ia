# ADR-001: Autenticação de requisições com JWT

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-002](../USERS/ADR-002-hash-de-senhas-com-bcrypt.md), [ADR-006](../AUTH/ADR-006-estrategia-de-autenticacao-local-com-jwt.md)

## Contexto e Declaração do Problema

A API precisa validar a identidade de requisições protegidas e aplicar autorização de forma transversal. A evidência disponível mostra o uso de tokens JWT enviados como Bearer e autorização por roles `ADMIN` e `OPERATOR`. A emissão dos tokens pertence ao módulo AUTH e está fora do escopo desta decisão.

O problema registrado é como validar e consumir tokens de forma compartilhada, sem confundir autenticação com autorização. As fontes não comprovam a motivação original nem as alternativas consideradas na adoção.

## Fatores de Decisão

- A capacidade é transversal: uma alteração pode afetar clientes da API e módulos consumidores.
- A validação utiliza um segredo configurado com pelo menos 16 caracteres; a validade padrão configurada é de oito horas.
- A identidade consumida inclui identificador, e-mail e role; os níveis de autorização observados são `ADMIN` e `OPERATOR`.
- A emissão de tokens e o contrato entre emissão e validação não foram examinados nesta decisão.

## Opções Consideradas

- **JWT no esquema Bearer com autorização por roles:** opção identificável no comportamento atual e formalizada neste ADR.
- As fontes não registram outras opções comparadas. **[NECESSITA DE INFORMAÇÃO]** Quais alternativas foram efetivamente avaliadas e quais critérios orientaram a escolha?

## Resultado da Decisão

Formaliza-se retrospectivamente o uso transversal de JWT Bearer para autenticar requisições e de roles para autorizar o acesso. Tokens ausentes, inválidos ou expirados são rejeitados. A identidade usada pela aplicação deriva de identificador, e-mail e role presentes no token. A configuração observada exige segredo com no mínimo 16 caracteres e define validade padrão de oito horas.

Este status registra o comportamento evidenciado, não a data, a motivação ou o processo histórico da decisão.

## Prós e Contras das Opções

**JWT Bearer com autorização por roles**

- **Prós:** fornece um mecanismo compartilhado de autenticação e uma distinção explícita entre identidade autenticada e permissões por role.
- **Contras:** a validação depende de um segredo compartilhado; a compatibilidade das informações do token depende da integração com o emissor, que não foi analisada.

## Consequências

- Clientes da API e módulos consumidores dependem do formato de autenticação e das roles observadas; mudanças podem exigir coordenação com a emissão de tokens em AUTH.
- As fontes não definem políticas de rotação do segredo ou revogação de tokens. **[NECESSITA DE INFORMAÇÃO]** Como esses processos devem funcionar?
- A configuração registra validade padrão de oito horas, mas não documenta sua justificativa nem como esse prazo se alinha à emissão. **[NECESSITA DE INFORMAÇÃO]** Qual política de expiração deve ser mantida entre emissão e validação?
- O conjunto de claims observado não permite concluir se existem outras claims contratuais. **[NECESSITA DE INFORMAÇÃO]** Quais claims são garantidas pelo emissor e necessárias aos consumidores?

## Referências

- `docs\adrs\mapping.md` — visão dos módulos AUTH e PLATFORM e do uso transversal de autenticação.
- `src\middlewares\auth.middleware.ts` — comportamento de autenticação e autorização observado.
- `src\config\env.ts` — restrição do segredo e validade padrão configurada.
- `src\shared\errors\http-errors.ts` — distinção entre erros de autenticação e de autorização.
