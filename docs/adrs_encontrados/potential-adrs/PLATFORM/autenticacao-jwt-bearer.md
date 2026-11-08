# Potencial ADR: Autenticação de requisições com JWT Bearer

**Módulo:** PLATFORM  
**Categoria:** Segurança/Arquitetura  
**Prioridade:** Documentação obrigatória (pontuação: 120)  
**Data da identificação:** 2026-11-08

---

## O que foi identificado

O middleware transversal `authenticate` exige um token no cabeçalho `Authorization` com esquema Bearer e valida o JWT usando `JWT_SECRET`. Após a validação, transfere `sub`, `email` e `role` do payload para `req.user`; tokens ausentes, inválidos ou expirados resultam em erro de autenticação. O middleware `requireRole` aplica autorização por roles `ADMIN` e `OPERATOR`. A configuração exige `JWT_SECRET` com pelo menos 16 caracteres e define `JWT_EXPIRES_IN` com padrão de oito horas.

Isso evidencia uma estratégia central de autenticação baseada em JWT Bearer no módulo PLATFORM. O fluxo que emite os tokens está fora do escopo desta análise (módulo AUTH); portanto, este candidato descreve somente a validação e o consumo transversal do token, sem concluir como o token é emitido ou quais claims adicionais são incluídas.

O histórico Git disponível mostra os arquivos no commit `init repository`, de 2026-06-24, sem modificações posteriores encontradas para esses arquivos. Essa evidência data a inclusão no repositório, mas não confirma a data, a motivação ou as alternativas consideradas para a decisão.

## Por que essa decisão pode merecer um ADR

- **Impacto:** o mecanismo é compartilhado por endpoints que exigem autenticação e autorização, conforme o mapeamento. Uma alteração pode afetar clientes da API e os módulos consumidores.
- **Trade-offs:** o código confirma a validação local de JWT com segredo compartilhado e a rejeição de tokens inválidos/expirados. Consequências específicas, como estratégia de revogação ou rotação do segredo, não são definidas pelos arquivos analisados.
- **Complexidade:** mudar o esquema de autenticação pode exigir alterações coordenadas na validação transversal, na emissão de tokens (fora deste escopo), na configuração e nos consumidores da API. A estimativa de duas a oito semanas é uma avaliação, não um dado histórico.
- **Conhecimento da equipe:** a estratégia afeta segurança e acesso a operações protegidas, tornando importante que a equipe compreenda as fronteiras entre autenticação e autorização.
- **Implicações futuras:** decisões sobre expiração, rotação de segredo, revogação e evolução das claims devem permanecer compatíveis entre emissão e validação. Esses pontos não são especificados pela evidência examinada.

**Regra dos três Es:** estrutural (validação centralizada de identidade e autorização), evidente (implementada em middleware compartilhado e configuração dedicada) e estável por expectativa técnica, embora a permanência da escolha não possa ser comprovada pelo histórico curto disponível.

**Pontuação:** base 70 por ser infraestrutura crítica de autenticação para uma API voltada a usuários; abrangência e impacto 20 por ser infraestrutura central; custo de mudança 15 (estimativa de duas a oito semanas); conhecimento da equipe 15 por ser importante em áreas específicas de segurança e acesso. **Total: 120/150.** A base de 70 é conservadora e a classificação como infraestrutura crítica decorre do papel transversal de autenticação observado no mapeamento.

## Evidências encontradas no código

### Arquivos principais

- [`src/middlewares/auth.middleware.ts`](../../../../../src/middlewares/auth.middleware.ts) - linhas 27-46 e 49-60
  - Exige o esquema Bearer, verifica o JWT com `env.JWT_SECRET`, preenche `req.user` e implementa autorização por roles.
- [`src/config/env.ts`](../../../../../src/config/env.ts) - linhas 8-10
  - Valida o segredo JWT e define a expiração padrão da configuração.
- [`src/shared/errors/http-errors.ts`](../../../../../src/shared/errors/http-errors.ts) - linhas 15-24
  - Define erros HTTP específicos para não autenticado e acesso proibido.

### Evidências de código

`src/middlewares/auth.middleware.ts:27-45`

```typescript
export const authenticate: RequestHandler = (req, _res, next) => {
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
};
```

`src/config/env.ts:8-10`

```typescript
  JWT_SECRET: z.string().min(16, 'JWT_SECRET must be at least 16 characters'),
  JWT_EXPIRES_IN: z.string().default('8h'),
});
```

### Análise de impacto

- **Introdução:** os arquivos aparecem no Git em 2026-06-24, no commit `init repository`; isso não comprova quando a decisão foi tomada.
- **Modificações:** um commit de inclusão foi encontrado para `auth.middleware.ts`; nenhuma alteração posterior consta no histórico disponível.
- **Última alteração:** 2026-06-24, commit `init repository`; a mensagem não registra tema técnico específico.
- **Abrangência:** 3 arquivos principais citados neste candidato, todos em PLATFORM; o mapeamento identifica a capacidade como reutilizada pela API e pelos módulos de domínio.
- **Temas recentes:** não há commits posteriores com temas verificáveis para os arquivos analisados.

## Questões para abordar no ADR formal (se criado)

- Qual problema motivou a adoção de JWT Bearer em vez de uma alternativa?
- Como a emissão de tokens em AUTH se alinha à validação de PLATFORM?
- Como são tratados rotação do segredo, revogação e expiração dos tokens?
- Quais claims são contratos entre emissão, middleware e consumidores?

## Potenciais ADRs relacionados

- Nenhum outro candidato de PLATFORM foi identificado com pontuação de 75 ou mais nesta análise.

## Observações adicionais

- A pasta padrão de ADRs existentes (`docs/adrs/generated/`) não existe; não foi possível comparar o candidato com ADRs formais anteriores. O diretório `docs/adrs/` contém apenas o mapeamento e o README.
- `JWT_EXPIRES_IN` é um detalhe subordinado à estratégia de autenticação e não foi proposto como ADR independente.
- A análise ficou restrita a PLATFORM; não foi examinado o código de emissão de tokens do módulo AUTH.
