# Prompt Tipo B — Geração de Dados de Teste (fixtures)

Foco: payloads válidos e inválidos, valores de fronteira, dados que atravessam serviços.

Prompt base:
```text
Você é um engenheiro de dados de teste.

Dado o schema abaixo, gere um arquivo JSON de FIXTURES com:
- 3 clientes válidos (um deles com nome com acentos e caracteres especiais)
- 5 produtos válidos (preços variados: barato, alto, com centavos)
- 10 pedidos válidos (1 item, N itens, quantidade alta)
- 8 payloads INVÁLIDOS por endpoint (cada um quebrando uma regra diferente)
- 5 pares "boundary" (exatamente no limite do schema)

Formato:
{
  "valid": { "customers": [...], "products": [...], "orders": [...] },
  "invalid": [ { "endpoint": "...", "payload": {...}, "regra_violada": "...", "status_esperado": 400 } ],
  "boundary": [...]
}

Regras: UUIDs fixos e reutilizáveis, sem dependência entre objetos além das FKs declaradas.
```

Ajuste:

    Exigir: determinismo (mesmos IDs entre execuções)

    Pedir: motivo para cada inválido e status_esperado explícito
