# An-lise-Contas-Laranja
# 🛡️ Análise de Dados para Prevenção à Fraude (SQL)

## 📌 Sobre o Projeto
Este projeto simula o dia a dia de uma equipe de investigação de fraudes financeiras (Threat Intelligence / Anti-Fraud). O objetivo é identificar e rastrear transações Pix suspeitas logo após o comprometimento de contas, um cenário muito comum em incidentes de segurança bancária.

Minha transição natural da engenharia e do estudo de vetores de ataque (como análise de phishing) para a análise de dados me levou a este case: entender como o dinheiro se move após a invasão e como podemos usar SQL para bloquear o ataque a tempo.

## 🎯 O Desafio
O desafio consistiu em duas frentes de investigação de transações de alto valor (acima de R$ 5.000,00):
1. **Identificação de Contas Laranjas:** Mapear transferências saindo de contas recém-criadas (menos de 48 horas).
2. **Rastreamento de Cash-out (Ranking):** Isolar a conta de destino que recebeu a maior transação, permitindo ação rápida da equipe de bloqueio.

## 🛠️ Tecnologias e Conceitos Utilizados
* **SQL** (Sintaxe compatível com Databricks/Spark)
* **JOINS** para cruzamento de dados transacionais e cadastrais.
* **CTEs (Common Table Expressions)** para organização lógica da query.
* **Window Functions (`ROW_NUMBER`)** para criação de rankings particionados sem perder a granularidade dos dados.

Para resolver a missão final de rastreamento do recebedor principal, desenvolvi a seguinte consulta utilizando funções de janela:

```sql
WITH RankingTransacoes AS (
    -- Passo 1: Criar um ranking das maiores transações recebidas por cada conta
    SELECT 
        tx_id,
        destino_cliente_id,
        origem_cliente_id,
        valor,
        data_hora,
        ROW_NUMBER() OVER(
            PARTITION BY destino_cliente_id 
            ORDER BY valor DESC
        ) AS ranking
    FROM transacoes_pix
)

-- Passo 2: Filtrar apenas a transação número 1 (maior valor) e cruzar com os dados do cliente
SELECT 
    r.tx_id,
    r.destino_cliente_id,
    c.nome AS nome_recebedor,
    r.origem_cliente_id,
    r.valor,
    r.data_hora
FROM RankingTransacoes r
JOIN clientes c ON r.destino_cliente_id = c.cliente_id
WHERE r.ranking = 1;
