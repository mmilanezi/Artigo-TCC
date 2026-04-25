## Citação Indireta — Texto Acadêmico para o Artigo do TCC

### Contexto e Motivação (Referencial Teórico — Seção 2.4)

O desenvolvimento de sistemas inteligentes de apoio à tomada de decisão na aviação civil insere-se em um movimento global de transição do paradigma Safety I, de caráter reativo, centrado na investigação de falhas após sua ocorrência, para o paradigma Safety II, de natureza proativa e preditiva, voltado à compreensão de por que as operações aéreas são bem-sucedidas e ao fortalecimento da resiliência sistêmica (ZIAKKAS; PECHLIVANIS, 2024). Nesse contexto, Kühl et al. (2022) distinguem Inteligência Artificial (IA) de Aprendizado de Máquina (ML), esclarecendo que IA é o campo mais amplo voltado ao desenvolvimento de agentes capazes de perceber e agir racionalmente, enquanto o ML constitui o subconjunto de métodos que permitem ao sistema aprender padrões a partir de dados sem que regras sejam explicitamente programadas. A convergência dessas tecnologias com a aviação é documentada por Demir, Moslem e Duleba (2024), cuja revisão sistemática de 224 estudos publicados entre 2004 e 2024 evidencia crescimento expressivo na aplicação de ML, aprendizado profundo e modelos de séries temporais à análise de acidentes, previsão de eventos de risco e melhoria das medidas de segurança operacional.

### Abordagem do Problema — FRAT e Machine Learning

A limitação das ferramentas tradicionais de avaliação de risco pré-voo, como o *Flight Risk Assessment Tool* (FRAT), é amplamente reconhecida. Conforme Walden et al. (2023), esses instrumentos dependem de entradas manuais sujeitas a viés cognitivo do operador, não integram dados históricos de acidentes e incidentes de forma automática e não produzem estimativas quantitativas robustas de risco. Os autores demonstram que modelos de aprendizado de máquina, como XGBoost e Random Forest, treinados com mais de 40 anos de registros do National Transportation Safety Board (NTSB), são capazes de prever categorias de risco com acurácia moderadamente elevada e com menor subjetividade do que os métodos tradicionais. Na mesma direção, Feldman et al. (2023) descrevem o sistema NASA IASMS (In-Time Aviation Safety Management System), que integra dados de múltiplas fontes aeronáuticas e ambientais para gerar diagnósticos preditivos durante o planejamento pré-voo, demonstrando que a automação dessa análise aumenta a consciência situacional do operador em cenários de risco meteorológico complexo.
- ok
### Técnicas de Machine Learning Utilizadas

O modelo preditivo desenvolvido neste trabalho foi construído sobre o paradigma de aprendizado de máquina supervisionado, no qual o algoritmo aprende a partir de um conjunto de exemplos rotulados, ajustando seus parâmetros internos de modo a melhorar progressivamente seu desempenho em uma tarefa específica (MITCHELL, 1997). Nesse contexto, cada registro do banco de dados do CENIPA foi associado a um rótulo de gravidade (Classes), como Voo Normal (0), Incidente (1), Incidente Grave (2) ou Acidente (3), fornecendo ao modelo a experiência de treinamento necessária para generalizar o padrão de risco a novos voos não observados anteriormente.

Para a composição dos modelos base, foram selecionados quatro algoritmos de naturezas distintas, escolha motivada pela necessidade de diversidade entre os preditores. O primeiro deles, o Random Forest (RF), é descrito por Hastie, Tibshirani e Friedman (2009) como uma variação substancial do bagging que constrói uma grande coleção de árvores descorrelacionadas e combina suas predições por voto majoritário, atingindo desempenho comparável ao boosting com menor custo de ajuste. Géron (2019) complementa que o algoritmo introduz aleatoriedade adicional na seleção de variáveis em cada nó, reduzindo a variância sem aumentar proporcionalmente o viés. Walden et al. (2023) confirmam sua eficácia no contexto aeronáutico, apontando o Random Forest como um dos dois modelos de melhor desempenho dentre XGBoost, RF, redes neurais e BERT para categorização de risco pré-voo.
-ok

O segundo modelo base, o XGBoost, é uma implementação otimizada do Gradient Boosting, fundamentado matematicamente por Hastie, Tibshirani e Friedman (2009) como a minimização de uma função de perda no espaço funcional por meio de gradiente descendente, e é reconhecido por Géron (2019) como componente frequente nas soluções vencedoras de competições de aprendizado de máquina. Sua capacidade de capturar interações complexas entre variáveis operacionais e meteorológicas, aliada à relativa interpretabilidade via importância de features, justificou sua adoção tanto neste trabalho quanto no FRAT modernizado proposto por Walden et al. (2023), onde foi utilizado como modelo de referência.

O terceiro modelo base, a Máquina de Vetores de Suporte com kernel de função de base radial (SVM-RBF), é especialmente indicado para dados meteorológicos por sua capacidade de separar classes com fronteiras de decisão complexas e não-lineares no espaço de características transformado pelo kernel (HASTIE; TIBSHIRANI; FRIEDMAN, 2009; GÉRON, 2019). ~melhorar~ Liu et al. (2024) validam diretamente essa escolha ao aplicar SVM-RBF para previsão de condições meteorológicas perigosas para a aviação, incluindo tempestades, turbulência e baixa visibilidade, utilizando as mesmas variáveis do presente trabalho (temperatura, umidade, velocidade do vento) e além da utilização de precipitação, e obtendo desempenho superior ao de Random Forest e LSTM nas métricas de acurácia, ROC-AUC e F1-score. ~melhorar~. Confusão no que foi usado neste trabalho com o trabalho do Liu

O quarto modelo base, o Perceptron Multicamadas (MLP), é uma rede neural artificial do tipo feedforward treinada por retropropagação do gradiente, cuja arquitetura com camadas ocultas e função de ativação não-linear permite aprender representações hierárquicas dos dados (HASTIE; TIBSHIRANI; FRIEDMAN, 2009; MITCHELL, 1997). Harms (2020) demonstra, em contexto aeronáutico real, que redes MLP com otimizador ADAM, ativação ReLU e early stopping são capazes de identificar precursores indiretos de eventos de segurança de voo, com os parâmetros meteorológicos ocupando posição dominante no ranking de relevância de features, atribuído por meio do algoritmo Relief.

### Stacking Ensemble e Validação Cruzada
-ok
A combinação dos quatro modelos base foi realizada pela técnica de Stacking Generalizado, proposta originalmente por Wolpert (1992). Segundo o autor, ao invés de adotar uma estratégia de escolha única do melhor modelo, o stacking combina os preditores de maneira mais sofisticada, utilizando suas predições como entradas de um novo espaço de representação, o nível 1, no qual um meta-aprendiz é treinado para encontrar a combinação ótima entre eles. Hastie, Tibshirani e Friedman (2009) formalizam essa abordagem e demonstram que a combinação de modelos nunca piora o desempenho em relação a qualquer modelo individual, quando estimada no nível populacional. Para garantir que as predições utilizadas no treinamento do meta-aprendiz fossem não enviesadas, utilizou-se a técnica de predições out-of-fold com validação cruzada de 5 partições. Wolpert (1992) define formalmente essa partição como o mecanismo central do stacking, e Géron (2019) reforça que esse procedimento assegura que as predições sejam limpas, uma vez que os preditores de nível zero nunca treinaram sobre as instâncias reservadas para geração das meta-features. Walden et al. (2023) adotam procedimento equivalente (n-fold cross-validation) para garantir a generalização dos modelos de FRAT modernizado.
-Ok mas precisa explicar as técnicas utilizadas, como o que são.

Como meta-aprendiz de nível 1, foi adotada a Regressão Logística Multinomial com regularização L2 (C = 1,0). A escolha por um modelo linear como combinador é fundamentada por Wolpert (1992) e Hastie, Tibshirani e Friedman (2009), que alertam que modelos mais simples tendem a generalizar melhor nesse papel, pois o espaço de meta-features já contém a informação discriminativa destilada pelos modelos de nível 0.
-ok
### Pré-Processamento e Balanceamento

No que se refere ao pré-processamento, as features numéricas foram padronizadas pelo StandardScaler, prática explicitamente recomendada por Hastie, Tibshirani e Friedman (2009) antes do treinamento de redes neurais, enquanto as features categóricas foram transformadas pelo OneHotEncoder. Para mitigar o desbalanceamento de classes inerente ao banco de dados do CENIPA, foi aplicada a técnica SMOTE (Synthetic Minority Over-sampling Technique), que gera amostras sintéticas das classes minoritárias por interpolação no espaço de features (CHAWLA et al., 2002). O uso de oversampling em dados aeronáuticos desbalanceados é corroborado por Liu et al. (2024), que identificam a raridade de eventos meteorológicos perigosos como causa primária do desequilíbrio de classes, e por Harms (2020), cujos experimentos demonstram que oversampling é etapa metodológica essencial para a aprendizabilidade do problema, sem ela, o modelo falha completamente em aprender a classe positiva (safety event).
-ok Explicar os termos
### Métricas de Avaliação
A avaliação dos modelos candidatos foi conduzida por meio de três métricas principais: acurácia, F1-score macro e ROC-AUC. A acurácia expressa a proporção de previsões corretas sobre o total de instâncias avaliadas, constituindo a medida mais intuitiva de desempenho; contudo, Liu et al. (2024) confirmam que ela pode ser enganosa em conjuntos com classes desbalanceadas, uma vez que um classificador trivial que prevê sempre a classe majoritária obtém valores elevados sem qualquer capacidade preditiva real sobre as classes raras, limitação igualmente demonstrada por Harms (2020) em dados de safety events aeronáuticos, onde a prevalência da classe negativa próxima a 97% tornaria a acurácia isolada um critério insuficiente. O F1-Score Macro é a média harmônica entre Precisão, proporção de previsões positivas corretas, e Revocação, proporção de casos positivos reais identificados, calculada individualmente para cada classe e depois com média aritmética entre elas; a variante macro atribui peso igual a todas as classes independentemente de sua frequência, propriedade que Sokolova e Lapalme (2009) formalizam como "treats all classes equally regardless of their frequency", essencial para que as classes de Incidente Grave e Acidente, numericamente raras, não sejam sacrificadas em favor das majoritárias. Liu et al. (2024) e Walden et al. (2023) adotam o F1-score como métrica de avaliação comparativa em seus respectivos estudos aeronáuticos. O ROC-AUC (Receiver Operating Characteristic - Area Under the Curve) resume a capacidade discriminativa do classificador em todos os limiares de decisão possíveis em um único escalar entre 0,5 desempenho aleatório e 1,0 classificador perfeito; Fawcett (2006) demonstra que o AUC equivale à probabilidade de que o modelo atribua pontuação de risco mais alta a um voo perigoso do que a um voo normal escolhidos ao acaso, e destaca sua insensibilidade à distribuição de classes, propriedade relevante em conjuntos aeronáuticos desbalanceados.
- ok
A seleção do modelo final foi fundamentada no F1-Score Macro como critério primário, por sua propriedade de atribuir peso igual a todas as classes independentemente de sua frequência o que Chicco e Jurman (2020) reforçam ao demonstrar que métricas sensíveis ao desequilíbrio de classes, como a acurácia isolada, são insuficientes em cenários com distribuição assimétrica. Após a validação cruzada de 5 partições para cada modelo candidato, o F1 Macro médio foi calculado por fold; o modelo com maior média o Stacking Ensemble com meta-learner de Regressão Logística foi selecionado como modelo final. Acurácia e ROC-AUC foram calculados e reportados no relatório de avaliação como métricas complementares de diagnóstico, permitindo comparação com estudos similares (LIU et al., 2024; HARMS, 2020), sem, contudo, integrarem o critério de seleção.

### Referências Bibliográficas (ABNT)

CHAWLA, N. V.; BOWYER, K. W.; HALL, L. O.; KEGELMEYER, W. P. SMOTE: Synthetic Minority Over-sampling Technique. **Journal of Artificial Intelligence Research**, v. 16, p. 321–357, 2002.

CHICCO, Davide; JURMAN, Giuseppe. The advantages of the Matthews correlation coefficient (MCC) over F1 score and accuracy in binary classification evaluation. **BMC Genomics**, v. 21, n. 6, 2020. DOI: 10.1186/s12864-019-6413-7.

DEMIR, Gülay; MOSLEM, Sarbast; DULEBA, Szabolcs. Artificial Intelligence in Aviation Safety: Systematic Review and Biometric Analysis. **International Journal of Computational Intelligence Systems**, v. 17, n. 279, 2024. DOI: 10.1007/s44196-024-00671-w.

FAWCETT, Tom. An introduction to ROC analysis. **Pattern Recognition Letters**, v. 27, n. 8, p. 861–874, 2006. DOI: 10.1016/j.patrec.2005.10.010.

FELDMAN, Jolene; MARTIN, Lynne; COSTEDOAT, Gregory; GUJRAL, Vimmy. Usability of Pre-Flight Planning Interfaces for Supplemental Data Service Provider Tools to Support Uncrewed Aircraft System Traffic Management. **Application of Emerging Technologies**, v. 115, p. 357–366, 2023. DOI: 10.54941/ahfe1004333.

GÉRON, Aurélien. **Hands-On Machine Learning with Scikit-Learn, Keras, and TensorFlow**. 2. ed. Sebastopol: O'Reilly Media, 2019.

HARMS, T. **A Machine Learning Approach to Flight Safety Event Prediction**. 2020. Dissertação (Mestrado em Engenharia Aeroespacial) — Delft University of Technology, Delft, 2020.

HASTIE, Trevor; TIBSHIRANI, Robert; FRIEDMAN, Jerome. **The Elements of Statistical Learning: Data Mining, Inference, and Prediction**. 2. ed. New York: Springer, 2009.

KIRWAN, Barry. The Impact of Artificial Intelligence on Future Aviation Safety Culture. **Future Transportation**, v. 4, p. 349–379, 2024. DOI: 10.3390/futuretransp4020018.

KÜHL, Niklas; SCHEMMER, Max; GOUTIER, Marc; SATZGER, Gerhard. Artificial Intelligence and Machine Learning: Untangling Concepts and Terminology for AI-Based Information Systems. **Electronic Markets**, 2022. DOI: 10.1007/s12525-022-00598-0.

LIU, Haoxing; XIE, Renjie; QIN, Haoshen; LI, Yizhou. Research on dangerous flight weather prediction based on machine learning. **Journal of Physics: Conference Series**, v. 2870, n. 012020, 2024. DOI: 10.1088/1742-6596/2870/1/012020.

MITCHELL, Tom M. **Machine Learning**. New York: McGraw-Hill, 1997.

SOKOLOVA, Marina; LAPALME, Guy. A systematic analysis of performance measures for classification tasks. **Information Processing & Management**, v. 45, n. 4, p. 427–437, 2009. DOI: 10.1016/j.ipm.2009.03.002.

WALDEN, Hunter; HILL, Kathleen; QUINN, Chi; TORRES, Erick; GANG, Isaac. Proposal for a Modernized Flight Risk Assessment Tool for General Aviation Pre-Flight Planning. George Mason University, 2023.

WOLPERT, David H. Stacked Generalization. **Neural Networks**, v. 5, n. 2, p. 241–259, 1992.

ZIEJA, Mariusz; SMOLIŃSKI, Henryk; GOŁDA, Paweł. Information Systems as a Tool for Supporting the Management of Aircraft Flight Safety. **Archives of Transport**, v. 36, n. 4, p. 67–76, 2015. DOI: 10.5604/08669546.1185211.

ZIAKKAS, Dimitrios; PECHLIVANIS, Konstantinos. The Role of Artificial Intelligence in Transportation Safety Management Systems: The Aviation Safety II Case Study. **Safety Management and Human Factors**, v. 151, p. 1–9, 2024. DOI: 10.54941/ahfe1005296.



NOVO - A PARTIR DAQUI

O sistema incorpora, em adição ao modelo preditivo, uma **camada de validação de segurança baseada em regras operacionais**, aplicada sobre a saída do classificador antes da geração do relatório de avaliação de risco. A motivação para esse componente parte de um princípio amplamente consolidado na literatura de IA aplicada à aviação: sistemas algorítmicos devem atuar como ferramentas de suporte à decisão dentro de um framework normativo, e não como substitutos das regras operacionais estabelecidas por organismos regulatórios (ZIAKKAS; PECHLIVANIS, 2024; KIRWAN, 2024). Kirwan (2024, p. 365) formaliza essa exigência ao introduzir o conceito de *golden rules* — limites operacionais que sistemas de IA em aviação não devem violar independentemente de sua lógica interna — e exemplifica com os mínimos de separação aérea como instância paradigmática desse princípio.

A camada opera em dois níveis determinísticos derivados de normas regulatórias internacionais. O **Nível L1** identifica condições meteorológicas adversas — vento acima do limiar crítico definido pelo JAR/FAR 25.237 para certificação de aeronaves, precipitação igual ou superior à classe "Chuva Moderada" da ANAC, condições de nevoeiro em superfície segundo a WMO (2025), ou temperatura compatível com formação de gelo em voo conforme o RBAC 121.629(b) — e eleva o status de qualquer predição favorável do modelo para APTO_MODERADO. O **Nível L2** identifica condições críticas — ventos acima do limiar de alerta de ciclone tropical do ICAO Annex 3 §5.1.1, rajadas no limite superior de microexplosão segundo o TP-2001-217, chuva forte segundo a ANAC, ou gelo confirmado segundo o ICAO Annex 3 §4.6.2.2 — e força o status NAO_APTO independentemente da predição do modelo. Adicionalmente, a camada detecta contradições entre a predição do classificador e as condições meteorológicas observadas — especificamente, quando o modelo prediz acidente com alta confiança em um cenário meteorologicamente benigno —, emitindo status INCONCLUSIVO em vez de propagar uma predição inconsistente com os dados físicos disponíveis.

Essa arquitetura é coerente com o framework de Sistemas de Gerenciamento de Segurança (SMS) promovido pela ICAO e pela EASA, que preconiza a existência de múltiplos mecanismos de mitigação de risco independentes e complementares (ZIAKKAS; PECHLIVANIS, 2024, p. 5; KIRWAN, 2024, p. 373). O modelo estatístico constitui a primeira barreira — avaliação probabilística de risco baseada em padrões históricos do CENIPA; a camada de validação operacional constitui a segunda barreira — conformidade determinística com limites físicos normativos. A independência entre as duas barreiras garante que limitações estruturais do modelo (viés de amostragem, extrapolação fora da distribuição de treinamento) não se propaguem para a avaliação final apresentada ao operador. Kirwan (2024, p. 376) fundamenta essa exigência ao observar que modelos algorítmicos operam essencialmente calculando probabilidades históricas, sem capacidade de reconhecer o valor da segurança da maneira que normas operacionais e julgamento humano o fazem — tornando indispensável a existência de restrições externas que delimitem o espaço de ação do classificador em contextos de risco real.
 


BRANDÃO, Roberto. **FGA Descomplicada**: meteorologia aeronáutica aplicada. [S.l.: s.n.], 2016.

FELDMAN, Jolene; MARTIN, Lynne; COSTEDOAT, Gregory; GUJRAL, Vimmy. Usability of Pre-Flight Planning Interfaces for Supplemental Data Service Provider Tools to Support Uncrewed Aircraft System Traffic Management. **Application of Emerging Technologies**, v. 115, p. 357–366, 2023. DOI: 10.54941/ahfe1004333.

HARMS, Tyler. **A Machine Learning Approach to Flight Safety Event Prediction**. 2020. Dissertação (Mestrado) — Iowa State University, Ames, Iowa, 2020.

ICAO. **Annex 3 to the Convention on International Civil Aviation — Meteorological Service for International Air Navigation**. 20. ed. Montreal: International Civil Aviation Organization, 2018.

KIRWAN, Barry. The Impact of Artificial Intelligence on Future Aviation Safety Culture. **Future Transportation**, v. 4, n. 2, p. 349–379, 2024. DOI: 10.3390/futuretransp4020018.

WALDEN, Christopher T.; RUSNOCK, Christina F.; BOUBIN, Jeremiah G. Proposal for a Modernized Flight Risk Assessment Tool Using Machine Learning. **Journal of Aviation/Aerospace Education & Research**, v. 32, n. 1, 2023. DOI: 10.15394/jaaer.2023.1914.

WMO. **Fog and Low-Visibility Procedures at Aerodromes**. Geneva: World Meteorological Organization, 2025.

ZIAKKAS, Dimitrios; PECHLIVANIS, Konstantinos. The Role of Artificial Intelligence in Transportation Safety Management Systems: The Aviation Safety II Case Study. **Aerospace**, v. 11, n. 2, p. 132, 2024. DOI: 10.3390/aerospace11020132.

ZIEJA, Mariusz; SMOLIŃSKI, Henryk; GOŁDA, Paweł. Information Systems as a Tool for Supporting the Management of Aircraft Flight Safety. **The Archives of Transport**, v. 36, n. 4, p. 67–76, 2015. DOI: 10.5604/08669546.1185211.



 

Grupo	Parâmetro	Valor	Documento	Seção / Artigo	Justificativa
 	vento forte	> 7,5 m/s	ICAO Annex 3, 11ª ed.	§7.4.3 e §7.4.4	Windshear ≥ 15 kt crítico para trajetórias de decolagem e pouso
	Vento	> 7,5 m/s	RBAC 121 Emenda 07	Art. 121.429(a)	Vento de través > 15 nós ativa procedimentos especiais obrigatórios
	Rajada	> 12,5 m/s	ICAO Annex 3, 11ª ed.	Apêndice 3, §4.1.5.2 c)2)	Rajada ≥ 5 m/s acima de vento médio de 7,5 m/s = limiar para reportagem no METAR
 	umidade alta	> 90%	ICAO Annex 3, 11ª ed.	Apêndice 3, §4.4.2.3 b)	FOG declarado quando visibilidade < 1.000 m, associado a umidade ≥ 90%
 	Chuva leve	> 0 mm	ICAO Annex 3, 11ª ed.	Apêndice 3, §2.3.2 d)	Qualquer precipitação (onset, cessation ou mudança de intensidade) exige SPECI
Camada Segurança L1	Vento	> 10 m/s	TP-2001-217 (NLR — van Es et al., 2001)	p. 18 — JAR/FAR 25.237	Mínimo de certificação para vento cruzado: 20 kt
Camada Segurança L1	Vento	> 10 m/s	TP-2001-217 (NLR — van Es et al., 2001)	p. 3 e p. 25	ICAO limite recomendado: 15 kt incluindo rajadas
Camada Segurança L1	Rajada	> 12,5 m/s	ICAO Annex 3, 11ª ed.	Apêndice 3, §4.1.5.2 c)2)	Threshold de rajada significativa: 5 m/s acima de 7,5 m/s crítico
Camada Segurança L1	Precipitação Moderada	> 2,5 mm/h	Glossário ANAC (DECEA/REDEMET)	Definição "Chuva Moderada"	"Moderada apresenta taxa de queda entre 2,5 mm e 7,5 mm por hora" — impacto em visibilidade e tração
Camada Segurança L1	Umidade	> 90%	ICAO Annex 3, 11ª ed.	Apêndice 3, §4.4.2.3 b)	Limiar de formação de nevoeiro em superfície
Camada Segurança L1	Gelo	T<2 e (U> 90 e P > 0)	L1: zona de icing (Brandão)		
Camada Segurança L1	Nevoeiro	U>90 e V<2.5	ICAO Annex 3 §4.4.2.3 b): ≤ 5 kt + RH alta		
Camada Segurança L1	Temperatura	< 2°C	ICAO Annex 3, 11ª ed.	Apêndice 3, §2.3.1 c)	Variação ≥ 2°C dispara SPECI; 2°C como buffer pré-congelamento
Camada Segurança L2	Vento	> 17 m/s	ICAO Annex 3, 11ª ed.	Apêndice 2, §5.1.1 e Apêndice 6, §5.1.3	Alerta de ciclone tropical ≥ 34 kt (17 m/s) — inclusão obrigatória
Camada Segurança L2	Rajada	> 25 m/s	TP-2001-217 (NLR — van Es et al., 2001)	p. 1	Velocidade máxima em microexplosão ~100 kt; 25 m/s é limiar conservador operacional
Camada Segurança L2	Precipitação Pesada	> 7,5 mm/h	Glossário ANAC (DECEA/REDEMET)	Definição "Chuva Forte"	"Forte apresenta taxa de queda superior a 7,5 mm por hora" — visibilidade severamente reduzida e risco de aquaplanagem
Camada Segurança L2	Umidade	> 95%	ICAO Annex 3, 11ª ed.	Apêndice 3, §4.3.2 e §2.501	RVR obrigatório abaixo de 1.500 m; visibilidade operacionalmente crítica
Camada Segurança L2	Temperatura	< 0°C	ICAO Annex 3, 11ª ed.	Apêndice 3, §4.6.2.2	Temperatura negativa deve ser identificada — condição de gelo obrigatória no METAR
Camada Segurança L2	Gelo	T<0 e (U> 90 e P > 0)	 L2: gelo confirmado (ICAO §4.6.2.2)		
Camada Segurança L2	Nevoeiro	U>95 e V<1.0	ICAO Annex 3 §4.3.2: calma + saturação		
Camada Segurança L2	Temperatura	< 0°C	RBAC 121 Emenda 07	Art. 121.629(b)	Proibida decolagem com geada, neve ou gelo aderido em superfícies críticas
	Precipitação Leve	> 0,1mm/h e < 2,5 mm/h	Glossário ANAC (DECEA/REDEMET) / ICAO Annex 3, 11ª ed.	Definição "Chuva Leve"	"Leve apresenta taxa de queda inferior a 2,5 mm por hora"
	Temperatura mín.	> 2°C	ICAO Annex 3, 11ª ed.	Apêndice 3, §2.3.1 c)	Acima do buffer pré-congelamento
	Temperatura máx.	< 38°C	RBAC 121 Emenda 07	Art. 121.175(b)	Performance degradada por altitude density acima de 38°C

 
## 10. Citação Indireta — Texto para o Artigo do TCC

### Resultados Quantitativos

O sistema foi avaliado sobre um conjunto de 18.171 registros históricos do CENIPA, particionado em 80% para treino (14.536 amostras) e 20% para teste (3.635 amostras). O desequilíbrio de classes, em particular a classe "Incidente Grave", com apenas 6,8% do conjunto de treino, foi tratado mediante a aplicação de SMOTE (*Synthetic Minority Over-sampling Technique*, Chawla et al., 2002) sobre o conjunto de treino, elevando todas as classes para 7.304 amostras cada.

Quatro algoritmos foram avaliados na fase de seleção: Random Forest (F1 Macro = 0,7553; ROC AUC = 0,9602), SVM com kernel RBF (F1 Macro = 0,7596; ROC AUC = 0,9525), Rede Neural MLP (F1 Macro = 0,7327; ROC AUC = 0,9332) e XGBoost (F1 Macro = 0,7539; ROC AUC = 0,9608). O modelo final, Stacking Ensemble com base learners RF + XGBoost + SVM + MLP e meta-learner de Regressão Logística treinado com out-of-fold predictions em 5 folds — obteve Acurácia = 0,879, F1 Macro = 0,744 e ROC AUC = 0,950 sobre o conjunto de teste. O critério de seleção do modelo final foi o maior F1-Score Macro (Sokolova & Lapalme, 2009), por sua propriedade de atribuir peso igual a todas as classes independentemente de sua frequência.

A análise por classe revela assimetria de desempenho consistente com a literatura: a classe "Voo Normal" atingiu F1 = 1,00 em todos os modelos; a classe "Incidente" alcançou F1 = 0,93; a classe "Acidente" obteve F1 = 0,74; e a classe "Incidente Grave" — a menor em volume (248 amostras no conjunto de teste) — obteve F1 = 0,31. Esse padrão é descrito por Harms (2020) como característico de modelos de ML treinados sobre dados aeronáuticos: eventos raros e limítrofes tendem a apresentar desempenho substancialmente inferior aos eventos mais frequentes, dada a escassez de padrões representativos no conjunto de treinamento.

### Contribuições e Discussão

Os resultados obtidos são comparáveis aos de estudos similares na literatura: Harms (2020) reporta F1 scores entre 0,70 e 0,80 para predição binária de safety events em dados aviação, enquanto Walden et al. (2023) relatam acurácias entre 0,83 e 0,91 para classificação de risco pré-voo com FRATs baseados em ML. O presente trabalho diferencia-se por três aspectos: a extensão para quatro classes de gravidade alinhadas com a taxonomia CENIPA; a integração de features meteorológicas no vetor de entrada, cujas importâncias (vento: 0,037; umidade: 0,029; rajada: 0,024; pressão: 0,027 no Random Forest) validam empiricamente a relevância meteorológica para predição de gravidade; e a presença de uma camada de validação de segurança baseada em regras operacionais que aplica limiares normativos de forma determinística sobre a saída do classificador.

A camada de validação de segurança opera de forma independente do modelo estatístico, garantindo que limitações estruturais do classificador — em particular o baixo desempenho na classe "Incidente Grave" e o viés de amostragem decorrente de um dataset composto exclusivamente por ocorrências investigadas — não se propaguem para a avaliação final apresentada ao operador. Esse princípio é consistente com a arquitetura de Sistemas de Gerenciamento de Segurança (SMS) promovida pelo ICAO, que preconiza múltiplos mecanismos de mitigação de risco independentes e complementares (Ziakkas & Pechlivanis, 2024; Kirwan, 2024). O resultado é um sistema que combina o poder preditivo de um ensemble de ML com a garantia de conformidade regulatória determinística — atendendo ao posicionamento consolidado na literatura de que sistemas algorítmicos em aviação devem atuar como ferramentas de suporte à decisão dentro de um framework normativo, e não como árbitros únicos de segurança operacional (Kirwan, 2024, p. 365).

---

## 11. Referências Bibliográficas (ABNT)

BREIMAN, Leo. Stacked regressions. **Machine Learning**, v. 24, n. 1, p. 49–64, 1996. DOI: 10.1007/BF00117832.

CHAWLA, Nitesh V.; BOWYER, Kevin W.; HALL, Lawrence O.; KEGELMEYER, W. Philip. SMOTE: Synthetic Minority Over-sampling Technique. **Journal of Artificial Intelligence Research**, v. 16, p. 321–357, 2002. DOI: 10.1613/jair.953.

HARMS, Tyler. **A Machine Learning Approach to Flight Safety Event Prediction**. 2020. Dissertação (Mestrado) — Iowa State University, Ames, Iowa, 2020.

KIRWAN, Barry. The Impact of Artificial Intelligence on Future Aviation Safety Culture. **Future Transportation**, v. 4, n. 2, p. 349–379, 2024. DOI: 10.3390/futuretransp4020018.

LIU, Wei et al. Flight Safety Risk Assessment Using Machine Learning: A Comprehensive Review. *[Dados do artigo a verificar — inserir referência completa conforme cópia disponível]*. 2024.

SOKOLOVA, Marina; LAPALME, Guy. A systematic analysis of performance measures for classification tasks. **Information Processing & Management**, v. 45, n. 4, p. 427–437, 2009. DOI: 10.1016/j.ipm.2009.03.002.

WALDEN, Christopher T.; RUSNOCK, Christina F.; BOUBIN, Jeremiah G. Proposal for a Modernized Flight Risk Assessment Tool Using Machine Learning. **Journal of Aviation/Aerospace Education & Research**, v. 32, n. 1, 2023. DOI: 10.15394/jaaer.2023.1914.

WOLPERT, David H. Stacked generalization. **Neural Networks**, v. 5, n. 2, p. 241–259, 1992. DOI: 10.1016/S0893-6080(05)80023-1.

ZIAKKAS, Dimitrios; PECHLIVANIS, Konstantinos. The Role of Artificial Intelligence in Transportation Safety Management Systems: The Aviation Safety II Case Study. **Aerospace**, v. 11, n. 2, p. 132, 2024. DOI: 10.3390/aerospace11020132.

