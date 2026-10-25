# Generated Scenarios

| ID | Cenário | Pré-condições | Passos | Resultado esperado | Falha exercitada |
| :--- | :--- | :--- | :--- | :--- | :--- |
| SC-01 | Pedido + pagamento aprovado (happy path) | Cliente e produto válidos, estoque = 5 | 1. POST /orders (qty 2) 2. POST /payments | 201 + 201; order.status=PAID; estoque=3 | — |
| SC-02 | Estoque insuficiente no POST /orders | Produto com estoque 1 | 1. POST /orders qty 3 | 409, sem chamada a Payments | Propagação de erro 409 do serviço Products |
| SC-03 | Pagamento recusado após pedido criado | Pedido em CREATED | 1. POST /payments (mock 402) | Order permanece CREATED; estoque não é debitado (rollback) | Falha parcial: passo 2 falha, passo 1 teve sucesso |
| SC-04 | Pagamento duplicado (idempotência) | Pedido já PAID | 1. POST /payments 2ª vez | 409 "Order already paid"; sem cobrança dupla | Efeito colateral / idempotência |
| SC-05 | Cliente inexistente no POST /orders | customerId aleatório | 1. POST /orders | 404 | Propagação de erro do serviço Customers |
| SC-06 | Products indisponível (5xx) durante criação | Mock Products → 503 | 1. POST /orders | 503 ao cliente; nenhum pedido persistido | Timeout/indisponibilidade de dependência |
| SC-07 | Payload válido, mas contrato divergente (extra field) | Item com discount extra | 1. POST /orders | Ignora campo extra (tolerância) ou 400 se additionalProperties:false | Divergência de contrato |
| SC-08 | Estoque zerado por outra requisição concorrente | Estoque=1; duas requests paralelas | 1. POST /orders x2 simultâneos | Um 201, outro 409 (sem over-sell) | Concorrência |
| SC-09 | Payment após remoção do pedido | Pedido existe e é removido via admin | 1. POST /payments | 404 | Estado compartilhado divergente |
| SC-10 | Compensação: pagamento OK, mas atualização de order falha | Mock OrderService PATCH → 500 | 1. POST /payments | Payments deve reverter cobrança (compensating transaction) ou registrar inconsistência | Atomicidade entre serviços |
| SC-11 | Pedido com 1 item vs N itens (cálculo total) | 3 itens | 1. POST /orders | total = Σ preço×qty (2 casas decimais) | Cálculo agregado entre Products e Orders |
| SC-12 | Retry de POST /orders com mesmo Idempotency-Key | Header presente | 1. POST x2 com mesma key | Mesmo order.id, 201 na 1ª, 200 na 2ª | Idempotência de criação |
