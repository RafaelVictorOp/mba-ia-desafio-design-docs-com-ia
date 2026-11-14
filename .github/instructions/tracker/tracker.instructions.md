# Regras de rastreabilidade

Sempre que criar ou modificar documentos em:

- docs/PRD.md
- docs/RFC.md
- docs/FDD.md
- docs/adrs/*.md

Utilize a skill `traceability-tracker` para:

1. Identificar os requisitos, decisões, restrições e trade-offs alterados.
2. Verificar a origem de cada item na TRANSCRICAO.md ou no código-fonte.
3. Atualizar docs/TRACKER.md.
4. Preservar IDs e referências existentes.
5. Validar cobertura mínima de 80%.
6. Informar itens sem evidência verificável.

Nunca invente timestamps, falantes ou caminhos de código.

Não considere a documentação concluída sem verificar
a rastreabilidade.
