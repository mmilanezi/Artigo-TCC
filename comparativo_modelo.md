# Comparativo de Modelos — Sistema Pré-Voo TCC/UNISINOS

**Autor:** mmgrassi  
**Data:** 09/05/2026  
**Contexto:** Trabalho de Conclusão de Curso — Sistema de Apoio à Decisão Pré-Voo para Aviação Civil Brasileira

---

## 1. Resumo Executivo

A migração do Sistema v1.x (4 classes) para o Sistema Beta 4.0 (3 classes) resultou em ganhos
expressivos em todas as métricas, com destaque para o **F1 Macro: de 0.7439 → 0.9069 (+21.9%)**.
O avanço é explicado por três mudanças metodológicas simultâneas documentadas abaixo.

---

## 2. Versões Comparadas

| Campo                  | Sistema Beta 3.8 (ref. v1.6)          | Sistema Beta 4.0 (beta-2.0)             |
|------------------------|-----------------------------------|-----------------------------------------|
| Versão do modelo       | 1.6 / 1.7 / 1.8                  | beta-2.0                                |
| Data do treino         | 19/04/2026                        | 09/05/2026                              |
| Número de classes      | **4**                             | **3**                                   |
| Classes                | Voo Normal, Incidente, Inc. Grave, Acidente | Voo Normal, Incidente, Acidente |
| Split de dados         | Sem separação formal de teste     | **80% treino / 15% validação / 5% teste** |
| Conjunto de avaliação  | Interno (potencial data leakage)  | Teste exclusivo — nunca visto no treino |
| SMOTE                  | Sim                               | Sim (apenas no treino)                  |
| Meta-learner           | Regressão Logística               | Regressão Logística                     |
| Base learners          | RF, SVM, MLP, XGBoost             | RF, SVM, MLP, XGBoost                  |

---

## 3. Mudanças Metodológicas

### 3.1 Fusão de Classes: 4 → 3 (Japkowicz & Stephen, 2002)

A classe **Incidente Grave** foi fundida à classe **Incidente** pelos seguintes motivos:

- Representava apenas ~6,8% dos dados no conjunto de treino (< 10%, limiar recomendado).
- F1-Score de apenas **0.31** no modelo v1.6 — o modelo não conseguia discriminá-la de forma confiável.
- Análise exploratória revelou alta sobreposição de features com a classe Incidente.
- Japkowicz & Stephen (2002) recomendam fusão quando a separabilidade empírica é baixa e a diferença de consequência operacional não justifica a complexidade adicional.

**Resultado:** A fronteira de decisão tornou-se mais nítida para as três classes restantes.

### 3.2 Split 80/15/5 (Varma & Simon, 2006 | Hastie et al., 2009)

Substituição do split binário (treino/teste) pelo split tripartido:

| Conjunto    | Proporção | Tamanho (n) | Uso                                              |
|-------------|-----------|-------------|--------------------------------------------------|
| Treino      | 80%       | 14.536      | Ajuste dos parâmetros dos modelos + SMOTE        |
| Validação   | 15%       | 2.726       | Seleção de hiperparâmetros (Fase 4 — diagnóstico)|
| Teste       | 5%        | 909         | Avaliação final exclusiva (Fase 5 — Stacking)    |

**Por que importa:** Varma & Simon (2006) demonstram que usar o mesmo conjunto para seleção de modelo
e avaliação final introduz viés otimista mensurável. O conjunto de teste (5%) permaneceu intocado
durante todo o processo de desenvolvimento, garantindo estimativa de desempenho sem contaminação.

### 3.3 Fase 4 como Diagnóstico (Wolpert, 1992)

No sistema v1.x, a Fase 4 selecionava um "modelo vencedor" entre os quatro candidatos.
No Beta 4.0, a Fase 4 passa a ser um **diagnóstico dos base learners** — todos alimentam o
Stacking Ensemble (Wolpert, 1992), pois são complementares, não competidores.
O conjunto de validação (15%) é usado apenas para diagnóstico nesta fase; a avaliação definitiva
acontece na Fase 5 com o conjunto de teste (5%).

---

## 4. Comparativo de Desempenho — Stacking Ensemble

### 4.1 Métricas Globais

| Métrica        | v1.6 (4 classes) | beta-2.0 (3 classes) | Δ absoluto | Δ relativo |
|----------------|:----------------:|:--------------------:|:----------:|:----------:|
| Acurácia       | 0.8790           | **0.9307**           | +0.0517    | +5.9%      |
| F1 Macro       | 0.7439           | **0.9069**           | +0.1630    | +21.9%     |
| F1 Weighted    | 0.8753           | **0.9315**           | +0.0562    | +6.4%      |
| ROC AUC        | 0.9500           | **0.9842**           | +0.0342    | +3.6%      |
| Tempo de treino| 991.4 s          | **403.6 s**          | −587.8 s   | −59.3%     |

> **Nota:** A melhora no F1 Macro (+21.9%) é a métrica mais relevante para o domínio,
> pois penaliza igualmente erros em todas as classes independentemente do suporte.
> A redução de tempo de treino reflete a menor complexidade com 3 classes.

### 4.2 Métricas por Classe

#### Sistema v1.6 — 4 classes (conjunto de teste interno)

| Classe          | Precision | Recall | F1-Score | Support |
|-----------------|:---------:|:------:|:--------:|:-------:|
| Voo Normal      | 1.00      | 1.00   | 1.00     | 1.000   |
| Incidente       | 0.92      | 0.93   | 0.93     | 1.827   |
| Incidente Grave | 0.35      | 0.28   | **0.31** | 248     |
| Acidente        | 0.73      | 0.75   | 0.74     | 560     |
| **Macro avg**   | **0.75**  | **0.74** | **0.74** | 3.635 |

#### Sistema Beta 4.0 — 3 classes (conjunto de teste exclusivo 5%)

| Classe          | Precision | Recall | F1-Score | Support |
|-----------------|:---------:|:------:|:--------:|:-------:|
| Voo Normal      | 1.00      | 1.00   | 1.00     | 250     |
| Incidente       | 0.95      | 0.93   | **0.94** | 519     |
| Acidente        | 0.76      | 0.81   | **0.78** | 140     |
| **Macro avg**   | **0.90**  | **0.91** | **0.91** | 909   |

#### Interpretação por Classe

**Voo Normal:** Desempenho perfeito mantido nas duas versões (F1=1.00). A classe é bem separada
pelas features operacionais.

**Incidente:** F1 saltou de 0.93 → **0.94** mesmo após absorver os casos de Incidente Grave.
Indica que a fusão foi bem-sucedida: os casos de Inc. Grave eram, de fato, mais similares a
Incidentes do que a Acidentes.

**Incidente Grave (v1.x):** F1=0.31 era o ponto crítico do modelo anterior. Precisão de 0.35
significa que 65% das predições nessa classe eram falso-positivos — operacionalmente inaceitável.
A fusão eliminou esse gargalo.

**Acidente:** F1 subiu de 0.74 → **0.78**. Com a remoção da confusão entre Acidente e
Incidente Grave (frequentes nas colunas da matriz de confusão), o modelo passou a discriminar
melhor os acidentes reais.

### 4.3 Matriz de Confusão — Beta 4.0

```
                 Predito →
Real ↓        Voo Normal   Incidente   Acidente
Voo Normal         250           0          0     ← Perfeito
Incidente            0         483         36     ← 6.9% classificados como Acidente
Acidente             0          27        113     ← 19.3% classificados como Incidente
```

**Análise operacional:**
- Falsos negativos de Acidente (27 casos classificados como Incidente): risco mais crítico.
  Recall de 81% significa que 19% dos acidentes são subestimados — área de melhoria futura.
- Zero confusão com Voo Normal em nenhuma direção: o sistema nunca libera um voo problemático
  como "normal", o que é o requisito de segurança mais importante.

---

## 5. Comparativo dos Base Learners (Fase 4 — Diagnóstico)

Base learners avaliados no conjunto de **validação (15%)** — valores diagnósticos, não definitivos.

| Modelo              | Acurácia | F1 Macro | ROC AUC | Tempo (s) |
|---------------------|:--------:|:--------:|:-------:|:---------:|
| Random Forest       | 0.9255   | 0.9022   | 0.9825  | 3.1       |
| XGBoost             | 0.9255   | 0.8981   | 0.9831  | 5.1       |
| SVM                 | 0.9109   | 0.8902   | 0.9783  | 135.2     |
| Rede Neural (MLP)   | 0.9193   | 0.8895   | 0.9762  | 5.6       |
| **Stacking (teste)**| **0.9307** | **0.9069** | **0.9842** | 403.6 |

O Stacking supera todos os base learners individuais, confirmando a validade da abordagem
de Wolpert (1992): a combinação de modelos complementares é superior ao melhor modelo individual.

---

## 6. Evolução Histórica (todas as versões)

| Versão    | Sistema  | Data        | F1 Macro (stacking) | ROC AUC |
|-----------|----------|-------------|:-------------------:|:-------:|
| 1.6       | v1.x     | 19/04/2026  | 0.7439              | 0.9500  |
| v1.7      | v1.x     | 19/04/2026  | 0.7416              | 0.9492  |
| **beta-2.0** | **Beta 4.0** | **09/05/2026** | **0.9069**  | **0.9842** |

A estagnação entre v1.6 e v1.7 indica que o teto do modelo de 4 classes havia sido atingido.
A transição para 3 classes com split formal desbloqueou o espaço de melhoria.

---

## 7. Conclusão

O avanço do Sistema v1.x para o Beta 4.0 não é resultado de ajuste fino de hiperparâmetros,
mas de **três decisões metodológicas** embasadas na literatura:

1. **Fusão de classes** (Japkowicz & Stephen, 2002) — eliminação de classe indetectável que
   deteriorava o F1 Macro sem benefício operacional.
2. **Split 80/15/5** (Varma & Simon, 2006) — separação rigorosa entre desenvolvimento e
   avaliação, garantindo estimativa honesta do desempenho real.
3. **Reframing da Fase 4** (Wolpert, 1992) — base learners são complementares; o ensemble
   é o classificador final, não uma competição entre modelos individuais.

**O resultado é um sistema com F1 Macro de 0.9069 e ROC AUC de 0.9842, avaliado em conjunto
de teste exclusivo, pronto para integração no sistema pré-voo.**

---

## 8. Referências

- Hastie, T., Tibshirani, R., & Friedman, J. (2009). *The Elements of Statistical Learning* (2ª ed.). Springer.
- Harms, R. (2020). Neural networks for predicting safety-critical events in aviation. *Safety Science*.
- Japkowicz, N., & Stephen, S. (2002). The class imbalance problem: A systematic study. *Intelligent Data Analysis*, 6(5), 429–449.
- Liu, Y. et al. (2024). SVM-based prediction of hazardous meteorological conditions. *Journal of Aviation*.
- Sokolova, M., & Lapalme, G. (2009). A systematic analysis of performance measures for classification tasks. *Information Processing & Management*, 45(4), 427–437.
- Varma, S., & Simon, R. (2006). Bias in error estimation when using cross-validation for model selection. *BMC Bioinformatics*, 7, 91.
- Walden, A. et al. (2023). Random Forest and XGBoost for aviation safety prediction. *Aerospace*.
- Wolpert, D. H. (1992). Stacked generalization. *Neural Networks*, 5(2), 241–259.
