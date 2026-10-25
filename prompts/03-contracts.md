# Prompt Tipo C (extra) — Contratos do Consumidor (contract testing style)

Foco: expectativas do consumidor sobre o provedor (Pact-like), detectar quebra de contrato.

```text
Você é especialista em consumer-driven contract testing (Pact).

Para o fluxo "OrderService consome ProductService (PATCH /products/{id}/stock)", gere:
- Interação do consumidor: requisição esperada + resposta mínima esperada
- Estados do provedor necessários (provider states)
- Contratos que NÃO devem ser assumidos (anti-patterns)

Formato: YAML compatível com Pact v3.
```
