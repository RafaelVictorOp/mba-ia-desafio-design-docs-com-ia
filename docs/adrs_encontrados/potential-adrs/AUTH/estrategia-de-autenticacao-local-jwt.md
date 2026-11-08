# Potencial ADR: Estratégia de autenticação local com bcrypt e JWT Bearer

**Módulo:** AUTH  
**Categoria:** Segurança / Arquitetura  
**Prioridade:** Documentação obrigatória (pontuação: 105/150)  
**Data da identificação:** 2026-11-08

---

## Contexto de ADRs existentes

Não foram encontrados ADRs formais em `docs/adrs/generated/`; `docs/adrs/README.md` contém instruções de armazenamento, mas não registra uma decisão.

Há duas análises de candidatos relacionadas no índice existente, não decisões formalizadas. O conjunto lexical de referência desta decisão é: autenticação, credenciais locais, bcrypt, JWT, Bearer, roles e hash.

- **PLATFORM — Autenticação de requisições com JWT:** similaridade lexical indicativa de **57,1%** (4 palavras comuns — autenticação, JWT, Bearer e roles, em 7 palavras-chave desta decisão). Refere-se à validação transversal e ao consumo dos tokens; este candidato também aborda a emissão em AUTH e o ciclo de senha.
- **USERS — Hash de senhas com bcrypt no ciclo de vida de usuários:** similaridade lexical indicativa de **42,9%** (3 palavras comuns — bcrypt, hash e autenticação, em 7 palavras-chave desta decisão). Refere-se ao hash de senhas; este candidato reúne esse aspecto com emissão e validação de tokens.

Esses percentuais são comparação lexical indicativa, não comprovam equivalência semântica. As análises são relacionadas e a cronologia de identificação é a mesma (2026-11-08); recomenda-se que, ao formalizar decisões, se avalie consolidar os aspectos de emissão em AUTH, validação em PLATFORM e hash em USERS num registro coerente, evitando decisões duplicadas. Não há evidência de que uma decisão formal substitua outra.

## O que foi identificado

O fluxo implementado mantém credenciais localmente: a criação de usuário gera um hash de senha com `bcrypt` e 10 rounds; no login, a senha fornecida é comparada com o hash armazenado. Após autenticação bem-sucedida, o serviço emite um JWT assinado com `JWT_SECRET`, contendo `sub`, `email` e `role`, e informa o tempo de expiração configurado.

O cliente envia o token no esquema Bearer. O middleware verifica a assinatura e a validade do JWT, associa `id`, `email` e `role` à requisição, e oferece `requireRole` para autorização baseada nas roles `ADMIN` e `OPERATOR`. As rotas de registro e login são públicas, enquanto `/me` exige autenticação. Hash de senha, JWT, autorização por role e a exposição de dados públicos compõem a estratégia de autenticação observada; a combinação é candidata a uma decisão arquitetural, não uma decisão formal já aprovada.

O histórico Git disponível registra os arquivos centrais do AUTH no commit inicial de 2026-06-24 (`init repository`); os arquivos consultados não mostram commits posteriores. Esse histórico não esclarece motivação nem alternativas consideradas.

## Por que essa decisão pode merecer um ADR

- **Impacto:** é uma capacidade central do OMS e integra o AUTH ao serviço de usuários, ao middleware compartilhado e ao transporte HTTP.
- **Trade-offs:** a solução usa credenciais locais e tokens Bearer assinados; as configurações de segredo e expiração são centralizadas no ambiente. Não foram observadas alternativas documentadas no código ou no histórico consultado.
- **Complexidade:** alterar a estratégia implica coordenar emissão e validação de credenciais/tokens, autorização, configuração e testes de integração; a estimativa de esforço abaixo é julgamento analítico, não um prazo registrado no repositório.
- **Conhecimento da equipe:** entender os claims e o contrato Bearer é importante para mudanças nas rotas protegidas e no middleware de autorização.
- **Implicações futuras:** a decisão influencia como a aplicação autentica requisições e representa a identidade/role do usuário; os efeitos de uma troca de estratégia devem ser avaliados em conjunto com os módulos dependentes.
- **Contexto temporal:** o padrão está presente no histórico consultado desde 2026-06-24 e não há alterações posteriores registradas para os arquivos centrais examinados.

### Pontuação

| Dimensão | Pontos | Justificativa |
|---|---:|---|
| Base — infraestrutura crítica para o domínio | 70 | Autenticação e autorização são capacidades centrais da API OMS observadas no módulo AUTH. |
| Abrangência e impacto | 10 | Impacto principal no AUTH e sua integração próxima com usuários/API; pontuação conservadora para 1–2 módulos diretamente envolvidos. |
| Custo de mudança | 10 | Estimativa analítica de 1–2 semanas para trocar a abordagem e adaptar emissão, validação, integração e testes; não é duração comprovada por histórico. |
| Conhecimento da equipe | 15 | Conhecimento importante nas áreas que implementam e mantêm autenticação e autorização. |
| **Total** | **105/150** | **Must-document** pelo limiar geral de 100 pontos. |

## Evidências encontradas no código

### Arquivos principais

- [`src/modules/auth/auth.service.ts`](../../../../../src/modules/auth/auth.service.ts), linhas 31–49
  - Compara a senha com `bcrypt`, emite JWT após login e inclui `sub`, `email` e `role` no token assinado.
- [`src/middlewares/auth.middleware.ts`](../../../../../src/middlewares/auth.middleware.ts), linhas 28–56
  - Extrai o token Bearer, verifica o JWT, popula `req.user` e implementa autorização por role.
- [`src/modules/auth/auth.routes.ts`](../../../../../src/modules/auth/auth.routes.ts), linhas 10–12
  - Define registro e login sem `authenticate` e protege a rota `/me`.
- [`src/modules/users/user.service.ts`](../../../../../src/modules/users/user.service.ts), linhas 16–47
  - Gera hash bcrypt com 10 rounds e converte o usuário para uma representação pública sem `passwordHash`.
- [`src/config/env.ts`](../../../../../src/config/env.ts), linhas 8–9
  - Exige `JWT_SECRET` com pelo menos 16 caracteres e define `JWT_EXPIRES_IN` com padrão `8h`.
- [`tests/auth.test.ts`](../../../../../tests/auth.test.ts), linhas 6–16 e 42–62
  - Testa registro sem expor senha/hash e autenticação por senha.

### Evidências de código

Trecho de `src/modules/auth/auth.service.ts:36–49`:

```typescript
const ok = await bcrypt.compare(input.password, user.passwordHash);
if (!ok) {
  throw new UnauthorizedError('Invalid credentials');
}
const token = this.signToken(user.id, user.email, user.role);
return {
  user: UserService.toPublic(user),
  tokens: { accessToken: token, expiresIn: env.JWT_EXPIRES_IN, tokenType: 'Bearer' },
};
```

Trecho de `src/middlewares/auth.middleware.ts:28–42`:

```typescript
const header = req.headers.authorization;
if (!header || !header.toLowerCase().startsWith('bearer ')) {
  next(new UnauthorizedError('Missing or invalid Authorization header'));
  return;
}

const token = header.slice(7).trim();
if (!token) {
  next(new UnauthorizedError('Missing bearer token'));
  return;
}

try {
  const payload = jwt.verify(token, env.JWT_SECRET) as JwtPayload;
  req.user = { id: payload.sub, email: payload.email, role: payload.role };
  next();
} catch {
  next(new UnauthorizedError('Invalid or expired token'));
}
```

### Análise de impacto

- **Introdução:** 2026-06-24, no commit `init repository`, segundo o histórico dos arquivos centrais consultados.
- **Modificações:** os arquivos `src/modules/auth/auth.service.ts` e `src/middlewares/auth.middleware.ts` aparecem no commit inicial; não há commits posteriores registrados para esses caminhos na consulta realizada.
- **Última alteração:** 2026-06-24, commit inicial. O tema da mensagem é a inicialização do repositório, sem explicação específica da escolha de autenticação.
- **Abrangência:** evidências principais nos arquivos do AUTH, middleware compartilhado, serviço de usuários, configuração de ambiente e teste de autenticação; a decisão integra o sistema pela API HTTP.
- **Temas recentes:** não há alterações posteriores observadas nos arquivos consultados; não foi possível inferir temas de manutenção recentes.

### Alternativas (se observáveis)

Não foram encontradas alternativas explicitadas nos arquivos de código ou nos commits consultados. A ausência de alternativas documentadas não demonstra quais opções foram avaliadas originalmente.

## Questões para abordar no ADR formal (se criado)

- Qual necessidade do OMS levou à escolha de credenciais locais, bcrypt e JWT Bearer?
- Por que esses mecanismos foram escolhidos em vez das alternativas consideradas à época?
- Quais requisitos orientaram a inclusão de `role` nos claims e a validação no middleware?
- Quais consequências de longo prazo e necessidades de evolução devem ser registradas?

## Potenciais ADRs relacionados

- [PLATFORM — Autenticação de requisições com JWT](../PLATFORM/autenticacao-jwt-bearer.md) — decisão relacionada sobre validação transversal do token.
- [USERS — Hash de senhas com bcrypt no ciclo de vida de usuários](../USERS/hash-de-senhas-com-bcrypt.md) — decisão relacionada sobre armazenamento e verificação de senha.

## Observações adicionais

- A pontuação de custo de mudança é uma estimativa analítica, não um fato verificável no histórico.
- O padrão de 10 rounds bcrypt e o padrão de expiração `8h` são detalhes da estratégia observada; não foram separados em candidatos independentes.
- Este arquivo é uma análise de candidato, não um ADR formal aprovado.
