# ADR-006: Estratégia de autenticação local com JWT

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-001](../PLATFORM/ADR-001-autenticacao-de-requisicoes-com-jwt.md), [ADR-002](../USERS/ADR-002-hash-de-senhas-com-bcrypt.md)

## Contexto e Declaração do Problema

O fluxo observado mantém credenciais localmente: a senha é armazenada como hash com bcrypt e comparada durante o login. Após autenticação bem-sucedida, a API emite um JWT assinado com identidade e papel do usuário. O cliente apresenta esse token como Bearer; a API o valida e usa a identidade e o papel para proteger recursos. Registro e login são públicos, enquanto a consulta do usuário autenticado exige autenticação.

Esse conjunto conecta o ciclo de credenciais, a emissão e validação de tokens e a autorização. O código comprova o comportamento, mas não registra a necessidade que motivou a escolha, os requisitos que a orientaram ou se ela foi aprovada como decisão arquitetural.

## Fatores de Decisão

- O fluxo atual autentica usuários por credenciais locais e verifica a senha armazenada como hash.
- A identidade e o papel acompanham o token e são usados na autenticação e autorização das requisições.
- A estratégia é compartilhada entre o fluxo de autenticação e as rotas que exigem acesso autenticado.
- [NECESSITA DE INFORMAÇÃO: requisitos e motivação que justificaram credenciais locais, bcrypt e JWT, incluindo as alternativas avaliadas.]

## Opções Consideradas

1. **Credenciais locais com bcrypt e JWT Bearer:** estratégia observada no código, incluindo autenticação por senha, emissão e validação do token e autorização por papel.

Não há evidência disponível de outras opções avaliadas; portanto, nenhuma alternativa é apresentada como se tivesse sido considerada.

## Resultado da Decisão

Este ADR propõe registrar a estratégia atualmente observada, sem atribuir uma motivação histórica não documentada. O status permanece **Proposto** até que os responsáveis confirmem se a estratégia deve ser adotada formalmente e mantida.

- [NECESSITA DE INFORMAÇÃO: confirmação da decisão e dos requisitos que determinam os papéis e os dados de identidade representados no token.]

## Prós e Contras das Opções

**Opção 1 — Credenciais locais com bcrypt e JWT Bearer**

**Prós**

- Integra o cadastro, o login e o acesso autenticado em uma estratégia local.
- Permite que a API associe identidade e papel às requisições e aplique autorização por papel.
- A representação pública do usuário não expõe a senha nem seu hash.

**Contras e limitações documentais**

- O contrato e o ciclo de vida do token dependem de configuração compartilhada entre emissão e validação.
- A documentação consultada não estabelece políticas de revogação ou rotação do segredo, nem explica a expiração adotada.
- [NECESSITA DE INFORMAÇÃO: política de expiração, revogação e rotação do segredo aplicável ao ambiente.]

## Consequências

- Mudanças na autenticação precisam considerar conjuntamente credenciais locais, emissão e validação de tokens e autorização por papel.
- O acesso a recursos protegidos exige o envio, pelo cliente, de JWT no esquema Bearer.
- A evolução dos dados de identidade e papel no token deve permanecer coerente entre emissão e validação.
- [NECESSITA DE INFORMAÇÃO: consequências operacionais e requisitos de evolução esperados, inclusive compatibilidade e gestão de sessões.]

## Referências

- `src/modules/users/user.service.ts` — criação do hash de senha e representação pública do usuário.
- `src/modules/auth/auth.service.ts` — verificação da senha e emissão do token após login.
- `src/middlewares/auth.middleware.ts` — validação do token e autorização por papel.
- `src/modules/auth/auth.routes.ts` — rotas públicas de cadastro e login e rota autenticada de consulta do usuário.
- `src/config/env.ts` — configuração do segredo e da expiração do token.
