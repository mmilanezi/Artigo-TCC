# Sistema Inteligente de Apoio à Decisão Pré-Voo

**Trabalho de Conclusão de Curso — Ciência da Computação | UNISINOS**  
**Autor:** Matheus Milanezi  
**Status:** Beta 4.0
**Version Documentation** 1.6
**Version Model ML** 1.6

---

## Sumário

1. [Visão Geral](#visão-geral)
2. [Recursos do Sistema](#recursos-do-sistema)
3. [Arquitetura Geral](#arquitetura-geral)
4. [Fontes de Dados](#fontes-de-dados)
5. [Pipeline de Desenvolvimento](#pipeline-de-desenvolvimento)
   - [Fase 1 — Ingestão de Dados](#fase-1--ingestão-de-dados)
   - [Fase 2 — Integração e Enriquecimento](#fase-2--integração-e-enriquecimento)
   - [Fase 2b — Geração de Dados Sintéticos](#fase-2b--geração-de-dados-sintéticos)
   - [Fase 3 — Engenharia de Features](#fase-3--engenharia-de-features)
   - [Fase 4 — Treinamento dos Modelos Base](#fase-4--treinamento-dos-modelos-base)
   - [Fase 5 — Ensemble por Stacking](#fase-5--ensemble-por-stacking)
   - [Fase 6 — Interface Web e Inferência](#fase-6--interface-web-e-inferência)
6. [Modelo de Machine Learning](#modelo-de-machine-learning)
   - [Arquitetura do Ensemble](#arquitetura-do-ensemble)
   - [Features Utilizadas](#features-utilizadas)
   - [Classes de Saída](#classes-de-saída)
   - [Lógica de Status e Overrides de Segurança](#lógica-de-status-e-overrides-de-segurança)
   - [Desempenho](#desempenho)
7. [Camadas do Sistema](#camadas-do-sistema)
   - [Camada de Dados](#camada-de-dados)
   - [Camada de Processamento](#camada-de-processamento)
   - [Camada de Inferência](#camada-de-inferência)
   - [Camada de Apresentação](#camada-de-apresentação)
8. [Interface Web](#interface-web)
9. [Relatório em PDF](#relatório-em-pdf)
10. [Integração com APIs Externas](#integração-com-apis-externas)
11. [Estrutura de Arquivos](#estrutura-de-arquivos)
12. [Bibliotecas Utilizadas](#bibliotecas-utilizadas)

---

## Visão Geral

O **Sistema Inteligente de Apoio à Decisão Pré-Voo** é uma aplicação web desenvolvida para auxiliar pilotos e operadores da aviação civil brasileira na avaliação de segurança de voos antes da decolagem. O sistema combina três fontes de informação para gerar uma análise preditiva:

- **Histórico de ocorrências** da aviação civil brasileira (CENIPA — 2006 a 2023)
- **Dados meteorológicos** reais e previstos (INMET histórico + API Open-Meteo)
- **Características da aeronave** e do itinerário planejado

A partir dessas entradas, um modelo de Machine Learning baseado em **Stacking Ensemble** classifica o risco de cada ponto do voo (decolagem, cruzeiro, escalas e pouso) em quatro categorias, emitindo um parecer final com recomendações e, opcionalmente, um relatório em PDF.

---

## Recursos do Sistema

### Análise de Voo
- Avaliação de risco por **ponto do itinerário** (decolagens, cruzeiro, escalas intermediárias, pousos)
- Suporte a **múltiplas escalas** com análise independente por trecho
- **Quatro níveis de classificação** de risco: Apto Seguro, Apto Moderado, Não Apto, Inconclusivo
- **Overrides de segurança meteorológica**: condições extremas forçam "Não Apto" independentemente do modelo

### Dados Meteorológicos
- Consulta automática ao banco de dados histórico do **INMET** (2000–fevereiro 2026) para datas passadas
- Consulta à **API Open-Meteo** para previsão de até **15 dias no futuro**
- Normalização de nomes de estações (remoção de acentos, mapeamento por estado)

### Busca de Aeródromos
- Base de dados de aeroportos do Brasil via **OurAirports**
- Busca por cidade e estado (UF)
- Retorna código ICAO, IATA, nome e coordenadas geográficas

### Interface Web
- Interface de usuário moderna em página única (SPA)
- Formulário com validação em tempo real
- Consulta dinâmica de aeródromos
- Adição e remoção de escalas intermediárias
- Exibição de resultado com status visual colorido e recomendações

### Relatório em PDF
- Geração de laudo técnico completo em PDF via **ReportLab**
- Seções: cabeçalho institucional, status geral, dados da aeronave e itinerário, análise por ponto, recomendações, diagnóstico do modelo
- Código visual de cores por status (verde, laranja, vermelho, cinza)

### Modos de Execução
- **Interface Web** (modo principal): acesso via navegador na porta 7550
- **Modo demo**: 4 cenários pré-definidos para demonstração e validação
- **CLI interativo**: entrada manual via terminal (`sistema_prevoo.py`)

---

## Arquitetura Geral

```
┌─────────────────────────────────────────────────────────────────┐
│                        USUÁRIO (Navegador)                      │
└────────────────────────────┬────────────────────────────────────┘
                             │ HTTP
┌────────────────────────────▼────────────────────────────────────┐
│                    Interface Web (Flask)                        │
│  /             → index.html (SPA)                               │
│  /api/analisar → análise do itinerário                          │
│  /api/aerodromos → busca de aeródromos                          │
│  /api/relatorio/pdf → geração de PDF                            │
│  /registros    → página de histórico de análises (Data Flywheel)│
│  /api/registros → lista análises com paginação                  │
│  /api/registros/estatisticas → estatísticas das análises        │
│  /health       → verificação de saúde                           │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼───────────────────────────────────┐
│                   Camada de Inferência                         │
│  src/inferencia.py  → carrega modelo e pipeline                │
│  src/relatorio.py   → lógica de status + overrides             │
│  src/features.py    → construção do vetor de features          │
└───────────┬───────────────────────────────┬────────────────────┘
            │                               │
┌───────────▼───────────┐   ┌───────────────▼──────────────────┐
│  Modelo Stacking      │   │       Dados Meteorológicos       │
│  modelos/             │   │  src/meteo.py                    │
│  stacking_ensemble    │   │  INMET (histórico) ou            │
│  .joblib              │   │  Open-Meteo API (previsão)       │
└───────────────────────┘   └──────────────────────────────────┘
            │
┌───────────▼──────────────────────────────────────────────────┐
│               Pipeline de Pré-processamento                  │
│  dados/dados_processados/pipeline_preprocessamento.joblib    │
│  StandardScaler + OneHotEncoder                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Fontes de Dados

| Fonte | Descrição | Uso |
|-------|-----------|-----|
| **CENIPA** | Centro de Investigação e Prevenção de Acidentes Aeronáuticos — ocorrências da aviação civil brasileira (2006–2023) | Labels de treinamento (acidente, incidente, incidente grave) |
| **INMET** | Instituto Nacional de Meteorologia — arquivos CSV históricos por ano/estado (2000–fev/2026) | Features meteorológicas em inferência e treinamento |
| **Open-Meteo** | API gratuita de previsão meteorológica (até 15 dias) | Features meteorológicas para datas futuras |
| **OurAirports** | Base de dados pública de aeroportos mundiais | Lookup de aeródromos brasileiros por cidade/estado |

---

## Pipeline de Desenvolvimento

### Fase 1 — Ingestão de Dados

**Objetivo:** carregar as fontes de dados brutas e normalizá-las.

**Scripts:**
- `pipeline/fase1_ler_cenipa.py` — lê `ocorrencia.csv` do CENIPA; normaliza colunas (minúsculo, sem acentos)
- `pipeline/fase1_ler_inmet.py` — lê os arquivos CSV do INMET organizados por ano/estado; cria índice por UF e período

**Saída:** DataFrames de ocorrências e dados meteorológicos prontos para integração.

---

### Fase 2 — Integração e Enriquecimento

**Objetivo:** cruzar ocorrências com dados de aeronaves, tipos de ocorrência e enriquecer os labels com base em severidade.

**Scripts:**
- `pipeline/fase2_gerar_aeronaves.py` — extrai e normaliza perfis de aeronaves das ocorrências
- `pipeline/fase2_gerar_rotas.py` — extrai dados de rotas (UF origem/destino, fase do voo)
- `pipeline/fase2_integrar_dados.py` — join CENIPA + aeronaves + tipos de ocorrência; define labels finais com base em nível de dano e número de fatalidades
- `pipeline/fase2_validar_integracao.py` — verifica integridade do dataset integrado

**Saída:** `dados/dados_integrados/dataset_integrado.csv`

**Regra de rotulagem:**
- Dano destruído + fatalidades → Classe 3 (Acidente)
- Dano substancial sem fatalidades → Classe 2 (Incidente Grave)
- Dano leve ou nenhum → Classe 1 (Incidente)

---

### Fase 2b — Geração de Dados Sintéticos

**Objetivo:** corrigir o viés do CENIPA (que registra apenas ocorrências, sem voos normais) adicionando a **Classe 0 (Voo Normal)**.

**Script:** `pipeline/fase2b_voos_normais.py`

**Abordagem:**
- Geração de **5.000 amostras sintéticas** de voos normais com perfis realistas de aeronaves
- Distribuições meteorológicas dentro de faixas seguras (baixo vento, sem precipitação, temperatura e umidade normais)
- Tipos de aeronave, operação e fases de voo com distribuição representativa da aviação civil brasileira

**Por que foi necessário:** sem a Classe 0, o modelo aprenderia que qualquer voo é um risco. A geração sintética permite ao modelo distinguir o que é seguro do que é perigoso.

---

### Fase 3 — Engenharia de Features

**Objetivo:** transformar o dataset integrado em um vetor de features numéricas adequado ao treinamento.

**Script:** `pipeline/fase3_processar_dados.py`

**Etapas:**
1. Leitura do dataset integrado + dados sintéticos da Fase 2b
2. Construção do vetor de features:
   - **13 features numéricas** → `StandardScaler`
   - **12 features binárias** → sem transformação
   - **5 features categóricas** → `OneHotEncoder` (gera ~164 colunas)
3. **SMOTE** (Synthetic Minority Over-sampling Technique) para balanceamento de classes
4. Divisão em treino/teste (80%/20%)

**Saída:**
- `dados/dados_processados/X_treino.csv`, `X_teste.csv`
- `dados/dados_processados/y_treino.csv`, `y_teste.csv`
- `dados/dados_processados/pipeline_preprocessamento.joblib`
- `dados/dados_processados/meta_features.json`

---

### Fase 4 — Treinamento dos Modelos Base

**Objetivo:** treinar e avaliar quatro modelos independentes, selecionando o melhor como modelo base.

**Script:** `pipeline/fase4_treinar_modelo.py`

**Modelos treinados:**

| Modelo | Configuração Principal |
|--------|----------------------|
| **Random Forest** | 300 árvores, `class_weight=balanced`, `min_samples_leaf=2` |
| **XGBoost** | 300 estimadores, `max_depth=6`, `learning_rate=0.1`, `subsample=0.8` |
| **SVM** | Kernel RBF, `C=1.2`, `class_weight=balanced` |
| **MLP (Rede Neural)** | 2 camadas ocultas [128, 64], ativação ReLU, solver Adam, early stopping |

<!-- **Critério de seleção:** Score composto = `60% × Acurácia + 40% × F1 Macro` REMOVIDO DOS CRITÉRIOS - VERSÃO 1.2 -->

**Saída:**
- `modelos/*.joblib` (todos os modelos serializados)
- `modelos/relatorio_fase4.txt`

---

### Fase 5 — Ensemble por Stacking

**Objetivo:** combinar os quatro modelos base em um meta-modelo que aprende a melhor forma de combiná-los.

**Script:** `pipeline/fase5_stacking.py`

**Abordagem:**
- **Nível 1 (Base Learners):** RF + XGBoost + SVM + MLP treinados no conjunto completo
- **Out-of-fold predictions:** as previsões dos modelos base são geradas com **validação cruzada de 5 folds** para evitar overfitting no meta-modelo
- **Nível 2 (Meta-Learner):** Regressão Logística multinomial (regularização `C=1.0`) treinada sobre as predições OOF

**Saída:**
- `modelos/stacking_ensemble.joblib`
- `modelos/relatorio_fase5.txt`

---

### Fase 6 — Interface Web e Inferência

**Objetivo:** disponibilizar o sistema via interface web com análise por itinerário, consulta meteorológica em tempo real e geração de relatório em PDF.

**Componentes desenvolvidos:**
- `web/app.py` — servidor Flask com endpoints da API
- `web/templates/index.html` — interface web (SPA)
- `web/static/css/style.css` — folha de estilos
- `web/static/js/main.js` — lógica frontend (679 linhas)
- `src/inferencia.py` — pipeline de inferência
- `src/relatorio.py` — lógica de status e overrides
- `src/pdf_relatorio.py` — geração do PDF em ReportLab
- `src/meteo.py` — integração meteorológica dual (INMET + Open-Meteo)
- `src/aerodromos.py` — base de aeródromos OurAirports
- `src/demo.py` — cenários de demonstração

---

## Modelo de Machine Learning

### Arquitetura do Ensemble

```
Entrada: Itinerário do Voo
    ↓
[Vetor de Features — 189 dimensões]
    ↓
[Pipeline de Pré-processamento]
  StandardScaler (numéricas) + OneHotEncoder (categóricas)
    ↓
┌─────────────────────────────────────────────────────────┐
│                   Nível 1 — Base Learners               │
│                                                         │
│  ┌─────────────┐  ┌──────────┐  ┌─────┐  ┌─────────┐    │
│  │Random Forest│  │ XGBoost  │  │ SVM │  │   MLP   │    │
│  │ 300 árvores │  │ 300 est. │  │ RBF │  │ 2 camada│    │
│  └──────┬──────┘  └────┬─────┘  └──┬──┘  └────┬────┘    │
│         │              │           │          │         │
│  Out-of-Fold Predictions (5-fold Cross-Validation)      │
└─────────┼──────────────┼────────────┼───────────┼───────┘
          └──────────────┴────────────┴───────────┘
                         │
            [Vetor de meta-features (4 modelos × 4 classes = 16)]
                         ↓
┌────────────────────────────────────────────────────────┐
│              Nível 2 — Meta-Learner                    │
│         Regressão Logística Multinomial (C=1.0)        │
└────────────────────────┬───────────────────────────────┘
                         ↓
          Classe Predita + Probabilidades por Classe
```

---

### Features Utilizadas

**Total: 189 features após pré-processamento**

#### Features Numéricas (13) — StandardScaler
| Feature | Descrição |
|---------|-----------|
| `pave_num_motores` | Quantidade de motores da aeronave |
| `pave_pmd_kg` | Peso máximo de decolagem (kg) |
| `pave_idade_aeronave` | Idade da aeronave (anos) |
| `pave_aeronave_assentos` | Número de assentos |
| `pave_vento_ms` | Velocidade do vento (m/s) |
| `pave_rajada_ms` | Velocidade da rajada (m/s) |
| `pave_precipitacao_mm` | Precipitação (mm) |
| `pave_umidade_pct` | Umidade relativa (%) |
| `pave_temp_c` | Temperatura (°C) |
| `pave_pressao_mb` | Pressão atmosférica (mb) |
| `pave_hora_voo` | Hora do voo (0–23) |
| `pave_mes` | Mês do voo (1–12) |
| `pave_n_tipos` | Número de tipos de ocorrência associados |

#### Features Binárias (12) — Sem transformação
| Feature | Descrição | Limiar | Fonte |
|---------|-----------|--------|-------|
| `pave_motor_pistao` | Aeronave com motor à pistão (1/0) | — | — |
| `pave_motor_jato` | Aeronave com motor a jato (1/0) | — | — |
| `pave_vento_forte` | Vento acima do limiar de windshear crítico (1/0) | > 7,5 m/s (15 kt) | ICAO Annex 3 §7.4.3 / RBAC 121.429(a) |
| `pave_rajada_forte` | Rajada significativa — gust ≥ 5 m/s acima de 7,5 m/s (1/0) | > 12,5 m/s | ICAO Annex 3 §4.1.5.2 c)2) |
| `pave_chuva` | Qualquer precipitação presente (1/0) | > 0 mm | — |
| `pave_umidade_alta` | Umidade próxima ao limiar de nevoeiro (1/0) | > 90% | ICAO Annex 3 §4.4.2.3 b) |
| `pave_voo_noturno` | Horário noturno (1/0) | 18h–6h | — |
| `pave_fase_critica` | Fase crítica do voo: decolagem, subida, aproximação final, pouso (1/0) | — | — |
| `pave_fase_solo` | Fase em solo: táxi, estacionamento (1/0) | — | — |
| `pave_instrucao` | Voo de instrução (1/0) | — | — |
| `pave_operacao_regular` | Operação aérea regular (1/0) | — | — |
| `pave_tipo_grave` | Tipo de ocorrência historicamente grave no CENIPA (1/0) | — | — |

#### Features Categóricas (5) — OneHotEncoder (~164 colunas)
| Feature | Categorias | Colunas Geradas |
|---------|-----------|----------------|
| `aeronave_motor_tipo` | Pistão, Jato, Turbofan, Turboélice, Elétrico | ~5 |
| `aeronave_tipo_operacao` | Regular, Instrução, Táxi aéreo, etc. | ~10 |
| `ocorrencia_tipo_principal` | +70 tipos do CENIPA | ~70 |
| `taxonomia_icao_principal` | ~20 taxonomias ICAO | ~20 |
| `uf` | 27 estados brasileiros | 27 |

---

### Classes de Saída

| Classe |      Label      | Descrição                                                         |
|--------|-----------------|-------------------------------------------------------------------|
| **0**  | Voo Normal      | Nenhuma ocorrência prevista — voo dentro dos padrões de segurança |
| **1**  | Incidente       | Ocorrência com dano leve; segurança preservada mas com desvio     |
| **2**  | Incidente Grave | Ocorrência com risco real para aeronave ou tripulantes            |
| **3**  | Acidente        | Ocorrência com dano substancial, destruição ou fatalidades        |

---

### Lógica de Status e Overrides de Segurança

O sistema converte as classes do modelo em **4 status operacionais**:

|     Status      |   Cor    |      Critério      |
|-----------------|----------|--------------------|
| `APTO_SEGURO`   | Verde    | Classe 0 com confiança ≥ 90% |
| `APTO_MODERADO` | Laranja  | Classe 1 (qualquer confiança) ou limítrofe |
| `NAO_APTO`      | Vermelho | Classe 2 ou 3      |
| `INCONCLUSIVO`  | Cinza    | Confiança máxima < 42% **ou** Classe 3 com ≥ 60% de probabilidade com meteorologia completamente segura (contradição detectada) **ou** dados meteorológicos reais indisponíveis (análise com medianas históricas) |

**Overrides de Segurança Meteorológica** — aplicados após a predição do modelo, somente quando dados meteorológicos reais estão disponíveis (INMET ou Open-Meteo). Organizados em dois níveis:

#### Nível L1 — Forças `APTO_MODERADO` (condições adversas)

| Condição | Limiar | Fonte Regulatória |
|----------|--------|-------------------|
| Vento | > 10 m/s (≈ 20 kt) | JAR/FAR 25.237 — mínimo de certificação para vento cruzado (TP-2001-217, p. 18) |
| Rajada | > 12,5 m/s | ICAO Annex 3 §4.1.5.2 c)2): gust ≥ 5 m/s acima do limiar crítico de 7,5 m/s |
| Precipitação | > 2,5 mm/h | ANAC Glossário: início de "Chuva Moderada" |
| Nevoeiro *(apenas fases de superfície)* | umidade ≥ 90% **e** vento < 2,5 m/s | WMO Hazardous Phenomena — Fog (2025) + ANAC Meteorologia Aeronáutica: ventos fracos (≤ 5 kt) com umidade próxima a 100% favorecem nevoeiro de radiação |
| Risco de gelo *(todas as fases)* | temp < 2°C **e** (umidade > 90% **ou** precipitação > 0) | Brandão (2016) — FGA Descomplicada: formação de gelo entre 0°C e +2°C com gotículas super-resfriadas; RBAC 121 Art. 121.629(b) |

#### Nível L2 — Força `NAO_APTO` (condições críticas)

| Condição | Limiar | Fonte Regulatória |
|----------|--------|-------------------|
| Vento | > 17 m/s (≈ 34 kt) | ICAO Annex 3 §5.1.1: limiar de alerta de ciclone tropical |
| Rajada | > 25 m/s | TP-2001-217, p. 1: limite superior de microexplosão (microburst) |
| Precipitação | > 7,5 mm/h | ANAC Glossário: início de "Chuva Forte" |
| Nevoeiro denso *(apenas fases de superfície)* | umidade ≥ 95% **e** vento < 1,0 m/s | WMO Hazardous Phenomena — Fog (2025) + ANAC: calma extrema com saturação favorece nevoeiro denso (vis < 1.000 m) |
| Gelo confirmado *(todas as fases)* | temp < 0°C **e** (umidade > 90% **ou** precipitação > 0) | ICAO Annex 3 §4.6.2.2: temperatura sub-zero como condição de gelo confirmada; RBAC 121 Art. 121.629(b); WMO Fog (2025): nevoeiro congelante deposita gelo de rime em aeronaves |

> **Nota sobre nevoeiro vs. gelo:** o nevoeiro é um fenômeno de superfície (ANAC: "ocorre junto à superfície") e por isso seus limiares só são verificados nas fases **crítica** (decolagem, subida, aproximação final, pouso) e **solo** (táxi, estacionamento). O risco de gelo, por sua vez, aplica-se a **todas as fases**, pois aeronaves em cruzeiro podem penetrar nuvens congelantes (FGA Descomplicada, Brandão, 2016).

> **Nota sobre `meteo_benigno`:** condição interna que determina se as condições meteorológicas são completamente benignas. Usada para detectar contradições: se o modelo prediz Acidente com ≥ 60% de confiança (`LIMIAR_CONTRADICAO = 0.60`) mas `meteo_benigno = True`, o sistema retorna `INCONCLUSIVO` em vez de `NAO_APTO`, sinalizando possível viés residual de amostragem geográfica do CENIPA. Adicionalmente, quando dados meteorológicos reais não estão disponíveis (`fonte_meteo = "padrao"`), o sistema também retorna `INCONCLUSIVO` para análises que seriam `APTO_SEGURO` ou `APTO_MODERADO`, uma vez que medianas históricas não refletem as condições reais do voo.

---

### Desempenho

|         Métrica          | Valor  |
|--------------------------|--------|
| **Acurácia**             | 87,54%  |
| **F1 Macro**             | 0,7428 |
| **F1 Weighted**          | 0,8753 |
| **ROC AUC**              | 0,95,07|
| **Tempo de Treinamento** | ~17 minutos (991 segundos) |
| **SMOTE Aplicado**       | Sim    |
| **Passthrough**          | Não (`passthrough=False`) |
| **Data de Treinamento**  | 19/04/2026 |

**Distribuição do conjunto de teste:**

|        Classe       | Amostras |
|---------------------|----------|
| 0 — Normal          |  1.000   |
| 1 — Incidente       |  1.827   |
| 2 — Incidente Grave |  248     |
| 3 — Acidente        |  560     |

---

## Camadas do Sistema

### Camada de Dados

Responsável pelo armazenamento e acesso a todas as fontes de informação.

|          Componente           |              Localização              |                 Descrição                     |
|-------------------------------|---------------------------------------|-----------------------------------------------|
| Dados brutos CENIPA           | `dados/dados_brutos/ocorrencias/`     | Ocorrências da aviação civil 2006–2023        |
| Dados brutos meteorológicos   | `dados/dados_brutos/meteo/`           | Arquivos INMET por ano/estado                 |
| Dataset integrado             | `dados/dados_integrados/`             | CENIPA + aeronaves + tipos (após Fase 2)      |
| Dataset processado            | `dados/dados_processados/`            | Features normalizadas, treino/teste separados |
| Pipeline de pré-processamento | `pipeline_preprocessamento.joblib`    | Scaler + Encoder serializados                 |
| Base de aeródromos            | `dados/aerodromos_brasil.json`        | Cache local do OurAirports                    |

### Camada de Processamento

Responsável pela engenharia de features e treinamento.

| Componente | Arquivo | Descrição |
|------------|---------|-----------|
| Construção de features | `src/features.py` | Monta o vetor de 189 features a partir da entrada |
| Processamento meteorológico | `src/meteo.py` | Busca INMET ou Open-Meteo conforme a data |
| Módulo de aeródromos | `src/aerodromos.py` | Lookup de aeroportos brasileiros |
| Wrapper XGBoost | `src/modelos.py` | Compatibilidade XGBClassifier com sklearn Pipeline |

### Camada de Inferência

Responsável pelo carregamento do modelo e geração da predição.

| Componente | Arquivo | Descrição |
|------------|---------|-----------|
| Pipeline de inferência | `src/inferencia.py` | Carrega ensemble + pipeline; executa predição |
| Lógica de relatório | `src/relatorio.py` | Determina status, aplica overrides, gera recomendações |
| Schema de dados | `src/schema.py` | Modelos Pydantic para validação de entrada (`ItinerarioVoo`, `DadosVoo`) |
| Configurações globais | `src/config.py` | Caminhos, limiares, mapeamentos, defaults meteorológicos |

### Camada de Apresentação

Responsável pela interface com o usuário.

| Componente | Arquivo | Descrição |
|------------|---------|-----------|
| Servidor Flask | `web/app.py` | API REST + serving do frontend |
| Interface web | `web/templates/index.html` | SPA com formulário e exibição de resultado |
| Estilos | `web/static/css/style.css` | Layout responsivo com código de cores por status |
| Lógica frontend | `web/static/js/main.js` | Validação, fetch API, renderização de resultado |
| Geração de PDF | `src/pdf_relatorio.py` | Laudo técnico em PDF via ReportLab |
| CLI interativo | `src/entrada.py` | Entrada manual via terminal |
| Demonstração | `src/demo.py` | 4 cenários pré-configurados para demo |
| Registro de análises | `src/registro.py` | Persistência em SQLite — Data Flywheel para retreino futuro |

---

## Interface Web

A interface é uma **Single Page Application (SPA)** acessível em `http://localhost:7550`.

### Formulário de Entrada

**Dados da Aeronave:**
- Matrícula, tipo de motor, número de motores, PMD (kg), ano de fabricação, assentos, tipo de operação

**Dados do Itinerário:**
- Cidade/UF de origem e destino com busca dinâmica de aeródromos
- Data e hora de partida, duração do voo
- Escalas intermediárias (adicionar/remover dinamicamente)

### Fluxo de Análise

```
Preenchimento → Validação → POST /api/analisar
    → Busca meteorológica automática
    → Construção do vetor de features por ponto
    → Inferência no modelo para cada ponto
    → Determinação do status geral
    → Exibição do resultado com recomendações
    → (Opcional) Download do PDF
```

### Histórico de Análises (Data Flywheel)

A interface web inclui uma página dedicada (`/registros`) que exibe o histórico de todas as análises realizadas, com filtro por status e paginação. Cada análise é armazenada em banco SQLite (`dados/registros.db`) com ID único, dados do itinerário, resultado e campos para rotulação humana futura — formando um ciclo de melhoria contínua do modelo.

### Restrições
- Data futura limitada a **15 dias** (limite da API Open-Meteo para previsão)
- Validação de campos obrigatórios no frontend antes do envio

---

## Relatório em PDF

O PDF é gerado via `POST /api/relatorio/pdf` e inclui as seguintes seções:

1. **Cabeçalho institucional** — título do sistema, data de emissão
2. **Status geral** — banner colorido com o resultado da análise
3. **Dados da aeronave** — matrícula, motor, PMD, assentos, tipo de operação
4. **Dados do itinerário** — origem, destino, data/hora, duração, escalas
5. **Análise por ponto** — tabela com status, meteorologia e tipo de ocorrência previsto para cada ponto (decolagem → cruzeiro → escalas → pouso)
6. **Recomendações** — lista de ações sugeridas conforme o status
7. **Seção diagnóstica** — probabilidades brutas do modelo, parâmetros utilizados

**Código de cores:**
- Verde `#16a34a` → APTO_SEGURO
- Laranja `#d97706` → APTO_MODERADO
- Vermelho `#dc2626` → NAO_APTO
- Cinza `#374151` → INCONCLUSIVO

---

## Integração com APIs Externas

### INMET (Instituto Nacional de Meteorologia)
- **Tipo:** Arquivos CSV locais (download prévio)
- **Cobertura:** 2000 a fevereiro de 2026
- **Uso:** Features meteorológicas para datas históricas e inferência em datas dentro desse período
- **Localização:** `dados/dados_brutos/meteo/{ano}/INMET_*.CSV`

### Open-Meteo
- **Tipo:** API REST gratuita (sem autenticação)
- **Cobertura:** previsão de até 15 dias a partir da data atual
- **Uso:** Features meteorológicas para datas futuras além do acervo INMET
- **Dados retornados:** temperatura, vento, rajada, precipitação, umidade, pressão

### OurAirports
- **Tipo:** CSV público (download único, cache local)
- **Uso:** Base de dados de aeródromos brasileiros para lookup por cidade/estado
- **Arquivo local:** `dados/aerodromos_brasil.json`
- **Dados:** nome, município, estado, ICAO, IATA, latitude/longitude

---

## Estrutura de Arquivos

```
Desenvolvimento/
├── pipeline/                          # Scripts do pipeline de dados
│   ├── fase1_ler_cenipa.py            # Leitura dos dados CENIPA
│   ├── fase1_ler_inmet.py             # Leitura dos dados INMET
│   ├── fase2_gerar_aeronaves.py       # Extração de perfis de aeronaves
│   ├── fase2_gerar_rotas.py           # Extração de rotas
│   ├── fase2_integrar_dados.py        # Join CENIPA + aeronaves + tipos
│   ├── fase2_validar_integracao.py    # Validação do dataset integrado
│   ├── fase2b_voos_normais.py         # Geração de 5.000 voos normais sintéticos
│   ├── fase3_processar_dados.py       # Feature engineering + SMOTE + split
│   ├── fase4_treinar_modelo.py        # Treinamento dos 4 modelos base
│   ├── fase4_analise_hiperparametros.py  # Análise de sensibilidade de hiperparâmetros
│   ├── fase5_stacking.py             # Treinamento do Stacking Ensemble
│   └── versionar_modelo.py            # Versionamento e ativação de modelos
│
├── src/                               # Módulos do sistema de inferência
│   ├── config.py                      # Constantes globais, caminhos, limiares
│   ├── schema.py                      # Pydantic: ItinerarioVoo, DadosVoo
│   ├── features.py                    # Construção do vetor de features
│   ├── modelos.py                     # XGBWrapper para compatibilidade sklearn
│   ├── meteo.py                       # Dados meteorológicos (INMET + Open-Meteo)
│   ├── aerodromos.py                  # Base de aeródromos (OurAirports)
│   ├── inferencia.py                  # Carregamento do modelo e inferência
│   ├── relatorio.py                   # Lógica de status, overrides, recomendações
│   ├── pdf_relatorio.py               # Geração de PDF (ReportLab)
│   ├── entrada.py                     # Coleta de dados via terminal
│   ├── demo.py                        # Cenários de demonstração
│   └── registro.py                    # Registro de análises em SQLite (Data Flywheel)
│
├── web/                               # Aplicação web Flask
│   ├── app.py                         # Rotas Flask e endpoints da API
│   ├── templates/index.html           # Interface web (SPA)
│   └── static/
│       ├── css/style.css              # Estilos
│       └── js/main.js                 # Lógica frontend (679 linhas)
│
├── modelos/                           # Modelos treinados e serializados
│   ├── stacking_ensemble.joblib       # Modelo final (produção — v1.6)
│   ├── melhor_modelo.joblib           # Melhor modelo individual (Fase 4)
│   ├── random_forest.joblib
│   ├── xgboost.joblib
│   ├── svm.joblib
│   ├── rede_neural_(mlp).joblib
│   ├── modelo_selecionado.json        # Metadados do modelo ativo
│   ├── relatorio_fase4.txt            # Relatório de avaliação da Fase 4
│   ├── relatorio_fase5.txt            # Relatório de avaliação da Fase 5
│   ├── hiperparametros/               # Resultados da análise de hiperparâmetros
│   │   ├── relatorio_hiperparametros.txt
│   │   └── resultados.json
│   └── historico/                     # Versões anteriores dos modelos
│       └── versoes.json
│
├── dados/
│   ├── aerodromos_brasil.json         # Cache de aeródromos brasileiros (OurAirports)
│   ├── registros.db                   # SQLite com histórico de análises (Data Flywheel)
│   ├── dados_brutos/                  # Dados originais CENIPA, INMET
│   ├── dados_integrados/              # Dataset integrado (Fase 2)
│   └── dados_processados/             # Features processadas (Fase 3)
│
├── planejamento/                      # Documentos de planejamento
│   └── desenvolvimento.md
│
├── sistema_prevoo.py                  # Ponto de entrada CLI
├── run_web.py                         # Launcher do servidor web
├── requirements.txt                   # Dependências Python
└── README.md                          # Documentação original
```

---

## Bibliotecas Utilizadas

### Machine Learning e Ciência de Dados

| Biblioteca | Versão | Uso |
|------------|--------|-----|
| **scikit-learn** | 1.8.0 | RandomForest, SVM, MLP, Regressão Logística, StackingClassifier, Pipeline, StandardScaler, OneHotEncoder, métricas |
| **xgboost** | 3.2.0 | XGBoostClassifier (modelo base de melhor desempenho individual) |
| **numpy** | 2.4.3 | Operações numéricas vetorizadas |
| **pandas** | 3.0.1 | Manipulação de DataFrames, leitura/escrita de CSV |
| **joblib** | 1.5.3 | Serialização e desserialização de modelos (`.joblib`) |
| **imbalanced-learn** | — | SMOTE para balanceamento de classes no treinamento |
| **scipy** | 1.17.1 | Funções matemáticas e estatísticas |
| **statsmodels** | 0.14.6 | Análises estatísticas auxiliares |

### Visualização

| Biblioteca | Versão | Uso |
|------------|--------|-----|
| **matplotlib** | 3.10.8 | Gráficos de avaliação do modelo |
| **seaborn** | 0.13.2 | Matriz de confusão e visualizações estatísticas |
| **plotly** | 6.6.0 | Visualizações interativas |

### Web e API

| Biblioteca | Versão | Uso |
|------------|--------|-----|
| **Flask** | — | Servidor web, roteamento, API REST |
| **Jinja2** | 3.1.6 | Renderização de templates HTML |
| **Werkzeug** | — | Utilitários WSGI |
| **requests** | 2.33.0 | Requisições HTTP (Open-Meteo, OurAirports) |
| **httpx** | 0.28.1 | Cliente HTTP assíncrono |
| **httpcore** | 1.0.9 | Core HTTP de baixo nível |

### Validação e Schema

| Biblioteca | Versão | Uso |
|------------|--------|-----|
| **pydantic** | — | Validação e tipagem de dados de entrada (`ItinerarioVoo`, `DadosVoo`) |

### Geração de PDF

| Biblioteca | Versão | Uso |
|------------|--------|-----|
| **ReportLab** | — | Geração de PDFs estruturados (Platypus, PageTemplate, Frame, Paragraph, Table) |

### Utilitários

| Biblioteca | Versão | Uso |
|------------|--------|-----|
| **arrow** | 1.4.0 | Manipulação avançada de datas e fusos horários |
| **python-dateutil** | 2.9.0 | Parsing e aritmética de datas |
| **colorama** | 0.4.6 | Cores no terminal (modo CLI) |
| **rich** | 14.3.3 | Tabelas e formatação rica no terminal |
| **unicodedata** | — | Normalização de strings (remoção de acentos) |
| **json** | — | Leitura/escrita de arquivos JSON |
| **csv** | — | Leitura de arquivos CSV |
| **yaml** | — | Leitura de arquivos de configuração |

### Desenvolvimento e Análise

| Biblioteca | Versão | Uso |
|------------|--------|-----|
| **Jupyter / JupyterLab** | — | Exploração de dados e prototipagem |
| **IPython** | — | Shell interativo para análise |

---

*TCC — Matheus Milanezi - UNISINOS, 2026.*
