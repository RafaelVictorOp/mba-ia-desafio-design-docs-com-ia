---
name: traceability-tracker
description: "Cria, audita e atualiza docs/TRACKER.md, rastreando requisitos, decisões, restrições e trade-offs de PRD, RFC, FDD e ADR até evidências verificáveis na transcrição ou no código. Use quando gerar, revisar ou modificar documentação técnica, quando solicitar rastreabilidade documental ou quando validar cobertura mínima de 80%."
---

# Skill: Tracker de rastreabilidade documental

## Objetivo

Criar e manter `docs/TRACKER.md` como referência cruzada entre afirmações identificáveis da documentação técnica e suas evidências em **transcrições** ou **código-fonte**. Minimizar alucinações, preservar integridade documental e demonstrar cobertura verificável de **no mínimo 80%**.

## Quando aplicar

- Após criar ou atualizar PRD, RFC, FDD, ADR ou outros documentos técnicos em `docs/`.
- Ao solicitar criação, atualização, auditoria ou revisão do tracker.
- Quando um documento alterar, adicionar ou excluir requisitos, decisões, restrições ou trade-offs.
- Antes da entrega final de documentação, quando a rastreabilidade for exigida.

Esta skill **não é um hook**: sua ativação automática pelo Copilot depende de ele reconhecer a relevância da tarefa. Para execução obrigatória ao fim de toda sessão, configure adicionalmente um hook ou uma verificação de CI.

## Entradas e descoberta

1. Localizar os documentos de referência existentes, prioritariamente `docs/PRD.md`, `docs/RFC.md`, `docs/FDD.md` e `docs/adrs/*.md`; incluir outros documentos de requisitos e design somente se forem parte da entrega.
2. Identificar a transcrição fornecida, anexada ou presente no repositório. Descobrir localização e formato antes de interpretar timestamps. **Não presumir** que existe uma transcrição.
3. Examinar os arquivos de código relevantes, inclusive contratos, configurações, migrations, testes e infraestrutura, quando servirem como evidência.
4. Ler `docs/TRACKER.md` existente, se houver, para preservar IDs estáveis e evitar duplicidade.
5. Não alterar arquivos de código ou documentos de origem; somente criar ou atualizar `docs/TRACKER.md`, salvo autorização expressa.

## Procedimento obrigatório

### 1. Inventariar itens documentais

- Percorrer os documentos elegíveis e extrair **afirmações identificáveis e verificáveis**, como requisitos funcionais e não funcionais, regras de negócio, decisões arquiteturais, restrições, contratos, alternativas e trade-offs.
- Tratar cada item atômico como uma unidade de cobertura. Separar afirmações distintas quando possuírem evidências independentes. Evitar criar uma linha para cada frase puramente explicativa, título, exemplo hipotético ou repetição do mesmo item.
- Registrar o documento e a seção em que cada item aparece; usar os IDs já presentes na documentação quando disponíveis.
- Para documentos com itens equivalentes repetidos, contar o item uma vez **por afirmação documental identificável** e documentar sem duplicar linhas arbitrariamente.

### 2. Localizar e verificar a origem

- **TRANSCRICAO:** localizar o trecho correspondente na transcrição e registrar `timestamp + nome do falante`, por exemplo `[09:17] Diego`. Copiar o timestamp e o nome exatamente como constam na fonte. Não inventar falantes, horários ou trechos. Se o texto não possuir timestamp ou identificação do falante, considerar a evidência insuficiente para o formato obrigatório e registrar como lacuna, a menos que esses metadados existam comprovadamente em outra parte da transcrição.
- **CODIGO:** encontrar o arquivo real no repositório e verificar o trecho que demonstra a afirmação. Registrar o caminho relativo ao repositório, por exemplo `src/modules/orders/order.service.ts`. Não citar caminhos que não existem. Não supor que um nome de arquivo, isoladamente, comprova o comportamento descrito.
- Registrar uma linha por ligação `item → fonte`. Se um item tiver fontes independentes, usar linhas adicionais com o **mesmo ID** e fontes/localizações diferentes; a unicidade do ID é do **item documental**, não da linha de evidência.
- Se a documentação não estiver comprovada pela transcrição nem pelo código, **não fabricar fonte**. Registrar o item no relatório de lacunas ao final da execução e solicitar correção/validação humana. Não inserir fontes como `HIPOTESE`, `INFERIDO`, `PENDENTE` ou valores vazios na coluna Fonte: somente `TRANSCRICAO` ou `CODIGO` são válidos.
- Se a transcrição afirmar intenção futura, não atribuir ao código uma implementação inexistente. Distinguir decisão proposta de comportamento já implementado.
- Se as fontes se contradisserem, apontar conflito, não escolher silenciosamente a evidência mais conveniente.

### 3. Definir identificadores estáveis

- Priorizar IDs explícitos dos documentos, sem renumerar IDs existentes que continuam representando o mesmo item.
- Para itens sem ID, gerar identificadores estáveis e legíveis, como `PRD-FR-01`, `PRD-NFR-02`, `RFC-ALT-02`, `FDD-CONTRATO-03`, `ADR-002`.
- Evitar que o mesmo ID seja usado para **itens diferentes**. O mesmo ID pode se repetir apenas quando representa o mesmo item sustentado por mais de uma evidência.
- Se um item documentado for removido, remover suas linhas do tracker; se for alterado, revalidar todas as fontes associadas.

### 4. Escrever ou atualizar o tracker

Gerar `docs/TRACKER.md` com **uma única tabela Markdown** e exatamente estas seis colunas, nesta ordem:

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| -- | --------- | ---- | ----------------- | ----- | ----------- |
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Permitir consultar pedidos pelo identificador | TRANSCRICAO | [09:17] Diego |
| FDD-CONTRATO-03 | docs/FDD.md | Contrato | Endpoint de consulta expõe a rota GET /orders/{id} | CODIGO | src/modules/orders/order.controller.ts |

**Atenção:** as linhas acima são apenas exemplos de formato, não evidências do projeto. Nunca copiá-las para um tracker real sem comprovação.

Regras de preenchimento:
- `ID`: identificador do item documental.
- `Documento`: caminho relativo do documento que contém o item.
- `Tipo`: classificação descritiva em português, como `Requisito Funcional`, `Requisito Não Funcional`, `Decisão`, `Restrição`, `Trade-off`, `Contrato`, `Alternativa` ou `Regra de Negócio`.
- `Conteúdo (resumo)`: uma linha clara e fiel ao documento; preservar condições, limites e números essenciais.
- `Fonte`: **exatamente** `TRANSCRICAO` ou `CODIGO`.
- `Localização`: `[mm:ss] Nome` ou outro timestamp literal presente na transcrição seguido do falante, ou caminho de arquivo existente no código.
- Escapar barras verticais (`\|`) em células Markdown e preservar uma linha física por evidência.
- Não adicionar colunas, seção de estatísticas, introdução ou comentários ao `docs/TRACKER.md`: manter o formato obrigatório da tabela. Comunicar indicadores e lacunas **na resposta ao usuário**.

### 5. Validar a cobertura mínima

Calcular:

- `T`: número total de itens identificáveis inventariados nos documentos elegíveis.
- `R`: número de itens **distintos** com pelo menos uma linha no tracker contendo fonte e localização verificadas.
- `Cobertura = (R / T) × 100` quando `T > 0`.
- **Meta:** cobertura **≥ 80%**.

Não contar linhas duplicadas, evidências não verificadas ou arquivos ausentes como cobertura adicional. Se `T = 0`, informar `cobertura não aplicável`, não declarar 100%.

Antes de concluir, verificar:

1. As seis colunas têm exatamente os nomes e a ordem especificados.
2. Todos os documentos e caminhos de código citados existem no repositório.
3. Todo apontamento de transcrição possui timestamp e falante reais.
4. Cada resumo corresponde ao item do documento de origem.
5. Nenhum ID designa dois itens diferentes.
6. Nenhum item removido permanece no tracker.
7. Os totais `T` e `R` contam itens únicos corretamente.
8. A cobertura atinge 80%, ou o resultado é reportado explicitamente como insuficiente.
9. Cada conflito entre documento e fonte é comunicado ao usuário.

### 6. Resposta obrigatória ao final

Após salvar `docs/TRACKER.md`, apresentar um resumo sucinto contendo:

- Caminho do arquivo gerado ou atualizado.
- Quantidade de itens identificáveis (`T`).
- Quantidade de itens rastreados com fonte válida (`R`).
- Cobertura percentual calculada e resultado `atingiu a meta` ou `abaixo de 80%`.
- Itens sem evidência, fontes inacessíveis e divergências relevantes, com documento e ID quando houver.
- Se for impossível acessar transcrições ou código, registrar a limitação e **não afirmar** que a rastreabilidade está validada.

Se a cobertura for inferior a 80%, entregar o tracker parcial, sem inventar evidência, e explicar o que precisa ser fornecido ou corrigido para atingir a meta.

## Limitações e comportamento seguro

- Não inventar referências, timestamps, falantes, arquivos, decisões ou trechos de código.
- Não transformar hipóteses em fatos confirmados.
- Não alterar o teor de PRD/RFC/FDD/ADR automaticamente para satisfazer a métrica.
- Não apagar registros válidos ao atualizar a tabela.
- Não atribuir a um arquivo de código a evidência de regra que não esteja demonstrada nele.
- Quando uma decisão surgir apenas em conversa sem transcrição persistida e identificável, sinalizar que falta fonte válida.
