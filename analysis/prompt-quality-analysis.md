# 4. Análise: influência do tipo de prompt e ajustes

## Como cada tipo influenciou qualidade e diversidade

| Tipo de prompt | O que trouxe de melhor | Limitação observada |
| :--- | :--- | :--- |
| **A — Cenários de integração** | Gerou diversidade de falhas sistêmicas (falha parcial, idempotência, concorrência, compensação). Foi o tipo que mais produziu casos de integração de verdade — os outros tendiam a virar teste de unidade disfarçado. | Na 1ª versão, a LLM devolveu muitos casos que eram apenas validação de schema (400 por campo inválido). Precisei proibir explicitamente. |
| **B — Dados de teste** | Excelente para fronteira e determinismo. Alto volume de fixtures reaproveitáveis. A exigência de status_esperado por fixture forçou a LLM a pensar no efeito e não só no payload. | Sem instrução de UUIDs fixos, ela inventou IDs aleatórios e as fixtures ficaram não executáveis em conjunto. Sem pedir "motivo/regra violada", a saída ficou sem valor de análise. |
| **C — Contratos** | Único que pegou quebras silenciosas de contrato (campos extras, mensagens de erro, ordem de campos). Reduz falsos positivos em CI. | A LLM inicialmente gerou contratos "ideais" em vez de refletir o consumidor real — precisei ancorar no fluxo concreto (OrderService→ProductService) e listar anti-patterns. |

## Ajustes de prompt que foram necessários (e por quê)

- **"Não gere casos de validação de schema; foque em interações entre endpoints."**
  - **Motivo:** o prompt A, sem isso, produzia ~60% de casos que eram testes de contrato/validação, não de integração.

- **Exigir formato tabular fixo e limite de casos (ex.: 12).**
  - **Motivo:** saídas em prosa eram difíceis de converter em testes; limite forçou priorização por risco.

- **Pedir explicitamente "falha parcial" e "compensação/rollback".**
  - **Motivo:** sem isso, a LLM só gerava happy path + erro no passo único. Nenhum cenário de estado intermediário inconsistente apareceu na 1ª rodada.

- **Exigir Idempotency-Key e efeitos colaterais duplicados.**
  - **Motivo:** idempotência não emerge espontaneamente — só aparece quando pedida nominalmente.

- **No prompt B: fixar UUIDs determinísticos e exigir status_esperado + regra_violada por fixture.**
  - **Motivo:** tornou as fixtures executáveis e auditáveis; sem isso, eram "exemplos soltos".

- **No prompt C: ancorar no fluxo concreto + pedir "anti-patterns".**
  - **Motivo:** a LLM tendia a gerar contratos aspiracionais. A lista de anti-patterns virou a parte mais útil (guia o que o consumidor não deve assumir).

- **Repetir o prompt com temperature=0.2 e rodar 3x, unindo resultados.**
  - **Motivo:** diversidade real veio da união de execuções — cada rodada trazia 1–2 casos novos (ex.: SC-08 concorrência e SC-10 compensação só apareceram em rodadas diferentes).

## Regra prática que emergiu

Para teste de integração, o prompt funciona melhor quando proíbe explicitamente o que é de unidade e de contrato, e quando nomeia os tipos de falha sistêmica desejados (falha parcial, idempotência, concorrência, compensação, consistência eventual). Sem esses "nortes", a LLM regride para o que é mais fácil gerar: validação de schema.
