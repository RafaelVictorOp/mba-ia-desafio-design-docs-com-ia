# Potencial ADR: Prisma como camada de acesso a dados de produtos

**Módulo:** PRODUCTS  
**Categoria:** Tecnologia  
**Prioridade:** Documentação obrigatória (pontuação: 100)  
**Data da identificação:** 2026-11-08

---

## Contexto de ADRs existentes

Não foram encontrados ADRs formais no diretório padrão `docs/adrs/generated/`; não há decisão existente para comparar.

## O que foi identificado

O repositório de produtos recebe um `PrismaClient` e realiza operações de consulta, contagem, criação, atualização e exclusão por meio de `this.prisma.product`. Também usa tipos `Prisma` e `Product` do Prisma Client. Isso caracteriza Prisma como a camada ORM/de acesso a dados usada pelo módulo.

O mapeamento existente identifica Prisma sobre MySQL para a persistência do sistema. Esta análise, porém, limita a evidência direta ao código de PRODUCTS e não investiga configuração, migrations ou uso em outros módulos. O código demonstra a tecnologia usada, mas não explica por que foi escolhida ou quais alternativas foram avaliadas.

## Por que essa decisão pode merecer um ADR

- **Impacto:** vincula as operações de persistência do catálogo às APIs e aos tipos do Prisma Client.
- **Trade-offs:** o repositório depende de tipos e operações Prisma; vantagens ou limitações comparativas não estão documentadas no módulo.
- **Complexidade:** substituir a camada de dados exigiria adaptar operações e tipos do repositório, mas o esforço não pode ser quantificado pela evidência disponível.
- **Conhecimento da equipe:** manutenção das consultas e alterações do catálogo requer familiaridade com Prisma.
- **Implicações futuras:** mudanças de ORM ou de estratégia de persistência podem afetar o contrato interno entre serviço e repositório e a integração com o banco.

### Pontuação

Categoria estrutural universal 3 (ORM ou camada de acesso a dados): base 75. Aplicados os mínimos especiais de abrangência (10), custo de mudança (10) e conhecimento da equipe (5), totalizando **100/150** e prioridade `must-document/`. A pontuação é o mínimo garantido pela regra; não representa uma estimativa independente do esforço.

## Evidências encontradas no código

### Arquivos principais

- [`src/modules/products/product.repository.ts`](../../../../../src/modules/products/product.repository.ts), linhas 1, 10 e 25–60
  - Importa tipos do Prisma Client e executa operações de persistência para produtos.

### Evidências de código

```typescript
import type { Prisma, PrismaClient, Product } from '@prisma/client';

export class ProductRepository {
  constructor(private readonly prisma: PrismaClient) {}

  findById(id: string): Promise<Product | null> {
    return this.prisma.product.findUnique({ where: { id } });
  }
```

### Análise de impacto

- Introdução: `product.repository.ts` aparece no histórico em 2026-06-24, no commit `init repository`; isso não comprova a data definitiva nem a motivação da decisão.
- Modificações: 1 commit registrado para o arquivo; não há histórico de evolução além da inclusão inicial.
- Última alteração: 2026-06-24, commit `init repository`; sem tema adicional documentado.
- Abrangência: evidência direta em 1 arquivo de PRODUCTS; o mapeamento identifica DATA como dependência comum, mas seus arquivos não foram analisados nesta fase.
- Temas recentes: nenhum além da inclusão inicial do arquivo.

### Alternativas

Não foram observadas alternativas de ORM ou camada de dados nos arquivos ou commits consultados.

## Questões para abordar no ADR formal (se criado)

- Quais requisitos orientaram a escolha do Prisma?
- Quais alternativas foram consideradas para acesso a dados?
- Como a decisão se relaciona com o banco MySQL e com os demais módulos consumidores?

## Potenciais ADRs relacionados

- [Uso do Express como framework HTTP no módulo PRODUCTS](./express-como-framework-http-do-modulo.md)
- [API REST para o catálogo de produtos](./api-rest-para-catalogo-produtos.md)

## Observações adicionais

A existência do Prisma Client no módulo comprova seu uso, não a motivação histórica da escolha. Verificar a decisão compartilhada de persistência do sistema antes de formalizar um ADR específico de PRODUCTS.
