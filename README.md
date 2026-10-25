# Testes de Integração com GenAI — LojaX Order API

> **Atividade da Unidade III** do curso de *Testes de Software Automatizados com Inteligência Artificial Generativa* (AKCIT / CEIA / UFG).  
> Estudo comparativo de engenharia de prompts aplicada à geração de testes de integração, síntese de fixtures de dados e contratos orientados pelo consumidor em um ecossistema de microsserviços.

---

## 📌 Resultado em Destaque

Três estratégias complementares de prompt foram avaliadas contra o contrato OpenAPI de um sistema distribuído de e-commerce, gerando **12 cenários de integração de alto risco sistêmico**, um **catálogo determinístico de fixtures com 18 variações** e uma **especificação formal de contrato Pact v3**:

| Tipo de Prompt | Abordagem / Técnica | Foco Principal | Volume Gerado | Ponto Forte Revelado | Ponto Cego Corrigido |
| :--- | :--- | :--- | :---: | :--- | :--- |
| **Prompt A** | Zero-Shot com restrições comportamentais | Falhas sistêmicas, atomicidade e estado compartilhado | 12 cenários BDD | Identificou concorrência, falha parcial e sagas compensatórias | Sem proibições explícitas, regredia para validação de schema |
| **Prompt B** | Engenharia de Dados de Teste | Payloads válidos, limites (boundary) e negativos | 18 fixtures estruturadas | Alto determinismo e rastreabilidade com status esperado e regra violada | Sem exigência de determinismo, gerava IDs randômicos não correlacionáveis |
| **Prompt C** | Consumer-Driven Contract Testing (Pact v3) | Quebra de contrato entre microsserviços | Interações Pact + 3 anti-patterns | Isola quebras silenciosas (campos extras, ordenação de JSON) | Tendência a gerar contratos "ideais"; exigiu ancoragem em anti-patterns |

---

## 🏗️ O Cenário de Integração (Arquitetura Sob Teste)

A API fictícia **"LojaX Order API"** ([api/openapi.yaml](api/openapi.yaml)) modela a interação síncrona entre quatro serviços interdependentes:

```
[ Cliente ]
    │
    ▼
[ POST /orders ] ──────────► [ ProductService ]
(Cria pedido & orquestra)     (Valida preço & debita estoque: PATCH /products/{id}/stock)
    │
    ▼ (se reserva OK)
[ POST /payments ] ────────► [ Gateway de Pagamento ]
(Efetiva cobrança)            (Sucesso -> status PAID
                               Recusa 402 -> Rollback de estoque no ProductService)
```

### O que qualifica esses testes como Integração (e não Unidade com HTTP):
1. **Transações Compensatórias (Saga):** A recusa do pagamento (passo 2) exige reversão obrigatória do estoque debitado na criação do pedido (passo 1).
2. **Concorrência e Race Condition:** Duas requisições simultâneas para o último item em estoque não podem gerar sobre-venda (*over-sell*).
3. **Idempotência de Criação e Cobrança:** Reenvio de chamadas com o mesmo `Idempotency-Key` ou cobrança de pedido já pago (`PAID`) não podem duplicar recursos nem gerar cobrança dupla.

---

## 🎯 Os Três Tipos de Prompt Avaliados

### 1. [Prompt Tipo A — Cenários de Integração](prompts/01-scenarios.md)
* **Objetivo:** Cobertura do fluxo distribuído ponta a ponta (happy path encadeado, timeouts, 5xx downstream, consistência eventual e idempotência).
* **Artefato Resultante:** [tests/scenarios/generated-scenarios.md](tests/scenarios/generated-scenarios.md).

### 2. [Prompt Tipo B — Síntese de Massa de Dados (Fixtures)](prompts/02-test-data.md)
* **Objetivo:** Geração de massas consistentes e reutilizáveis, contemplando clientes com caracteres especiais, produtos em diferentes faixas de preço, pedidos multi-itens, valores de fronteira exatos e 8 casos de erro 400/409 documentados.
* **Artefato Resultante:** [tests/data/fixtures.json](tests/data/fixtures.json).

### 3. [Prompt Tipo C — Contratos do Consumidor (Pact v3)](prompts/03-contracts.md)
* **Objetivo:** Modelar a expectativa mínima do consumidor (`OrderService`) sobre o provedor (`ProductService`) ao debitar estoque via `PATCH /v1/products/{id}/stock`, além de explicitar o que **não** deve ser assumido (*anti-patterns*).
* **Artefato Resultante:** [tests/contracts/order-to-products.pact.yaml](tests/contracts/order-to-products.pact.yaml).

---

## 📊 Matriz dos Cenários de Integração Gerados

Os 12 cenários priorizados por risco sistêmico documentados em [generated-scenarios.md](tests/scenarios/generated-scenarios.md):

| ID | Cenário | Passos Chave | Resultado Esperado | Categoria de Falha Sistêmica |
| :--- | :--- | :--- | :--- | :--- |
| **SC-01** | Pedido + pagamento aprovado | POST /orders ➔ POST /payments | 201 + 201; status PAID; estoque debitado | Happy Path encadeado |
| **SC-02** | Estoque insuficiente na criação | POST /orders (qty > stock) | 409 Conflict sem chamada a Payments | Propagação de erro de dependência |
| **SC-03** | Pagamento recusado após pedido criado | POST /payments (mock 402) | Pedido em CREATED; rollback de estoque | Falha parcial (passo 2 falha, passo 1 OK) |
| **SC-04** | Pagamento duplicado | POST /payments 2ª vez | 409 "Order already paid"; sem bi-tributação | Efeito colateral / Idempotência |
| **SC-05** | Cliente inexistente | POST /orders (invalid customerId) | 404 Not Found | Propagação de erro do Customers |
| **SC-06** | ProductService indisponível | Mock Products ➔ 503 | 503 ao cliente; sem pedido órfão persistido | Timeout / Degradação downstream |
| **SC-07** | Contrato divergente (campo extra) | POST /orders com `discount` | Tolerância a campos adicionais ou 400 | Divergência de contrato |
| **SC-08** | Concorrência na última unidade | POST /orders x2 simultâneos | Um 201 e um 409 (sem over-sell) | Race Condition / Concorrência |
| **SC-09** | Pagamento após remoção de pedido | POST /payments (pedido deletado) | 404 Not Found | Estado compartilhado divergente |
| **SC-10** | Compensação atômica | PATCH Order falha após pagamento | Reversão de cobrança ou flag de inconsistência | Atomicidade entre serviços |
| **SC-11** | Cálculo agregado de total | POST /orders (3 itens distintos) | `total = Σ(preço × qty)` com precisão decimal | Integridade de cálculo compartilhado |
| **SC-12** | Retry com Idempotency-Key | POST /orders x2 com mesma key | Mesmo order.id, 201 na 1ª e 200 na 2ª | Idempotência de criação |

---

## ⚙️ Ajustes de Prompt que Mudaram o Resultado

A engenharia de prompts refinou as respostas do modelo através de restrições negativas e imposição de formatos determinísticos (análise completa em [analysis/prompt-quality-analysis.md](analysis/prompt-quality-analysis.md)):

| Ajuste Aplicado | Motivo do Ajuste | Efeito Prático Medido |
| :--- | :--- | :--- |
| **"Não repita validação de schema"** | O Prompt A puro devolveu ~60% de casos unitários (400 por string curta, email inválido). | Eliminou ruído unitário e forçou 100% de cenários entre múltiplos endpoints. |
| **Exigir ao menos 2 falhas parciais** | O LLM gerava apenas happy path ou falha no passo 1. | Emergiu o cenário de rollback compensatório de estoque (SC-03 e SC-10). |
| **Nomear explicitamente concorrência e idempotência** | Fenômenos temporais e concorrência não emergem espontaneamente. | Produziu os cenários SC-08 (race condition no estoque) e SC-12 (`Idempotency-Key`). |
| **Fixar IDs determinísticos no Prompt B** | Modelos tendem a inventar UUIDs randômicos em cada bloco. | Tornou as fixtures correlacionáveis e executáveis em conjunto (`c1` consome `p1`). |
| **Pedir Anti-Patterns no Prompt C** | LLMs costumam criar contratos aspiracionais que amarram campos desnecessários. | Gerou cláusulas explícitas sobre não depender de ordem de JSON, nomes ou mensagens livres. |
| **Temperatura baixa (0.2) em 3 rodadas** | Garantir cobertura e diversidade com baixa taxa de alucinação. | Concorrência (SC-08) e Compensação (SC-10) emergiram da união das rodadas. |

---

## 📂 Estrutura do Repositório

```text
integration-tests-llm/
├── README.md                                 # Visão geral executiva do estudo
├── api/
│   └── openapi.yaml                          # Contrato OpenAPI 3.0.3 da API fictícia
├── prompts/
│   ├── 01-scenarios.md                       # Prompt Tipo A (cenários comportamentais de integração)
│   ├── 02-test-data.md                       # Prompt Tipo B (síntese de dados determinísticos)
│   └── 03-contracts.md                       # Prompt Tipo C (consumer-driven contracts Pact v3)
├── tests/
│   ├── scenarios/
│   │   └── generated-scenarios.md            # Matriz BDD com os 12 cenários gerados
│   ├── data/
│   │   └── fixtures.json                     # Massas de dados: válidas, inválidas e boundary
│   └── contracts/
│       └── order-to-products.pact.yaml       # Especificação formal do contrato Pact v3
└── analysis/
    └── prompt-quality-analysis.md            # Análise detalhada de LLMOps e qualidade (§4)
```

---

## 💡 Principal Aprendizado

> *"Para testes de integração, o prompt funciona melhor quando **proíbe explicitamente o que é de unidade e validação de schema**, e quando **nomeia nominalmente os tipos de falha sistêmica desejados** (falha parcial, compensação, concorrência, consistência eventual e idempotência). Sem essas diretrizes de contorno, modelos de linguagem regridem naturalmente para o caminho de menor esforço: validação sintática de campos."*
