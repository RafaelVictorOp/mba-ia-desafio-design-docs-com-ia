# Processo de produção dos Design Docs

> **Status:** modelo para preenchimento. Substitua os campos entre `[colchetes]` pelos dados reais do projeto e remova as instruções antes da entrega.

## Sobre o desafio

O desafio consiste em [descrever, em suas palavras, o problema ou sistema que precisa ser projetado] e em produzir um conjunto de *Design Docs* que permita compreender o contexto, os requisitos, as decisões arquiteturais e os trade-offs envolvidos. A entrega deve ajudar leitores técnicos a avaliar não apenas a solução proposta, mas também os motivos por trás das escolhas.

Durante a produção, meu objetivo foi [explicar o objetivo pessoal ou técnico, por exemplo: transformar requisitos em uma proposta arquitetural verificável]. O principal cuidado foi [indicar restrições, escopo, prioridades e critérios de qualidade].

## Ferramentas de IA utilizadas

| Ferramenta | Papel no processo                                                | Como validei o resultado |
| ---------- | ---------------------------------------------------------------- | ------------------------ |
| Codex      | Explorar requisitos, estruturar documentos, revisar clareza etc. |                          |
| ChatGpt    | Elaboração de agentes, validação de requisitos               |                          |

**Limites do uso de IA:** [Descreva o que foi produzido por você, o que teve auxílio de IA e quais decisões foram tomadas manualmente.]

## Workflow adotado

Organizei o trabalho em etapas para manter a rastreabilidade entre requisitos, arquitetura e decisões. A sequência real foi:

1. **Leitura e decomposição do desafio:** Entendimento do que se trata o arquivo "TRANSCRICAO.md¨ 
2. **Levantamento de requisitos e restrições:** [Requisitos funcionais, não funcionais, premissas, riscos, critérios de aceitação.]
3. **Elaboração do HLD (High-Level Design):** [Visão geral, componentes, fluxos, integrações, principais escolhas arquiteturais.]
4. **Detalhamento técnico:** [LLD, contratos, modelos, diagramas ou outros documentos, caso existam.]
5. **Registro de decisões em ADRs:** [Quais decisões mereceram ADR e por quê; alternativas e consequências.]
6. **Revisão e consolidação:** [Checagem de consistência, lacunas, contradições, adequação ao escopo e organização final.]

**Como interagi com a IA:** [Explique se usou conversas separadas por documento, forneceu contexto progressivamente, solicitou críticas, comparou alternativas, exigiu justificativas, revisou por seção etc.]

**Critério para avançar entre etapas:** [O que precisava estar resolvido antes de iniciar o próximo documento.]

## Prompts customizados

> Os textos a seguir são **exemplos adaptáveis**, não um registro de prompts já executados. Substitua-os pelos prompts reais usados no projeto. Inclua pelo menos dois.

### Prompt 1: estruturar o HLD

```text
Atue como arquiteto de software em uma revisão técnica.
Contexto: [descreva o problema, domínio e usuários].
Requisitos: [liste os requisitos relevantes].
Restrições: [tecnologia, prazo, escala, segurança, orçamento etc.].

Proponha a estrutura de um HLD contendo contexto, escopo,
componentes, integrações, fluxos principais, requisitos não
funcionais, riscos e questões em aberto.
Não invente requisitos ou decisões: sinalize explicitamente
lacunas e premissas. Para cada escolha, explique os trade-offs.
```

**Objetivo do prompt:** [Por que ele foi criado e que resultado você esperava.]

**Resultado e intervenção humana:** [O que foi aproveitado, rejeitado ou ajustado.]

### Prompt 2: revisar uma decisão arquitetural (ADR)

```text
Revise a decisão arquitetural a seguir: [descreva a decisão].
Contexto e restrições: [informe os dados conhecidos].
Alternativas consideradas: [opção A, opção B, opção C].

Produza uma análise no formato ADR com contexto, decisão,
justificativa, alternativas rejeitadas e consequências positivas
e negativas. Questione premissas não comprovadas, identifique
riscos operacionais e indique o que ainda precisa ser validado.
Não trate recomendações como fatos sem evidência.
```

**Objetivo do prompt:** [Explique a finalidade.]

**Resultado e intervenção humana:** [Descreva sua revisão.]

### Outros prompts relevantes (opcional)

```text
[Cole aqui um terceiro prompt real: revisão cruzada dos docs,
identificação de inconsistências, análise de requisitos etc.]
```

## Iterações e ajustes

**Total de iterações principais:** [N]
**Como contei uma iteração:** [Ex.: um ciclo de geração → revisão → correção de uma seção ou documento.]

| Iteração | Documento/etapa          | Problema na saída da IA                                   | Ajuste feito por mim                                | Resultado            |
| ---------- | ------------------------ | ---------------------------------------------------------- | --------------------------------------------------- | -------------------- |
| 1          | [HLD / outro]            | [Ex.: solução genérica, sem considerar uma restrição] | [Contexto ou instrução adicional fornecida]       | [O que melhorou]     |
| 2          | [ADR / outro]            | [Ex.: trade-offs superficiais ou premissa incorreta]       | [Correção, contraponto ou pedido de comparação] | [O que mudou]        |
| 3          | [Revisão final / outro] | [Ex.: inconsistência entre documentos]                    | [Ajuste manual ou novo prompt]                      | [Como foi resolvida] |

**Principais aprendizados:** [Explique como sua revisão técnica influenciou o resultado e o que mudou na maneira de formular prompts.]

**O que não foi resolvido (se aplicável):** [Pendências, hipóteses a validar, riscos aceitos e limites da documentação.]

## Como navegar a entrega

> Altere os caminhos abaixo para refletir a estrutura **real** do repositório. Exclua itens inexistentes.

```text
.
├── README.md                      # Processo de produção (este documento)
├── docs/
│   ├── hld.md                     # Visão arquitetural de alto nível
│   ├── lld.md                     # Detalhamento técnico (se houver)
│   ├── adr/
│   │   ├── 0001-[decisao].md      # Decisão arquitetural 1
│   │   └── 0002-[decisao].md      # Decisão arquitetural 2
│   └── diagrams/                  # Diagramas e fluxos (se houver)
└── [outros-arquivos]
```

**Ordem sugerida de leitura:**

1. **`README.md`**: contexto do desafio e relato do processo de produção.
2. **`[caminho/do/hld.md]`**: visão geral da solução, componentes e fluxos.
3. **`[caminho/do/lld.md]`**: detalhes de implementação, se fizer parte da entrega.
4. **`[caminho/dos/adrs/]`**: decisões arquiteturais, alternativas e consequências.
5. **`[caminho/dos/diagramas/]`**: apoio visual aos documentos, se aplicável.

**Relação com o enunciado original (opcional):** [Link para uma cópia preservada do enunciado, por exemplo `docs/enunciado-original.md`, se você optar por mantê-la.]

---

### Checklist antes de substituir o README da raiz

- [ ] Descrevi o desafio em 1 ou 2 parágrafos, com minhas próprias palavras.
- [ ] Listei apenas as ferramentas de IA que realmente utilizei e seus papéis.
- [ ] Registrei a ordem real de criação dos documentos e a dinâmica com a IA.
- [ ] Incluí ao menos dois prompts reais, em blocos de código.
- [ ] Documentei erros, correções e o número de iterações principais.
- [ ] Conferi todos os caminhos e a ordem de leitura dos arquivos entregues.
- [ ] Removi instruções, exemplos fictícios e campos `[pendentes]`.
