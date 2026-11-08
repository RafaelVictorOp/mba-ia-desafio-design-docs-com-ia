# Potencial ADR: Hash de senhas com bcrypt no ciclo de vida de usuários

**Módulo:** USERS  
**Categoria:** Segurança  
**Prioridade:** Documentação obrigatória (pontuação: 105/150)  
**Data da identificação:** 2026-11-08

---

## O que foi identificado

O módulo `USERS` transforma a senha recebida em um hash usando `bcrypt` com fator de custo configurado em 10, antes de persistir o campo `passwordHash`. O fluxo de login em `AUTH` consome esse mesmo formato por meio de `bcrypt.compare`. O schema Prisma define `passwordHash` como campo do modelo `User`. Isso evidencia uma escolha de armazenamento e verificação de credenciais que atravessa o ciclo de vida de usuários e a autenticação.

O código comprova a implementação observada, mas não registra por que `bcrypt` foi escolhido, quais alternativas foram avaliadas ou se o fator 10 reflete uma política deliberada. O histórico Git mostra esses arquivos no commit `init repository`, de 2026-06-24, sem commits posteriores nos arquivos consultados; isso indica apenas a primeira inclusão rastreável no repositório, não a data nem a motivação da decisão.

## Por que essa decisão pode merecer um ADR

- **Impacto:** determina como credenciais são armazenadas no banco e verificadas no login; a troca afeta USERS, AUTH e os dados persistidos.
- **Trade-offs:** o código usa hashing com custo computacional configurado (`10`); a intenção desse valor não está documentada. A necessidade de equilibrar custo de verificação e proteção contra tentativa de adivinhação é uma consequência técnica inferida, não uma motivação comprovada do projeto.
- **Complexidade:** alterar o algoritmo ou o formato de hash pode exigir compatibilidade temporária com hashes existentes, rehash em login ou redefinição de senhas. São implicações possíveis, não mecanismos já identificados no código.
- **Conhecimento da equipe:** pessoas que alterem criação de usuários ou autenticação precisam preservar a compatibilidade entre `bcrypt.hash` e `bcrypt.compare` e evitar exposição do hash.
- **Implicações futuras:** uma mudança de algoritmo ou política de custo deve considerar os hashes persistidos e o caminho de login. Não há evidência no repositório sobre população de usuários ou política de atualização desses hashes.

**Pontuação:** base 70 (infraestrutura crítica ao domínio de autenticação); abrangência 10 (um ou dois módulos diretamente envolvidos); custo de mudança 10 (estimativa de uma a duas semanas para coordenar os pontos de criação, verificação e compatibilidade dos dados; sem dados de implantação, a estimativa é incerta); conhecimento da equipe 15 (importante nas áreas de usuários e autenticação). **Total: 105/150**, classificado em `must-document/`. A classificação crítica é uma avaliação arquitetural desta análise, não uma motivação declarada no código.

## Evidências encontradas no código

### Arquivos principais

- [`src/modules/users/user.service.ts`](../../../../../src/modules/users/user.service.ts), linhas 16 e 21-33
  - Define o fator de custo e calcula o hash antes de chamar o repositório.
- [`src/modules/users/user.repository.ts`](../../../../../src/modules/users/user.repository.ts), linhas 14-16
  - Persiste `passwordHash` no registro criado.
- [`src/modules/auth/auth.service.ts`](../../../../../src/modules/auth/auth.service.ts), linhas 31-40
  - Verifica a senha informada com o hash armazenado.
- [`prisma/schema.prisma`](../../../../../prisma/schema.prisma), linhas 25-38
  - Define `passwordHash` no modelo persistido `User`.
- [`package.json`](../../../../../package.json), linhas 25-34
  - Declara a dependência `bcrypt` na aplicação.

### Evidências de código

```typescript
// src/modules/users/user.service.ts:16, 21-33
const BCRYPT_ROUNDS = 10;
const passwordHash = await bcrypt.hash(input.password, BCRYPT_ROUNDS);
const created = await this.users.create({
  email: input.email,
  passwordHash,
  name: input.name,
  role: input.role,
});
```

```typescript
// src/modules/auth/auth.service.ts:31-40
const user = await this.users.findByEmail(input.email);
if (!user) {
  throw new UnauthorizedError('Invalid credentials');
}
const ok = await bcrypt.compare(input.password, user.passwordHash);
if (!ok) {
  throw new UnauthorizedError('Invalid credentials');
}
```

```prisma
// prisma/schema.prisma:25-38
model User {
  id           String   @id @default(uuid()) @db.Char(36)
  email        String   @unique @db.VarChar(255)
  passwordHash String   @db.VarChar(255)
  name         String   @db.VarChar(150)
  role         UserRole @default(OPERATOR)
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  createdOrders        Order[]              @relation("OrderCreatedBy")
  statusHistoryChanges OrderStatusHistory[] @relation("StatusChangedBy")

  @@map("users")
}
```

### Análise de impacto

- **Introdução:** os arquivos consultados foram incluídos no histórico disponível em 2026-06-24, no commit `init repository`; não é possível determinar se esse foi o momento da decisão.
- **Modificações:** um commit aparece no histórico de cada arquivo principal consultado; não foram encontrados commits posteriores nesses arquivos.
- **Última alteração:** 2026-06-24, no commit `init repository`; o assunto não documenta motivação específica para bcrypt ou para o fator de custo.
- **Abrangência:** módulo USERS, consumo direto pelo módulo AUTH e modelo `User` persistido via Prisma.
- **Temas recentes:** não há commits posteriores nem temas recentes identificáveis nos arquivos examinados.

### Alternativas (se observáveis)

Não foram encontradas alternativas explicitadas nos arquivos consultados, configurações ou assuntos de commits. Não se infere que outras opções tenham sido avaliadas.

## Questões para abordar no ADR formal (se criado)

- Qual necessidade orientou a escolha de `bcrypt`?
- Que alternativas de algoritmo ou serviço de gestão de credenciais foram avaliadas?
- Como o fator de custo 10 foi definido e como deverá ser revisto?
- Qual estratégia deve ser usada para migrar hashes existentes se o algoritmo ou o fator de custo mudar?
- Quais requisitos de segurança e desempenho devem orientar essa decisão?

## Potenciais ADRs relacionados

Nenhum candidato relacionado foi identificado na análise restrita a USERS. A estratégia mais ampla de autenticação pertence a outro módulo e não foi analisada nesta etapa.

## Observações adicionais

- O diretório padrão `docs/adrs/generated/` não existe; não havia ADRs formais disponíveis para comparação de duplicidade, relação ou cronologia.
- A análise ficou restrita a USERS. `src/modules/auth/auth.service.ts` foi consultado somente para verificar o consumo direto do hash criado pelo módulo.
- A pontuação de custo e prioridade é uma estimativa técnica baseada no código observado; não há dados de produção, volume de contas ou requisitos de desempenho no escopo consultado.
