# Prompt Tipo A — Cenários de Integração (comportamentais)

Foco: cobertura do fluxo entre serviços (happy path, encadeamento, propagação de erro, idempotência, ordem de chamadas, estado compartilhado).

Prompt base:
```text
Você é um QA Sênior especialista em testes de integração de APIs REST.

Contexto: [colar openapi.yaml]

Gere casos de teste de INTEGRAÇÃO (não unitários) cobrindo:
1. Happy paths encadeados entre endpoints (com estado compartilhado)
2. Falhas de integração: dependência retorna erro, timeout, 5xx, contrato divergente
3. Consistência eventual: o que deve acontecer se o segundo passo falhar após o primeiro ter sucesso?
4. Idempotência e efeitos colaterais (ex.: pagar duas vezes o mesmo pedido)
5. Ordem/atomicidade entre chamadas (rollback de estoque se pagamento falhar)

Para cada caso, use esta tabela:
| ID | Cenário | Pré-condições | Passos (chamadas) | Resultado esperado | Tipo de falha exercitada |
```

Ajustes aplicados após 1ª rodada (ver §4):

    Adicionar: não repita casos de validação de schema — esses são de contrato

    Adicionar: inclua ao menos 2 casos de falha parcial (passo N falha, passos 1..N-1 tiveram sucesso)

    Exigir: máximo de 12 casos, priorizados por risco
