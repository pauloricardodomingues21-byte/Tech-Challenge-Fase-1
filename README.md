# Tech Challenge Fase 1 – NPS Preditivo

Projeto do Tech Challenge da Fase 1 da Pós Tech. O objetivo é entender quais fatores operacionais influenciam a satisfação do cliente de um e-commerce e prever, antes da pesquisa de NPS, quais clientes têm risco de se tornar detratores.

## Principais resultados

- O NPS atual da base é **−2**: 24% de detratores, 22% de promotores e 54% de neutros.
- O **atraso na entrega** é o fator mais importante. Sem atraso, 10% dos clientes viram detratores; com 3 a 5 dias, 56%. O ponto de ruptura está no 3º dia.
- O **tempo de resolução de problemas** é o segundo fator: acima de 8 dias, 39% viram detratores. O número de contatos com o atendimento não altera a nota.
- Idade, região e tempo de relacionamento **não** explicam a nota (média de 7,4 em todos os grupos).
- O modelo de classificação (regressão logística) identifica **83% dos detratores**, com precisão 1,7 vez maior que uma escolha aleatória.

## Base de dados

Arquivo CSV fornecido pelo curso, com 5.000 pedidos e 21 colunas:

| Grupo | Colunas |
| --- | --- |
| Cliente | `customer_id`, `customer_age`, `customer_region`, `customer_tenure_months` |
| Pedido | `order_id`, `order_value`, `items_quantity`, `discount_value`, `payment_installments` |
| Entrega | `delivery_time_days`, `delivery_delay_days`, `freight_value`, `delivery_attempts` |
| Atendimento | `customer_service_contacts`, `resolution_time_days`, `complaints_count` |
| Satisfação e negócio | `nps_score` (alvo, 0 a 10), `csat_internal_score`, `repeat_purchase_30d` |

Há 60 valores ausentes em `customer_age` e 60 em `order_value`, tratados com a mediana no modelo.

## Estrutura do repositório

```
├── data/
│   └── base_nps_teste.csv              # base de dados
├── notebooks/
│   ├── eda_nps_colab.ipynb             # tratamento e análise exploratória
│   └── CSV_modelo_nps_colab.ipynb      # modelo preditivo
├── reports/
│   ├── NPS_Preditivo_Tech_Challenge.pptx        # apresentação executiva
│   └── Tech_Challenge_Fase1_NPS_Preditivo.pdf   # relatório completo
└── README.md
```

Ao rodar os notebooks, os gráficos são salvos em `reports/` e os modelos treinados em `models/`.

## Metodologia

**1. Entendimento do negócio e definição da target.** A variável alvo é `nps_score`. Para o modelo, ela foi agrupada no padrão do NPS, e o alvo é `detrator = 1` quando a nota é de 0 a 6.

**2. Análise exploratória.** Distribuição do NPS, percentual de detratores por faixa de cada fator operacional (atraso, resolução, reclamações, tentativas de entrega, contatos), perfil do cliente, correlações e relação entre NPS e recompra.

**3. Modelo preditivo.**
- Variáveis removidas para evitar vazamento de dados: `csat_internal_score` (outra medida de satisfação) e `repeat_purchase_30d` (acontece depois do pedido), além dos identificadores.
- Pré-processamento em `Pipeline` do scikit-learn: mediana e padronização nas variáveis numéricas, moda e one-hot encoding na região.
- Divisão 80% treino e 20% teste, estratificada, com `random_state=42`.
- Comparação entre regressão (Ridge e Gradient Boosting) e classificação (regressão logística e Gradient Boosting).

| Abordagem | Modelo | Resultado no teste |
| --- | --- | --- |
| Regressão (nota 0–10) | Ridge | MAE 0,90 · R² 0,289 |
| Regressão (nota 0–10) | Gradient Boosting | MAE 0,92 · R² 0,264 |
| Classificação (detrator) | **Regressão logística (escolhido)** | **ROC-AUC 0,783** |
| Classificação (detrator) | Gradient Boosting | ROC-AUC 0,763 |

Com limiar de 0,40, o modelo escolhido tem recall de 83% e precisão de 41% para detratores. O limiar prioriza encontrar detratores, já que contatar um cliente satisfeito custa pouco.

## Como reproduzir

1. Abra o notebook desejado em `notebooks/` e clique no botão **Open in Colab**.
2. Rode todas as células (**Ambiente de execução → Executar tudo**).
3. Quando a primeira célula pedir, envie o arquivo `data/base_nps_teste.csv`.
4. Rode primeiro o notebook de EDA e depois o do modelo. Os resultados aparecem no próprio notebook.

Bibliotecas usadas: pandas, NumPy, matplotlib, scikit-learn e joblib (todas já disponíveis no Google Colab).

## Limitações

- As variáveis mais importantes (atraso, reclamações, tempo de resolução) só existem durante ou após a entrega: o modelo antecipa a pesquisa, não o problema.
- Os resultados mostram associação, não causalidade. As recomendações devem ser validadas em piloto.
- Algumas faixas têm poucos casos (por exemplo, apenas 8 pedidos com 6 a 10 dias de atraso).

## Autores

[Paulo Ricardo Domingues Cezario]

