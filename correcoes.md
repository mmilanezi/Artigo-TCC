Correções:
1) Nase sessão 2.4.4 - > A literatura recente revela que a IA tem desempenhado papéis diversos e complementares OK
na segurança aeronáutica, mas também evidencia limitações estruturais que ainda impedem a
formação de sistemas verdadeiramente integrados. Harms (2020) demonstra que técnicas supervisionadas são capazes de prever eventos de segurança com base em variáveis indiretas, mas
destaca que a eficácia dos modelos depende fortemente da disponibilidade de dados históricos,
especialmente porque eventos negativos são raros , desafio que o presente trabalho aborda por
meio da geração de dados sintéticos e do balanceamento via SMOTE.

A parte onde fala: "desafio que o presente trabalho aborda por
meio da geração de dados sintéticos e do balanceamento via SMOTE." Foi gerado dados sintéticos para voo seguro, voo normal. Isso está correto e coerente com o texto acima?

2) ocorrências 2006–2023 - Correto é 2007 - 2025  OK
Verifiar em todo o artigo e corrigir.

3)  O Quadro 3.1 OK
apresenta as variáveis numéricas e binárias do modelo.
[ht] Features numéricas e binárias do modelo preditivo
 O que é esse HT?

4) 3.2 Arquitetura do modelo ensemble OK
O modelo preditivo é um Stacking Ensemble composto por quatro algoritmos base , Random
Forest (100 árvores), XGBoost (300 estimadores, max_depth=6), SVM-RBF e MLP (uma
camada oculta, otimizador Adam) , combinados por Regressão Logística Multinomial como
meta-aprendiz. A seleção de quatro modelos de naturezas distintas é motivada pela diversidade
dos mecanismos de aprendizado: métodos de ensemble de árvores (RF e XGBoost), separação por hiperplano no espaço transformado (SVM-RBF) e representação hierárquica não-linear
(MLP) garantem que diferentes aspectos dos dados meteorológicos, operacionais e históricos
sejam capturados de forma complementar.

Erro de hiper parametro, os corretos estão a abixo:

HIPERPARAMETROS_STACKING = {
    "random_forest": {
        "n_estimators":    300,
        "max_depth":       None,
        "min_samples_leaf": 2,
        "class_weight":    "balanced",
    },
    "svm": {
        "kernel":        "rbf",
        "C":             1.2,
        "gamma":         "scale",
        "class_weight":  "balanced",
    },
    "mlp": {
        "hidden_layer_sizes": [128, 64],
        "activation":         "relu",
        "solver":             "adam",
        "max_iter":           300,
        "early_stopping":     True,
        "validation_fraction": 0.1,
    },
    "xgboost": {
        "n_estimators":    300,
        "max_depth":       6,
        "learning_rate":   0.1,
        "subsample":       0.8,
        "colsample_bytree": 0.8,
    },
    "meta_learner": {
        "solver":   "lbfgs",
        "C":        1.0,
        "max_iter": 1000,
    },
    "stacking": {
        "cv":          5,
        "passthrough": False,
    },
}

5) 3.3 Lógica de status e overrides de segurança
A saída do modelo , quatro classes com probabilidades associadas , é convertida em quatro
status operacionais apresentados ao usuário, conforme o Quadro 3.3.
[ht] Mapeamento entre classes do modelo e status operacionais

Está correto chamar de quadro 3.3?

6) Em adição ao modelo preditivo, o sistema incorpora uma camada de validação de segurança baseada em regras operacionais derivadas de normativos aeronáuticos internacionais, OK
aplicada sobre a saída do classificador antes da geração do relatório. 

Em um ponto do artigo é chamado de overrides e em outro de camada de segurança. Verifique se o entendimento está correto na relação de ambas, pois se trata da mesma coisa. Senti que no artigo está representado como coisas distintas.

7) Ajustar citações como: OK
 (ICAO Annex 3 §4.4.2.3 (International Civil Aviation Organization,

2018; World Meteorological Organization, 2025))

Segundo as normals da ABNT: 

1. Problemas Identificados
Mistura de dados: Você misturou a identificação de um parágrafo (normativo) com citações de autoria (bibliográficas) dentro do mesmo parênteses.

Excesso de elementos: A ABNT não recomenda repetir o nome completo da organização (ICAO) se você já utilizou a sigla, e a inclusão do parágrafo específico (§) junto com o ano e autores gera uma poluição visual que dificulta a leitura.

Sintaxe da ABNT: A norma de citação exige que, se for uma citação indireta, você coloque apenas o sobrenome do autor (ou nome da entidade) e o ano. Se for citação direta, você deve incluir a página ou o item/parágrafo.

2. Sugestões de Correção
Opção A: Focada na norma técnica (se você está citando o conteúdo do Anexo)
Se o seu objetivo é citar o conteúdo normativo, a ABNT prefere que a entidade seja citada primeiro:

Conforme estabelecido pela International Civil Aviation Organization (ICAO, 2018, § 4.4.2.3)...

Opção B: Citação de múltiplas fontes no texto
Se você está usando o Anexo 3 da ICAO (2018) e o documento da WMO (2025) para fundamentar um mesmo parágrafo, utilize ponto e vírgula para separar as fontes:

...conforme as diretrizes internacionais para meteorologia aeronáutica (ICAO, 2018; WMO, 2025).

Mantenha a citação limpa: (SIGLA, Ano).

O detalhamento ("Anexo 3, §4.4.2.3") deve aparecer preferencialmente no corpo do texto antes da citação ou em nota de rodapé, para não quebrar o fluxo da leitura.

8) O Quadro 3.3
sintetiza os limiares de cada nível.
[ht] Camada de validação de segurança: limiares operacionais por nível

Revisar as todas as tabelas e, colocar o número da tabela junto, exemplo:

Quadro 1: Nome da tabela
Tabela
Fonte: Elaborado pelo autor.

9) "que preconiza múltiplos mecanismos de mitigação de risco independentes e complementares"  OK
A palvra preconiza está estranha, poderia ver um sinônimo?

10) Verificar todas as virgulas e espaçamentos entre palavras, tem coisa errada. OK

11) 3.4 Interface web e relatório OK
O sistema é disponibilizado como uma Single Page Application (SPA), acessível via navegador na porta 7550. O formulário de entrada coleta dados da aeronave (matrícula, tipo de motor,
PMD, ano, assentos), do itinerário (cidade e UF de origem e destino, data, hora e duração do
voo) e permite adicionar escalas intermediárias. Ao submeter, o sistema consulta automaticamente os dados meteorológicos , INMET histórico para datas passadas e API Open-Meteo para
previsões de até 15 dias , constrói o vetor de features por ponto do itinerário, executa a inferência e exibe o resultado com código visual de cores e recomendações textuais. Opcionalmente, o
piloto pode gerar e baixar um relatório técnico em PDF, contendo status geral, análise por ponto
do itinerário, dados da aeronave e diagnóstico do modelo.

No texto acima eu notei alguns pontos, primeiro, não é relevante falar da porta que eu usei para testes interno. pode simplesmente remover essa informação e adaptar a frase para ficar coerente. Segundo, troque "PMD" pelo nome real, que é peso na decolagem, ou semelhante, para fácil entendimento. A parte que fala que o piloto pode gerar e baixar um relatório, não necessariamente pode ser o piloto mas qualquer pessoa, podemos alterar para outro nome, no lugar de piloto.

12) Ajuste de data: OK
Os dados meteorológicos históricos foram obtidos do INMET (Instituto Nacional de Meteorologia, 2024) (arquivos CSV por ano e estado, cobrindo o período 2000–2026), integrados
às ocorrências do CENIPA por data e estado de origem. Para operações com datas futuras (até
15 dias à frente), o sistema consulta a API Open-Meteo em tempo de inferência. A base de
aeródromos brasileiros foi obtida do OurAirports, permitindo o lookup por cidade e UF.

Falta a informação que fala que os dados históricos do CENIPA são até dia 31 de março de 2026. Dados históricos após essa data, são encontrados pela API open Mateo.

13) palavras como "oversampling" devem ser explicadas no rodapé. OK

14) O conjunto resultante, composto por 18.171 registros após integração, geração sintética e
limpeza, foi dividido em treino (80%, 14.536 amostras) e teste (20%, 3.635 amostras), sendo o
SMOTE aplicado exclusivamente ao conjunto de treino para evitar vazamento de dados. Após a
aplicação do SMOTE, todas as quatro classes foram elevadas para 7.304 amostras no conjunto
de treino, totalizando 29.216 amostras balanceadas para o treinamento dos modelos base.

A parte acima explica o dataset, certo? Está coerente com os dados abaixo? Como foi calculado o composto por 18.171 registros?

Split: 80/20 ordenado por classe
Limpeza: remove coordenadas fora do Brasil e dados nulos.
Features resultantes: 30 
Após OneHotEncoder: 189 colunas
SMOTE: aplicado apenas no treino para equalizar classes
Antes: 
Classe 0: 4.000 
Classe 1: 7.304
Classe 2: 994 
Classe 3: 2.238
Após SMOTE: 
Todas as classes ficaram com 7.304
Total de amostras: 29.216
Teste permanece inalterado (avaliação realista)

15) 4.3 Treinamento e validação cruzada: OK
Os quatro modelos base foram treinados de forma independente no conjunto de treino. As
predições utilizadas para treinar a Regressão Logística meta-aprendiz foram geradas via outof-fold predictions com validação cruzada de 5 partições, evitando que as predições do nível 0
fossem enviesadas pelo treino nos mesmos dados. Após a geração das meta-features, o metaaprendiz foi treinado sobre elas, e o ensemble completo foi serializado para uso em inferência.
O critério de seleção de modelos candidatos na Fase 4 do pipeline foi um score composto:
Score = 0,60 × Acurcia + 0,40 × F1M acro, privilegiando o equilíbrio entre desempenho
geral e representatividade das classes minoritárias.

o texto acima está errado. Primeiro, ficou confuso a sessão de como foi realizado as predições de validação. E a parte onde fala sobre o score composto, eu não estou utilizando mais essa métrica. Os modelos foram baseados apenas em avaliação manual considerando F1 macro e ROC AUC. Pode consultar o arquivo DOCUMENTACAO.md para melhor entendimento. E a frase "Fase 4", essa informação não é relevante, pois esse nome só é usado para uso interno.

16) Tabela 2 – Métricas de desempenho do Stacking Ensemble no conjunto de teste OK
Métrica Valor
Acurácia 87,54%
F1-Score Macro 0,7428
ROC-AUC 0,9507
Score Composto 0,8224
Tempo de treinamento ≈18 min (1.082 s)
Fonte: Elaborado pelo autor

Essa tabela está errada, score composto não é usado, e o tempo de treino foi de 991 segundos. O restante está correto.

17) Uma contribuição metodológica do trabalho é a comparação entre o Stacking Ensemble OK
e os modelos base individuais na fase de seleção: o score composto de 0,8224 do ensemble
superou todos os modelos treinados isoladamente, corroborando a fundamentação teórica de
Hastie, Tibshirani e Friedman (2009) de que a combinação de modelos não piora o desempenho
esperado e, em geral, melhora a generalização ao capturar perspectivas complementares sobre
os dados.

Não foi feita a comparação e a seleção de modelos entre os modelos, uma vez que usamos todos. Não faz sentido falar isso, concorda? E novamente apareceu o score composto, onde não é uma métrica usada. Reescreva esse paragrafo.

18) 
Os resultados são comparáveis aos de estudos similares: Harms (2020) reporta F1-scores
entre 0,70 e 0,80 para predição binária de eventos de segurança em dados de aviação, enquanto
Walden et al. (2023) relatam acurácias entre 0,83 e 0,91 para classificação de risco pré-voo com
FRATs baseados em ML. O presente trabalho diferencia-se por três aspectos: (i) a extensão
para quatro classes de gravidade alinhadas à taxonomia CENIPA; (ii) a integração de features
meteorológicas , cujas importâncias no Random Forest (vento: 0,037; umidade: 0,029; rajada:
0,024; pressão: 0,027) validam empiricamente a relevância meteorológica para predição de
gravidade; e (iii) a presença da camada de validação de segurança de dois níveis, baseada em
limiares operacionais normativos.

no trecho cima, a frase "ii) a integração de features
meteorológicas , cujas importâncias no Random Forest (vento: 0,037; umidade: 0,029; rajada:
0,024; pressão: 0,027) validam empiricamente a relevância meteorológica para predição de
gravidade;" onde foi retirado esses dados de vento, umidade, rajada, pressão? Por que foram trazidos 
para essa sessão?

19) Do ponto de vista prático, a integração do modelo preditivo com overrides determinísticos OK
de segurança meteorológica, consulta em tempo real ao INMET e Open-Meteo.

O trecho a cima fala da consulta em tempo real, ao open-mateo sim, mas é correto afirmar ao INMET? pois a base de consulta é a dados históricos que estão na base de dados da aplicação, e não a consultas ao INMET, mesmo que seja dados BAIXADOS do INMET.

20) para os trabalhos futuros, a aplicação contém o recurso onde é salvo em um banco de dados todas as análises que foram feitas, pelo modelo, para serem avaliadas posteriormente. Para trabalhos futuros, tem se a ideia de utulizar essas avaliações para melhorar o modelo, retreinando-o, aplicando a metodologia de self-training. OK

21) Colocar nas referências "acessado em", pois há referencias sem essa informação. Deixar o campo em branco para meu preenchimento.

22) Tem imagens em "images" que não foram usadas, como a imagem da arquitetura do modelo "artigo_v2/images/modelo ml.jpg" e a imagem da fase de processamento dos dados, "artigo_v2/images/Fase de dados.jpg" onde mostra a parte do fluxo de processamento dos dados brutos em datasets. As imagens  no anexo devem ser colocadas no relatório, na parte onde comenta sobre a parte web, por que são exemplos da saída do modelo. No anexo, eu vou por fotos dos relatórios em PDF, e interface do modelo. OK

23) Nesse trecho: Nas primeiras décadas da aviação comercial, especialmente entre os anos 1950 e 1970, a OK
maioria dos acidentes era atribuída a falhas técnicas, estruturais ou de projeto. Consequentemente, os esforços de segurança se concentravam quase que exclusivamente na melhoria do
desempenho mecânico, na padronização de procedimentos de manutenção, na certificação de
aeronaves e no fortalecimento da confiabilidade dos equipamentos. Com o amadurecimento
tecnológico e a redução das falhas mecânicas, o setor identificou o erro humano como o principal fator contribuinte para acidentes, deslocando a atenção para treinamento, ergonomia, fatores
cognitivos e cultura organizacional.


No trecho acima não está faltando referência?

---

## ANÁLISE DAS CORREÇÕES — RESULTADOS E DÚVIDAS

### Itens CORRIGIDOS no arquivo `artigo_v2/artigo_v2.tex`:

- **Item 1** ✅ Corrigido (confirmado pelo usuário): Seção 2.4.4 — frase reescrita para "eventos negativos são raros, desafio que o presente trabalho aborda por meio da introdução de uma classe de referência de voos normais (Classe 0) via geração sintética e do balanceamento das classes de eventos via SMOTE."
- **Item 2** ✅ Corrigido: "2006–2023" → "2007–2025" em todos os trechos. Também corrigido `\cite{CENIPA2024}` → `\cite{CENIPA2025}` (a chave CENIPA2024 não existia no .bib).
- **Item 4** ✅ Corrigido: RF agora correto com 300 árvores, pesos balanceados, profundidade irrestrita. MLP corrigido para duas camadas ocultas [128, 64], ativação ReLU, early stopping. Também corrigido na seção 2.2.2 (Fundamentação Teórica).
- **Item 6** ✅ Corrigido: Ao introduzir a camada pela primeira vez, o texto agora diz "camada de *overrides* de segurança, também denominada camada de validação de segurança". Terminologia unificada no Quadro e nas referências posteriores.
- **Item 7** ✅ Corrigido: Citações agora no formato ABNT: "(RBAC 121, Emenda 07, Art. 121.429(a)) \cite{RBAC121}" e "(ICAO, Anexo 3, §4.4.2.3) \cite{ICAO2018,WMO2025}". Detalhamento normativo movido para o corpo do texto, fora dos parênteses da citação.
- **Item 9** ✅ Corrigido: "preconiza" → "recomenda".
- **Item 10** ✅ Corrigido: Todos os padrões " , texto , " (travessão incorreto como vírgula com espaço) substituídos por ", texto,". Também corrigidos espaços soltos antes de vírgulas em vários pontos.
- **Item 11** ✅ Corrigido: Removida menção à porta 7550; "PMD" expandido para "Peso Máximo de Decolagem (PMD)"; "piloto" substituído por "usuário" na seção 3.4 e na conclusão.
- **Item 12** ✅ Corrigido: INMET agora especificado como cobrindo "de 2000 a 31 de março de 2026". Adicionada informação de que datas posteriores a essa data usam a API Open-Meteo.
- **Item 13** ✅ Corrigido: Adicionada nota de rodapé explicando *oversampling* (superamostragem).
- **Item 15** ✅ Corrigido: Removidos "score composto" e "Fase 4 do pipeline". Seção reescrita com descrição mais clara do processo out-of-fold e validação cruzada. Critério de avaliação agora descrito como F1-Score Macro e ROC-AUC.
- **Item 16** ✅ Corrigido: Linha "Score Composto" removida da tabela. Tempo de treinamento corrigido para ≈17 min (991 s).
- **Item 17** ✅ Corrigido: Parágrafo reescrito. Removidas afirmações sobre "comparação/seleção" entre modelos e sobre "score composto". Novo texto fundamenta o uso do ensemble na teoria de Hastie et al. e na diversidade dos algoritmos.
- **Item 19** ✅ Corrigido: Conclusão agora menciona "banco de dados histórico do INMET" e "API Open-Meteo em tempo real para datas futuras", diferenciando corretamente as duas fontes.
- **Item 20** ✅ Corrigido: Adicionado parágrafo nos trabalhos futuros sobre o Data Flywheel (banco SQLite de análises) e retreinamento via metodologia de *self-training*.
- **Item 3** ✅ Corrigido (confirmado pelo usuário): Removido `[ht]` de todos os `\begin{quadro}[ht]` (3 ocorrências). O ambiente `quadro` do template UNISINOS não é um float LaTeX padrão e não aceita argumento de posicionamento, por isso `[ht]` aparecia como texto no PDF.
- **Item 22** ✅ Corrigido: Adicionadas imagens `Modelo ML.JPG` (seção 3.2), `Fase de dados.JPG` (seção 4.2) e `Interface web.png` (seção 3.4). As quatro figuras de status operacional (Exemplo_apto_seguro, etc.) foram movidas do apêndice para a seção 3.4. O apêndice foi atualizado para receber as fotos dos relatórios PDF (a serem inseridas pelo autor). **Atenção**: os arquivos `Modelo ML.JPG` e `Fase de dados.JPG` têm espaços no nome. Foi adicionado o pacote LaTeX `grffile` para lidar com isso. Se houver erro de compilação, renomeie os arquivos para `ModeloML.JPG` e `FaseDados.JPG` e atualize os `\includegraphics` correspondentes.

---

### Itens que PRECISAM DE SUA CONFIRMAÇÃO (dúvidas):

**Item 1** ✅ **RESOLVIDO** — ver entrada no bloco "Itens CORRIGIDOS" acima.

~~**Item 1 — DÚVIDA original: Coerência do trecho sobre dados sintéticos + SMOTE**~~

Trecho: "eventos negativos são raros, desafio que o presente trabalho aborda por meio da geração de dados sintéticos e do balanceamento via SMOTE."

Análise: Os dados sintéticos foram gerados para a **Classe 0 (Voo Normal)** — não para as classes de eventos negativos. O SMOTE foi aplicado para balancear as classes de eventos (1, 2 e 3). Portanto, a "geração de dados sintéticos" corrige a ausência de voos normais no dataset, enquanto o SMOTE corrige o desequilíbrio entre as classes de eventos. O texto atual implica que ambos endereçam a raridade de eventos negativos, o que é tecnicamente impreciso. **Sugestão de reescrita**: "eventos negativos são raros, desafio que o presente trabalho aborda por meio da introdução de uma classe de referência de voos normais (Classe 0) via geração sintética e do balanceamento das classes de eventos via SMOTE." **Confirmar se deseja aplicar esta correção.**

Usuário: Confirmo a correção acima
---

**Item 3** ✅ **RESOLVIDO** — ver entrada no bloco "Itens CORRIGIDOS" acima.

~~**Item 3 — INFORMAÇÃO original: O que é "[ht]"**~~

O `[ht]` que aparece nos trechos LaTeX (ex: `\begin{quadro}[ht]`) é um **especificador de posicionamento de elementos flutuantes** do LaTeX: `h` = *here* (aqui, onde está no código), `t` = *top* (topo da página). Esse código **não aparece no PDF compilado** — é apenas uma instrução para o compilador sobre onde preferir colocar o quadro/tabela/figura. Nenhuma correção necessária.

Usuário: O meu questionamento é exatamente por conta de que está aparecendo no PDF compilado. Poderia verificar e ajustar para não aparecer no PDF compilado?
---

**Item 5 — INFORMAÇÃO: Numeração dos quadros**

A numeração "Quadro 3.1", "Quadro 3.2", "Quadro 3.3" é gerada **automaticamente pelo LaTeX** com base na ordem em que os quadros aparecem no documento e na seção em que estão. O valor correto será exibido no PDF compilado. Se a numeração estiver errada no PDF, verifique se algum `\begin{quadro}` foi inserido ou removido em uma ordem diferente.

---

**Item 8 — INFORMAÇÃO: Formatação de tabelas e quadros**

O formato de cabeçalho dos quadros/tabelas (ex: "Quadro 1 – Nome da tabela") é controlado pelo arquivo de classe `UNISINOSartigo.cls`. Os comandos `\caption{}` e `\fonte{}` já estão sendo usados corretamente. O formato exato (uso de ":" ou "–", posição do número) dependerá da configuração do template. **Verifique no PDF compilado** se o formato está de acordo com as normas da UNISINOS. Se necessário, ajuste o arquivo `UNISINOSartigo.cls`.

---

**Item 14 — VERIFICAÇÃO: Cálculo dos 18.171 registros**

O número 18.171 está **correto**. Demonstração:
- Conjunto de treino (80%): Classe 0=4.000 + Classe 1=7.304 + Classe 2=994 + Classe 3=2.238 = **14.536** ✓
- Conjunto de teste (20%): Classe 0=1.000 + Classe 1=1.827 + Classe 2=248 + Classe 3=560 = **3.635** ✓
- Total: 14.536 + 3.635 = **18.171** ✓
- Verificação: 18.171 × 0,80 = 14.536,8 ≈ 14.536 ✓ (arredondamento)

O texto está correto. Nenhuma alteração necessária.

---

**Item 18 — DÚVIDA: Origem dos valores de importância de features do Random Forest**

Trecho: "cujas importâncias no Random Forest (vento: 0,037; umidade: 0,029; rajada: 0,024; pressão: 0,027) validam empiricamente a relevância meteorológica"

**De onde foram extraídos esses valores?** Se foram obtidos durante o treinamento do modelo (via `feature_importances_` do scikit-learn), eles são válidos e podem permanecer no texto. Nesse caso, seria útil mencionar a fonte explicitamente (ex: "conforme calculado pelo modelo Random Forest treinado"). Se foram copiados de outro lugar ou são estimativas, devem ser removidos ou substituídos pelos valores reais do modelo treinado. **Confirmar a origem desses valores.**

---

**Item 21 — AÇÃO NECESSÁRIA: Referências sem "acessado em"**

Verificação do arquivo `.bib`: a maioria das referências com URL já possui `urlaccessdate`. A exceção é **WALDEN2023**, que não tem URL nem data de acesso (é um relatório de projeto de curso da George Mason University, possivelmente disponível online). **Ação necessária**: Se o relatório estiver disponível online, adicionar ao WALDEN2023 em `artigo_v2.bib`:
```
url = {URL_DO_DOCUMENTO},
urlaccessdate = {XX mmm. 20XX}
```
Se não estiver disponível online, a referência está adequada como `@misc` sem URL.

---

**Item 23** ✅ **Corrigido** (confirmado pelo usuário): Adicionado `\cite{HARMS2020,DEMIR2024}` ao final do parágrafo sobre 1950–1970, antes do ponto final da última frase ("...cultura organizacional \cite{HARMS2020,DEMIR2024}."). HARMS2020 discute a trajetória histórica da segurança aeronáutica e DEMIR2024 é a revisão sistemática com contexto histórico — ambos confirmados pelo usuário como mais adequados.

~~**Item 23 — DÚVIDA original:** Referência faltando no parágrafo sobre 1950–1970~~

O parágrafo sobre a evolução histórica da segurança aeronáutica (anos 1950–1970, erro humano, etc.) não possui citação. Sugestões de referências que cobrem essa evolução histórica e já estão no arquivo .bib:
- **HARMS2020** (Delft University): discute a trajetória histórica da segurança aeronáutica
- **DEMIR2024**: revisão sistemática que inclui contexto histórico do desenvolvimento da área
- **ZIAKKAS2024**: discute a transição Safety-I → Safety-II com contexto histórico

**Sugestão**: adicionar `\cite{HARMS2020}` ao final do parágrafo, ou citar `\citeonline{DEMIR2024}` no início. **Confirmar qual referência cobrir essa afirmação.**

Usuário: Acredito que Harms (2020) e Demir (2024) está mais adequado a utilização. Se você concordar, por favor revise a bibliografia e inclua sem alterar o texto, apenas ajustando onde se encaixa melhor a referência.


24) Revisar métricas de desempenho onde tem porcentagem e número de pontuação score, encontrei porcentagem onde era número de pontuação.

25) overfitting não está explicado, colocar no rodapé o que é o significado, conforme bibliografia existente. 

26) Quadro 3.3 não citado. Pode incluir no final do parágrafo algo como: "o quadro 3.3 logo abaixo, demosntra os limiares utilizados no sistema"