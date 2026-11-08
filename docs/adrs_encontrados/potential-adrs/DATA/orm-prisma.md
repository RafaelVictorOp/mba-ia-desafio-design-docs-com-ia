# Potencial ADR: Uso do Prisma como ORM e camada de acesso a dados

**Módulo:** DATA  
**Categoria:** Tecnologia (ORM/camada de acesso a dados)  
**Prioridade:** Documentação obrigatória (pontuação: 130)  
**Data da identificação:** 2026-11-08

---

## Contexto de ADRs existentes

Não há ADRs formais na pasta `docs/adrs/generated/`. O `PRD` e o `RFC` mencionam Prisma como tecnologia existente e em propostas futuras, mas não são ADRs formais sobre a escolha do ORM.

O índice de análises também contém candidatos relacionados ao uso do Prisma em módulos de domínio. Esta análise não reavaliou esses módulos; ao formalizar decisões, convém verificar se um ADR compartilhado em DATA deve ser referenciado pelos candidatos específicos ou consolidá-los.

## O que foi identificado

O schema de dados é declarado em `prisma/schema.prisma`, o pacote fixa `prisma` e `@prisma/client` na versão 5.22.0, e a configuração central cria e exporta uma instância compartilhada de `PrismaClient`. O mapeamento identifica DATA como dependência dos módulos de domínio. O módulo DATA também contém uma migration versionada e seed.

O histórico Git disponível mostra os arquivos relevantes no commit inicial `init repository`, datado de 2026-06-24. Não há commits posteriores nos arquivos consultados nem mensagem que explique a escolha do Prisma ou alternativas consideradas. A data do commit inicial não comprova quando ou por que a decisão foi tomada.

## Por que essa decisão pode merecer um ADR

- **Impacto:** Prisma estrutura o schema, o cliente gerado, as migrations e a forma como os repositórios dos módulos acessam o banco.
- **Trade-offs:** o código utiliza tipos, modelo e operações do cliente Prisma; essa integração facilita o acesso tipado observado, mas atrela a camada de persistência à ferramenta e ao seu modelo. As alternativas não estão registradas.
- **Complexidade:** substituir o ORM exige rever schema, migrations, client/configuração e consultas existentes. O esforço é uma estimativa, não uma medição documentada.
- **Conhecimento da equipe:** a maior parte do acesso aos dados demonstrado depende de conceitos e APIs do Prisma.
- **Implicações futuras:** alterações no ORM ou em sua versão podem afetar geração de client, migrations e repositórios; não há evidência no histórico que detalhe estratégias de upgrade.

### Pontuação

- Base da Etapa 0 — ORM/camada de acesso a dados: **75**
- Abrangência e impacto — ferramenta central para o acesso à persistência nos módulos: **20**
- Custo de mudança — estimativa de duas a oito semanas para substituir a camada e adaptar as consultas: **15**
- Conhecimento da equipe — importante para a maioria das funcionalidades que acessam dados: **20**
- **Total: 130/150 — must-document**

Os pontos de custo e conhecimento são estimativas de análise, não fatos medidos no repositório. A categoria estrutural da Etapa 0 não passa pelos sinais de exclusão. A escolha de migrations e detalhes de configuração são considerados parte desta decisão, não candidatos separados.

## Evidências encontradas no código

### Arquivos principais

- [`prisma/schema.prisma`](../../../../../prisma/schema.prisma) — linhas 1–9
  - Declara o gerador `prisma-client-js`, o datasource e o provider MySQL.
- [`src/config/database.ts`](../../../../../src/config/database.ts) — linhas 1–10
  - Cria uma instância de `PrismaClient` e exporta uma instância compartilhada no processo.
- [`package.json`](../../../../../package.json) — linhas 25–26 e 48
  - Fixa `@prisma/client` e `prisma` na versão 5.22.0.

### Evidências de código

```typescript
// src/config/database.ts:1-10
import { PrismaClient } from '@prisma/client';
import { env } from './env.js';

export function createPrismaClient(): PrismaClient {
  return new PrismaClient({
    log: env.NODE_ENV === 'development' ? ['warn', 'error'] : ['error'],
  });
}

export const prisma: PrismaClient = createPrismaClient();
```

```prisma
// prisma/schema.prisma:1-9
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider          = "mysql"
  url               = env("DATABASE_URL")
  shadowDatabaseUrl = env("SHADOW_DATABASE_URL")
}
```

### Análise de impacto

- **Introdução:** os arquivos relevantes estão presentes no commit inicial de 2026-06-24; o histórico não estabelece quando a decisão foi efetivamente tomada.
- **Modificações:** 1 commit observado nos arquivos de pacote, schema e configuração consultados.
- **Última alteração:** 2026-06-24, commit `init repository`.
- **Abrangência:** 1 módulo analisado (DATA), cujo uso pelos módulos de domínio está registrado no mapeamento; 3 arquivos principais citados.
- **Temas recentes:** não há commits posteriores nem temas de alteração documentados. A mensagem disponível é apenas `init repository`.

### Alternativas (se observáveis)

Não foram encontradas alternativas de ORM explicitadas no código ou nas mensagens de commit consultadas.

## Questões para abordar no ADR formal (se criado)

- Quais critérios levaram à escolha do Prisma em vez de acesso SQL direto ou outro ORM?
- Que benefícios e limitações do client tipado são relevantes para o projeto?
- Como serão avaliadas atualizações de versão, migrations e eventual substituição?

## Potenciais ADRs relacionados

- [Uso do MySQL como banco relacional da aplicação](./banco-relacional-mysql.md) — escolha relacionada e complementar; este candidato trata do ORM, enquanto o outro trata do banco.

## Observações adicionais

O código comprova o uso atual do Prisma, não a intenção original nem uma comparação entre alternativas. A seleção do ORM é tratada como uma decisão estrutural separada da escolha do banco, embora as duas estejam integradas no datasource.
