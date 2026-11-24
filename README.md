# Processo de produção dos Design Docs

## Sobre o desafio

O desafio consistiu em transformar uma transcrição de reunião técnica sobre notificações outbound de pedidos em uma documentação arquitetural verificável. A entrega parte do sistema `order-management-api`, existente no workspace, e cobre o levantamento de requisitos, o desenho proposto de webhooks, os impactos na arquitetura atual e as decisões que ainda precisam de aprovação formal.

Durante a produção, o objetivo pessoal foi construir um conjunto coerente de PRD, RFC, FDD, HLD e ADRs que permitisse separar fatos da implementação atual, decisões da reunião, propostas técnicas e itens pendentes de validação. O principal cuidado foi preservar a rastreabilidade até `TRANSCRICAO.md`, até o código observado e até o histórico de commits, sem apresentar uma proposta futura como capacidade já implementada.

## Ferramentas de IA utilizadas

| Ferramenta     | Papel declarado no processo                                                                                  | Evidência e validação                                                                                                     |
| -------------- | ------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| Codex          | Explorar requisitos, navegar no repositório, estruturar documentos e revisar a consistência dos artefatos. | Histórico Git que registra criação e ajustes dos documentos; revisão posterior do conteúdo e das referências cruzadas. |
| ChatGPT        | Elaboração dos agentes de entrevista e validação de requisitos, arquitetura e decisões.                 | Arquivos em `.github/agents/` com os procedimentos utilizados como base dos agentes especializados.                        |
| GitHub Copilot | Auxílio de programação e edição no workspace, incluindo consolidação e rastreabilidade desta entrega. | Alterações visíveis no workspace e no histórico Git.                                                                     |

## Workflow adotado

A sequência real, confirmada pelo histórico Git e pelos arquivos produzidos, foi:

1. **Leitura da transcrição:** decomposição da reunião em contexto, requisitos, restrições, decisões técnicas e itens não confirmados.
2. **PRD:** criação do documento de produto a partir da transcrição e da análise do sistema existente.
3. **Análise da arquitetura atual:** identificação dos ADRs já representados pelo código de usuários, autenticação, produtos, clientes, pedidos, API REST e dados.
4. **RFC:** consolidação da proposta técnica, das alternativas realmente consideradas na transcrição e das lacunas arquiteturais.
5. **ADRs da feature:** geração dos registros para outbox, HMAC, entrega at-least-once, autorização por vínculo, soft delete, classificação de payload, retries e identidade no replay.
6. **HLD:** análise conjunta do estado atual do OMS e da arquitetura proposta para webhooks.
7. **FDD:** detalhamento técnico dos fluxos, contratos, erros, persistência, observabilidade e critérios de implementação.
8. **Rastreabilidade e revisão:** criação do tracker e da cobertura, ajuste de links e checagem das divergências entre transcrição, código e documentos.

**Como interagi com a IA:** primeiro foi utilizado o agente `prd-interviewer` para estabelecer o problema e os requisitos a partir da transcrição. Em seguida, a análise do workspace identificou os padrões existentes e os ADRs já expressos pela implementação. O RFC foi elaborado com base no PRD e no código, diferenciado o que estava confirmado, informado ou apenas proposto. Os ADRs foram revisados para não transformar uma proposta em decisão aceita sem evidência formal. Por fim, o FDD detalhou os itens necessários à implementação, com marcadores explícitos para pendências posteriores à transcrição.

**Critério para avançar entre etapas:** o PRD precisava estar estruturado antes do RFC; o RFC precisava distinguir estado atual, proposta e lacunas antes da criação dos ADRs; o HLD dependia da arquitetura confirmada no código e da proposta definida nos documentos; o FDD só avançou depois de consolidar os fluxos e contratos. As revisões finais exigiram consistência entre documentos, referências reais e ausência de afirmações sobre implementação que não existia no workspace.

## Prompts customizados

Não foi preservado um export do chat nem os textos literais enviados em cada sessão. Por isso, os blocos abaixo são instruções de trabalho reconstruídas a partir dos agentes e das etapas verificáveis no repositório. Eles não devem ser apresentados como transcrição ou como conteúdo que o modelo recebeu integralmente.

### Prompt 1: gerar o PRD a partir da transcrição

```text
Considere o arquivo .github/agents/prd-interviewer.agent.md como especificação do agente.

Leia TRANSCRICAO.md e as estruturas técnicas relevantes do workspace.
Conduza a coleta em etapas, diferenciando confirmado, hipótese e pendente.
Não invente dados, prazos, falantes, métricas ou decisões.
Preserve os timestamps literais da transcrição e referencie o código apenas
quando ele comprovar o comportamento descrito.

Gere um PRD rastreável para o sistema existente, com problema, público,
objetivos, escopo, requisitos, arquitetura proposta, riscos, critérios de
aceitação e testes.
```

**Objetivo:** estruturar os requisitos da feature de webhooks sem confundir a solicitação dos clientes com implementação já disponível.

**Resultado e intervenção humana:** foi produzido `docs/PRD.md`, que registrou a meta de entrega em menos de 10 segundos, o worker separado, o outbox, o HMAC, as retentativas e as exclusões. O conteúdo recebeu ajustes de metadados e consolidou a rastreabilidade temporal. Não existe evidência verificável do texto literal enviado para o agente.

### Prompt 2: produzir o RFC com o agente rfc-interviewer

```text
Considere o arquivo .github/agents/rfc-interviewer.agent.md como especificação do agente.

Atue como facilitador técnico de RFC em português brasileiro.
Investigue docs/PRD.md, TRANSCRICAO.md, docs/adrs/ e o código impactado.

Faça uma pergunta por vez, diferencie Confirmado, Informado pelo usuário,
Hipótese, Proposta e Não confirmado e não apresente hipóteses como decisões.
Compare pelo menos duas alternativas realmente plausíveis, incluindo manter o
estado atual quando aplicável, descreva trade-offs e registre pendências.

Não implemente código nem crie ADR como decisão aprovada. Use as seções
obrigatórias do modelo de RFC definido no agente e peça aprovação antes de
consolidar o documento.
```

**Objetivo:** usar a entrevista estruturada do agente para transformar a proposta de webhooks em um RFC rastreável, com alternativas e estado Draft explícito.

**Resultado e intervenção humana:** foi criado `docs/RFC.md`, separando o estado atual confirmado no código das decisões informadas pela transcrição e das propostas pendentes. Ajustes posteriores adicionaram links aos ADRs relacionados. O texto acima é uma instrução reconstruída a partir do arquivo do agente, não o prompt literal preservado da sessão.

### Prompt 3: estruturar o HLD com o agente hld-interviewer

```text
Considere o arquivo .github/agents/hdl-interviewer.agent.md como especificação do agente.

Atue como especialista em HLD e conduza uma entrevista técnica em português
brasileiro. Investigue primeiro o estado atual em WORKSPACE.md, src/, prisma/,
docker-compose.yml e a documentação já existente em docs/.

Reúna objetivo técnico, topologia, componentes, fluxos, modelo de dados,
interfaces, escalabilidade, segurança, observabilidade, riscos, ADRs associados
e próximos passos. Diferencie arquitetura implementada de proposta futura.

Não repita o PRD nem invente métricas, ambiente de implantação ou dependências.
Siga exatamente o esqueleto Markdown do agente e marque lacunas como Não confirmado
ou A definir.
```

**Objetivo:** documentar a arquitetura sistêmica atual e, separadamente, a extensão proposta de webhooks.

**Resultado e intervenção humana:** foi criado `docs_oms/HLD_SISTEMA.md`, mostrando o monólito modular, o MySQL via Prisma e a ausência comprovada de worker, outbox e integração outbound no código atual. O agente orientou a estrutura, mas o texto foi reconstruído a partir do arquivo de configuração e não representa uma transcrição de chat.

### Prompt 4: identificar e gerar ADRs com o agente adr-generator-from-rfc

```text
Considere o arquivo .github/agents/adr/adr-generator-from-rfc.agent.md como
especificação do agente.

Analise docs/RFC.md, os ADRs existentes em docs/adrs/ e os módulos diretamente
impactados da implementação. Identifique decisões arquiteturais duradouras,
dependências, trade-offs e relações com decisões anteriores.

Não implemente código, não altere ADRs existentes e não converta propostas em
decisões accepted sem aprovação explícita. Use proposed para decisões ainda não
aprovadas. Não crie alternativas históricas inexistentes no RFC.

Gere ADRs em pt-BR conforme MADR 4.0, com uma única decisão por arquivo,
numeração única no repositório, seções obrigatórias e referências verificáveis.
Quando houver lacuna material, informe a pendência e prossiga com os candidatos aptos.
```

**Objetivo:** registrar separadamente decisões arquiteturais relevantes sem ampliar o FDD ou transformar uma recomendação provisória em aprovação formal.

**Resultado e intervenção humana:** foram criados ADR-018 a ADR-025 para outbox, HMAC, entrega at-least-once, vínculo usuário-cliente, soft delete, payload excedente, retry/DLQ e identidade no replay. Parte dos registros permanece como proposta ou depende de validação. O bloco é uma instrução reconstruída a partir do agente configurado, não o prompt literal utilizado.

## Iterações e ajustes

**Total de iterações principais verificáveis:** 6

Uma iteração foi contada como um ciclo completo de criação ou ajuste documentado por commit e associado a uma mudança material no conjunto de artefatos.

| Iteração | Documento ou etapa     | Problema ou lacuna                                                                        | Ajuste realizado                                                                                           | Resultado                                                                       |
| ---------- | ---------------------- | ----------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| 1          | PRD                    | Era necessário transformar a reunião em requisitos estruturados e rastreáveis.         | Criação do PRD e posterior inclusão dos metadados e da rastreabilidade temporal.                        | Estrutura inicial de contexto, escopo, objetivos e requisitos.                  |
| 2          | Arquitetura atual      | A proposta poderia ser confundida com componentes já existentes.                         | Análise dos módulos, rotas, Prisma, MySQL e ausência de worker ou outbox no código atual.              | Separação verificável entre estado atual e feature futura.                   |
| 3          | RFC                    | Algumas propostas técnicas ainda não tinham aprovação formal.                         | Classificação de requisitos, alternativas e propostas como confirmado, informado, hipótese ou pendente. | RFC mantido como Draft e sem aprovação arquitetural implícita.               |
| 4          | ADRs da feature        | Decisões importantes precisavam ser registradas sem duplicar o RFC.                      | Criação dos ADR-018 a ADR-025 e ajustes posteriores de vínculos, soft delete, retry e identidade.       | Decisões propostas e trade-offs registrados separadamente.                     |
| 5          | HLD e rastreabilidade  | Era necessário consolidar a visão sistêmica e explicar o que havia sido implementado.  | Criação do HLD, dos artefatos de arquitetura encontrados e do índice documental.                        | Visão atual do OMS ligada às extensões propostas.                            |
| 6          | FDD e revisão cruzada | Contratos e confirmações posteriores precisavam de detalhamento sem perder pendências. | Criação do FDD, inclusão de links no RFC e ajuste dos documentos relacionados.                          | Fluxos técnicos, contratos e lacunas consolidados para implementação futura. |

**Principais aprendizados:** a revisão técnica mostrou a importância de separar três estados em toda afirmação: confirmado no código, informado na reunião e proposto no RFC. Também ficou evidente que o agente ideal não deve converter uma recomendação em decisão aceita e que timestamps, falantes e caminhos só devem ser usados quando existem na fonte. Por fim, o histórico Git ajuda a provar autoria e sequência, mas não substitui a exportação das sessões e a confirmação formal das decisões.

**O que não foi resolvido:**

- O FDD registra uma confirmação posterior sobre vínculos usuário-cliente, mas essa confirmação não consta em `TRANSCRICAO.md`.
- Os ADRs marcados como `proposed` não representam aprovação formal da equipe.
- A arquitetura de implantação em produção, disponibilidade, volume sustentado e metas percentis não foram confirmados.
- A implementação de webhook, outbox, worker, DLQ e vínculos persistidos não foi localizada no código atual.

## Como navegar a entrega

```text
.
├── README.md
├── TRANSCRICAO.md
├── WORKSPACE.md
├── docs/
│   ├── PRD.md
│   ├── RFC.md
│   ├── FDD.md
│   ├── TRACKER.md
│   ├── TRACKABILITY_COVERAGE.md
│   ├── index.json
│   └── adrs/
│       ├── ADR-018-outbox-mysql-worker-polling.md
│       ├── ADR-019-assinatura-hmac-por-endpoint.md
│       ├── ADR-020-entrega-at-least-once-com-event-id.md
│       ├── ADR-021-autorizacao-por-vinculo-usuario-cliente.md
│       ├── ADR-022-soft-delete-e-cancelamento-de-entregas.md
│       ├── ADR-023-classificar-payload-excedente-na-criacao.md
│       ├── ADR-024-retry-com-backoff-e-dlq.md
│       └── ADR-025-identidade-de-evento-e-tentativa-no-replay.md
├── docs_oms/
│   ├── HLD_SISTEMA.md
│   └── adrs_encontrados/
│       ├── mapping.md
│       ├── potential-adrs-index.md
│       ├── generated/
│       └── potential-adrs/
└── src/
    ├── app.ts
    ├── modules/
    └── middlewares/
```

**Ordem sugerida de leitura:**

1. `README.md`: método, sequência e limitações da documentação.
2. `TRANSCRICAO.md`: fonte original da reunião técnica.
3. `docs/PRD.md`: problema, objetivos, escopo e requisitos.
4. `docs/RFC.md`: proposta técnica, alternativas e pendências.
5. `docs_oms/HLD_SISTEMA.md`: arquitetura atual e visão sistêmica.
6. `docs/adrs/`: decisões relevantes da feature proposta.
7. `docs/FDD.md`: detalhamento técnico dos fluxos e contratos.
8. `docs/TRACKER.md` e `docs/TRACKABILITY_COVERAGE.md`: rastreabilidade e cobertura.

**Relação com o estado atual:** os módulos existentes comprovam Express, autenticação JWT, padrões por módulo, Prisma, MySQL e alteração transacional de pedidos. A implementação de webhook está descrita como proposta futura e não foi localizada em `src/`, no `package.json` ou no schema Prisma.
