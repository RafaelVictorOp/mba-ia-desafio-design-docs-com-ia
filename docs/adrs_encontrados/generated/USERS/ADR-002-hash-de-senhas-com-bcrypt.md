# ADR-002: Hash de senhas com bcrypt

**Status:** Proposta
**Date:** 08-11-2026
**Related ADRs:** [ADR-006](../AUTH/ADR-006-estrategia-de-autenticacao-local-com-jwt.md)

## Contexto e Declaração do Problema

O fluxo de criação de usuários transforma a senha em um hash com bcrypt e fator de custo 10 antes de persistir o valor destinado ao hash. No login, a senha informada é verificada em relação ao hash persistido. O modelo de usuário prevê um campo para esse hash.

Essa escolha conecta a criação de usuários, a autenticação e os dados persistidos. As fontes confirmam o comportamento atual, mas não registram a motivação da escolha de bcrypt nem se o fator 10 corresponde a uma política deliberada.

## Fatores de Decisão

- Preservar compatibilidade entre o formato produzido no cadastro e o consumido no login.
- Armazenar e verificar o hash de senha no ciclo de vida de usuários e autenticação.
- Considerar o fator de custo observado, sem presumir que tenha sido validado contra requisitos operacionais.
- **[NECESSITA DE INFORMAÇÃO]** Qual necessidade e quais requisitos de segurança e desempenho orientaram a escolha de bcrypt?
- **[NECESSITA DE INFORMAÇÃO]** Como o fator 10 foi definido e sob quais critérios deverá ser revisto?

## Opções Consideradas

1. **bcrypt com fator de custo 10:** única opção comprovada pela implementação consultada.
2. **Alternativas:** não há evidência de outros algoritmos ou serviços de gestão de credenciais avaliados.

**[NECESSITA DE INFORMAÇÃO]** Quais alternativas foram efetivamente avaliadas e por que foram aceitas ou descartadas?

## Resultado da Decisão

Registrar como abordagem vigente o uso de bcrypt com fator de custo 10 para o hash de senhas de usuários, mantendo compatibilidade entre o valor persistido e sua verificação durante o login. Esta decisão descreve o estado demonstrado pelas fontes; elas não comprovam a motivação original nem uma avaliação formal de alternativas.

## Prós e Contras das Opções

- **bcrypt com fator 10**
  - **Prós:** o formato produzido para persistência é o mesmo consumido no fluxo de login, conforme a implementação consultada.
  - **Contras:** as fontes não demonstram por que esse algoritmo e esse fator foram escolhidos, nem sua adequação a requisitos mensurados.
- **Alternativas não documentadas**
  - **Prós e contras:** não podem ser comparados com as evidências disponíveis.

## Consequências

- Criação de usuários e autenticação dependem de manter compatibilidade com o formato de hash persistido.
- Uma mudança de algoritmo ou de política de custo pode afetar hashes existentes; as fontes não identificam estratégia de migração, rehash ou redefinição de senhas.
- **[NECESSITA DE INFORMAÇÃO]** Qual estratégia será adotada para tratar hashes existentes caso o algoritmo ou o fator de custo mude? As fontes também não informam a população de usuários nem uma política de atualização desses hashes.
- Não há dados que permitam concluir sobre o impacto de desempenho ou sobre a suficiência do fator de custo observado.

## Referências

1. `src/modules/users/user.service.ts` — hash de senha calculado com fator 10 antes da persistência.
2. `src/modules/users/user.repository.ts` — persistência do valor destinado ao hash no registro de usuário.
3. `src/modules/auth/auth.service.ts` — verificação da senha informada em relação ao hash persistido.
4. `prisma/schema.prisma` — campo de hash no modelo persistido de usuário.
5. `package.json` — dependência bcrypt declarada pela aplicação.