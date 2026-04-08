PROJETO: TRACKASIFI SCRAPPE
TIPO: Protocolo e plataforma open source de inteligência colaborativa (defensiva)
OBJETIVO: Transformar indicadores técnicos distribuídos (IOCs) em inteligência coletiva para reduzir tempo de detecção de ameaças sem violar privacidade

======================================================================
1. PRINCÍPIO FUNDAMENTAL
======================================================================

O sistema NÃO coleta tráfego bruto nem conteúdo de comunicação.

O sistema coleta apenas:
- Indicadores técnicos mínimos (IOCs)
- Dados anonimizados e agregados
- Eventos sem identificação pessoal

Base legal e ética:
- Consentimento explícito
- Minimização de dados
- Anonimização
- Transparência
- Revogação

Referência legal:
Lei Geral de Proteção de Dados (LGPD)

======================================================================
2. VISÃO DO SISTEMA
======================================================================

Arquitetura em 3 camadas:

1) Agente local (edge)
2) Protocolo de dados (padronização)
3) Rede de ingestão e distribuição (hub ou federado)

======================================================================
3. ESCOPO DO MVP
======================================================================

MVP NÃO inclui:
- Coleta de payload
- Inspeção profunda de pacotes
- Monitoramento invasivo
- Identificação de usuário

MVP inclui:
- Extração básica de IOCs
- Envio via API
- Banco de dados
- Feed público
- Consentimento + revogação

======================================================================
4. DEFINIÇÃO DE IOC (INDICADORES)
======================================================================

Tipos aceitos:
- IP suspeito
- Domínio suspeito
- URL truncada
- Hash de arquivo (SHA256)
- Porta/protocolo anômalo

Campos padrão:

{
  indicator_type: "ip|domain|hash|url",
  indicator_value: string,
  confidence: number (0-100),
  source_app: string (genérico),
  event_time_bucket: timestamp arredondado (ex: hora),
  country: opcional (granularidade baixa),
  signature: assinatura criptográfica
}

NUNCA incluir:
- Nome de usuário
- Email
- Cookies
- Headers completos
- Conteúdo de requisição
- IP local do usuário
- Identificadores únicos

======================================================================
5. AGENTE LOCAL (EDGE)
======================================================================

Função:
- Observar eventos locais de forma limitada
- Extrair apenas indicadores
- Aplicar filtro de privacidade
- Enviar dados anonimizados

Componentes:

1) Detector simples:
- Lista de domínios suspeitos
- Regex de padrões maliciosos
- Hashing de arquivos executados

2) Privacy Filter:
- Remove PII
- Trunca URLs
- Remove parâmetros sensíveis
- Aplica hashing com sal

3) Consent Manager:
- Tela de opt-in
- Registro de consentimento
- Opção de revogação
- Versionamento de termos

4) Sender:
- Envio via HTTPS
- Assinatura de eventos
- Rate limit

======================================================================
6. BACKEND (MVP CENTRALIZADO)
======================================================================

Componentes:

1) API Gateway
- Recebe eventos
- Valida schema
- Valida assinatura

2) Ingestion Service
- Normaliza dados
- Remove duplicatas
- Aplica scoring inicial

3) Database
- PostgreSQL ou Elastic
- Estrutura append-only

4) Correlation Engine (básico)
- Frequência de ocorrência
- Cluster por indicador
- Score de reputação

5) Public Feed API
- GET /indicators
- Filtros por tipo, tempo, score

======================================================================
7. SEGURANÇA
======================================================================

- HTTPS obrigatório
- Assinatura de eventos
- Rate limiting
- Logs auditáveis
- Sanitização de entrada

Proteções futuras:
- Anti data poisoning
- Sistema de reputação
- Detecção de anomalias de envio

======================================================================
8. CONSENTIMENTO
======================================================================

Fluxo:

1) Usuário instala agente
2) Tela explica claramente:
   - O que é coletado
   - O que NÃO é coletado
   - Finalidade
3) Usuário aceita explicitamente
4) Sistema registra:
   - Timestamp
   - Versão dos termos
   - Hash do consentimento

Revogação:
- Usuário pode desativar
- Dados futuros não são enviados
- Solicitação de exclusão disponível

======================================================================
9. OPEN SOURCE
======================================================================

Licença:
- MIT ou Apache 2.0

Repositório inclui:
- Código do agente
- Backend
- Documentação
- Threat model
- Política de privacidade

Transparência:
- Logs agregados públicos
- Roadmap aberto
- Issues abertas

======================================================================
10. ROADMAP
======================================================================

FASE 1 (0-3 meses)
- Agente simples
- API básica
- Banco
- Feed público
- Consentimento funcional

FASE 2 (3-6 meses)
- Assinatura criptográfica
- Sistema de reputação
- Melhorias no filtro de privacidade
- Dashboard básico

FASE 3 (6-12 meses)
- Arquitetura federada
- Múltiplos nós
- Governança aberta
- Auditoria externa

======================================================================
11. FEDERAÇÃO (FUTURO)
======================================================================

Modelo:
- Nós independentes
- Sincronização entre nós
- Sem autoridade central

Benefícios:
- Resiliência
- Neutralidade
- Escalabilidade

======================================================================
12. GOVERNANÇA
======================================================================

Modelo:
- Fundação ou comunidade open source

Regras:
- Neutralidade
- Sem exclusividade
- Sem uso ofensivo
- Auditoria contínua

======================================================================
13. RISCOS E MITIGAÇÃO
======================================================================

RISCO: coleta indevida de dados
MITIGAÇÃO: filtro rígido + schema limitado

RISCO: uso abusivo
MITIGAÇÃO: termos + governança

RISCO: data poisoning
MITIGAÇÃO: reputação + validação

RISCO: percepção negativa
MITIGAÇÃO: transparência + open source

======================================================================
14. STACK SUGERIDA
======================================================================

Agente:
- Rust ou Go

Backend:
- Python (FastAPI) ou Node.js

Banco:
- PostgreSQL
- Elastic/OpenSearch

Infra:
- Docker
- VPS simples inicialmente

======================================================================
15. POSICIONAMENTO
======================================================================

NÃO comunicar como:
- vigilância
- coleta de dados
- ferramenta de inteligência estatal

COMUNICAR como:
- defesa colaborativa
- threat intelligence open source
- pesquisa de segurança

======================================================================
16. RESULTADO FINAL
======================================================================

Sistema capaz de:
- Coletar indicadores anonimizados
- Correlacionar eventos
- Distribuir inteligência técnica
- Reduzir tempo de detecção de ameaças

Sem:
- violar privacidade
- coletar dados sensíveis
- centralizar poder

======================================================================
FIM DO PROJETO
======================================================================
