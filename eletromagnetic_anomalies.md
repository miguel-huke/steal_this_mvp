PROJETO: DETECÇÃO DE ANOMALIAS MAGNÉTICAS COM MACHINE LEARNING (END-TO-END)

OBJETIVO
Construir um sistema completo capaz de:
1. Coletar dados de campo magnético
2. Processar e limpar os dados
3. Treinar modelos de machine learning para identificar anomalias
4. Gerar mapas e visualizações
5. Disponibilizar via API e infraestrutura open source
6. Permitir aplicação prática (terra, indústria, exploração espacial)

--------------------------------------------------

FASE 1 — AQUISIÇÃO DE DADOS

FONTES:
- Dados públicos de satélites (NASA, ESA)
- Sensores físicos (magnetômetros)
- Simulações sintéticas (para bootstrap)

FORMATO PADRÃO:
- timestamp
- latitude
- longitude
- altitude (opcional)
- Bx, By, Bz (componentes do campo magnético)
- |B| (magnitude)

EXEMPLO:
t=1712500000, lat=-23.55, lon=-46.63, Bx=12.1, By=-3.2, Bz=41.8

REQUISITOS:
- frequência mínima: 1Hz (ideal: 10Hz+)
- precisão do sensor calibrada

--------------------------------------------------

FASE 2 — PRÉ-PROCESSAMENTO

ETAPAS:
1. Remoção de ruído
   - filtro Savitzky-Golay ou média móvel

2. Correção de drift
   - remover tendência temporal

3. Normalização
   - z-score ou min-max scaling

4. Interpolação
   - corrigir dados faltantes

5. Alinhamento espacial
   - transformar coordenadas em grid (ex: 10m x 10m)

SAÍDA:
dataset limpo e consistente

--------------------------------------------------

FASE 3 — ENGENHARIA DE FEATURES

FEATURES:
- magnitude |B|
- derivadas temporais dB/dt
- variância local
- energia espectral (FFT)
- diferença entre pontos vizinhos

OBJETIVO:
transformar dados brutos em representação útil para ML

--------------------------------------------------

FASE 4 — DEFINIÇÃO DE ANOMALIA

TIPOS:
- outlier estatístico
- mudança abrupta
- padrão espacial incomum

CRITÉRIO:
anomalia = desvio significativo do padrão aprendido

--------------------------------------------------

FASE 5 — MODELAGEM

BASELINE:
- Isolation Forest
- DBSCAN
- One-Class SVM

INTERMEDIÁRIO:
- Autoencoder (reconstrução)
- LSTM (séries temporais)

AVANÇADO:
- Transformers
- Graph Neural Networks
- Modelos multimodais

PIPELINE:
dados -> features -> modelo -> score de anomalia

OUTPUT:
score entre 0 e 1 (probabilidade de anomalia)

--------------------------------------------------

FASE 6 — TREINAMENTO

PASSOS:
1. separar dados normais
2. treinar modelo apenas com padrão normal
3. validar com dados mistos

MÉTRICAS:
- precision
- recall
- F1-score
- ROC-AUC

--------------------------------------------------

FASE 7 — INFERÊNCIA

INPUT:
novo vetor magnético

PROCESSO:
- aplicar mesmo pré-processamento
- extrair features
- passar no modelo

OUTPUT:
- score de anomalia
- classificação (normal/anômalo)

--------------------------------------------------

FASE 8 — VISUALIZAÇÃO

FERRAMENTAS:
- mapas 2D (heatmap)
- mapas 3D
- dashboards web

CAMADAS:
- intensidade magnética
- anomalias detectadas
- histórico temporal

TECNOLOGIAS:
- Python (matplotlib, plotly)
- QGIS
- WebGL / Three.js

--------------------------------------------------

FASE 9 — API

STACK:
- FastAPI (Python)

ENDPOINTS:

POST /predict
INPUT:
{
  "Bx": float,
  "By": float,
  "Bz": float,
  "lat": float,
  "lon": float
}

OUTPUT:
{
  "anomaly_score": float,
  "is_anomaly": boolean
}

GET /map
retorna mapa agregado

--------------------------------------------------

FASE 10 — INFRAESTRUTURA

ARQUITETURA:

[SENSOR]
   ↓
[EDGE DEVICE (Raspberry Pi)]
   ↓
[API]
   ↓
[BANCO DE DADOS]
   ↓
[DASHBOARD]

TECNOLOGIAS:
- Docker
- Redis (cache)
- PostgreSQL (dados)
- MQTT (streaming)

--------------------------------------------------

FASE 11 — OPEN SOURCE / DOAÇÃO

ESTRUTURA:
- repositório GitHub
- documentação completa
- dataset aberto
- modelos pré-treinados

LICENÇAS:
- MIT (livre)
- GPL (copyleft)

DOAÇÃO REAL:
- disponibilizar API pública
- permitir deploy comunitário
- criar rede distribuída de sensores

--------------------------------------------------

FASE 12 — APLICAÇÕES

1. MINERAÇÃO
- detectar depósitos minerais

2. SEGURANÇA
- detectar interferência eletromagnética
- localizar dispositivos ocultos

3. INDÚSTRIA
- monitoramento de máquinas

4. EXPLORAÇÃO ESPACIAL
- mapear anomalias na Lua/Marte

5. INFRAESTRUTURA CRÍTICA
- detectar falhas em redes elétricas

--------------------------------------------------

FASE 13 — MVP RÁPIDO (IMPLEMENTAÇÃO EM DIAS)

PASSOS:
1. coletar dataset público
2. aplicar pré-processamento simples
3. treinar Isolation Forest
4. criar API com FastAPI
5. visualizar com plot simples

RESULTADO:
sistema funcional básico

--------------------------------------------------

FASE 14 — ESCALA

MELHORIAS:
- aumentar volume de dados
- usar deep learning
- integrar múltiplos sensores
- adicionar análise em tempo real

--------------------------------------------------

PRINCÍPIO CENTRAL

o sistema funciona aprendendo o padrão normal com alta precisão
qualquer desvio relevante é tratado como anomalia

--------------------------------------------------

FIM DO PROJETO
