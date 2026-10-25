# Integration Tests com LLM — Estudo

API fictícia "LojaX Order API" usada como base para avaliar 3 tipos de prompt
(A: cenários de integração, B: dados de teste, C: contratos do consumidor).

- `api/openapi.yaml` — contrato da API fictícia
- `prompts/` — prompts utilizados (versão final após ajustes)
- `tests/` — saídas geradas pela LLM (revisadas manualmente)
- `analysis/prompt-quality-analysis.md` — análise de qualidade/diversidade e ajustes de prompt

## Como cada tipo de prompt contribuiu
- **A** → cenários de falha sistêmica (falha parcial, idempotência, concorrência, compensação)
- **B** → fixtures determinísticas + fronteira + inválidos auditáveis
- **C** → detecção de quebra de contrato entre consumidor e provedor

## Principais aprendizados
- Proibir explicitamente casos de unidade/validação de schema
- Nomear os tipos de falha sistêmica desejados
- Rodar múltiplas execuções com temperatura baixa e unir resultados
- Fixar IDs determinísticos nas fixtures
