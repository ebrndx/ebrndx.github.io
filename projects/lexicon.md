---
title: "Lexicon: Plataforma de Governança de Taxonomia"
layout: single
permalink: /projects/lexicon/
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/lexicon-banner.jpg
excerpt: "Plataforma low-code que reduziu erros de taxonomia em 85% e eliminou dependência de planilhas compartilhadas"
tags:
  - Power Platform
  - BigQuery
  - Governança
  - Automação
---

## Visão Geral

**Problema:** Times de mídia perdiam horas corrigindo erros de nomenclatura de campanhas. Planilhas compartilhadas geravam inconsistências, duplicatas e retrabalho constante no BI.

**Solução:** Plataforma de governança com validação na origem, fluxos automatizados de aprovação e exportação canônica para análise.

**Impacto:**
- 85% de redução em erros de taxonomia
- Tempo de aprovação: de 3 dias para 4 horas
- Eliminação de 12 planilhas compartilhadas
- Fonte única de verdade para 50+ usuários

**Stack:**
Power Apps · Power Automate · SharePoint Lists · Teams · BigQuery · Looker Studio

---

## O Problema

### Contexto
Em operações de mídia digital com múltiplos clientes, frentes e canais, a padronização de nomenclaturas é crítica para análise e otimização. Mas o processo estava quebrado:

**Sintomas:**
- Cada analista preenchia campanhas de um jeito diferente
- Erros descobertos apenas no BI (3-5 dias depois)
- Planilhas compartilhadas travavam e geravam versões conflitantes
- 30% das horas de BI eram gastas corrigindo dados na base
- Impossível rastrear quem mudou o quê e quando

**Dor real:**
> "Quinta-feira, 18h: descobrimos que 2 semanas de dados estavam errados porque alguém digitou 'Meta' em vez de 'facebook'. Perdemos o final de semana corrigindo."

**Por que não escalava:**
- Google Sheets tem limite de 5M células (atingido mensalmente)
- Sem validação em tempo real
- Sem controle de concorrência
- Sem trilha de auditoria estruturada
- Integração frágil com BI (importação manual ou scripts quebradiços)

---

## A Solução

### Arquitetura de Alto Nível

[DIAGRAMA 1: Fluxo completo do Lexicon]
```
┌─────────────┐
│ Power Apps  │  Validação UX + regras de preenchimento
└──────┬──────┘
       │
       ▼
┌─────────────┐
│Power Automate│ Orquestração + validação + deduplicação
└──────┬──────┘
       │
       ├──────────────┐
       ▼              ▼
┌─────────────┐  ┌─────────┐
│ SharePoint  │  │  Teams  │  Aprovações + notificações
│   Lists     │  └─────────┘
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  BigQuery   │  Camada analítica (current + history)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│Looker Studio│  Consumo + dashboards de governança
└─────────────┘
```

### Decisões de Arquitetura

**1. Por que Power Platform em vez de app custom?**

✅ Prós da escolha:
- Time de negócio já usava Microsoft 365
- Integração nativa com Teams (aprovações já aconteciam lá)
- Low-code permitiu iteração rápida com feedback do usuário
- Menor custo de manutenção

❌ Trade-offs aceitos:
- Menos flexibilidade que código puro
- Limite de 500 chamadas/dia por fluxo (contornado com paralelização)

**2. Por que SharePoint Lists como staging?**

✅ Prós:
- Permissões integradas com Azure AD
- Auditoria nativa (created_by, modified_by)
- API REST confiável para exportação
- Versionamento de itens (history log)

❌ Trade-offs:
- Limite de 30M itens por lista (não foi problema na escala atual)
- Performance degrada acima de 5k itens (resolvido com arquivamento mensal)

**3. Por que BigQuery como destino final?**

✅ BigQuery virou a fonte única de verdade porque:
- BI já consumia de lá
- Escala ilimitada
- Query rápida com particionamento
- Versionamento por tabela (current vs history)
- Integração com Looker Studio sem latência

---

## Implementação

### 1. Chave Canônica (Core do Sistema)

**Desafio:** Como garantir que "Meta - Claro - Awareness" é o mesmo registro que "meta-claro-awareness"?

**Solução:** Chave determinística gerada de forma consistente.

**Estrutura:**
```
cliente_veiculo_frente_estrategia_objetivo_produto_pilar_target_ps_dataVersao
```

**Normalização antes de gerar:**
- Lowercase
- Trim de espaços
- Remoção de caracteres especiais
- Ordem fixa de campos

**Resultado:**
- Idempotência: reenvio não duplica
- Deduplicação automática
- Join confiável entre sistemas

**Exemplo genérico:**
```
Input:  Cliente A | Meta | Frente X | ...
Output: clientea_meta_frentex_..._v1
```

### 2. Fluxos Principais (Power Automate)

[DIAGRAMA 2: Mapa de fluxos e triggers]

**A. Save and Export**
- Trigger: Submit no Power Apps
- Validação de campos obrigatórios e domínios permitidos
- Geração de chave canônica
- Busca de duplicata (por chave)
- Decisão: inserir novo ou retornar existente
- Export para BigQuery
- Retorno de status para o usuário

**B. History Save Export**
- Trigger: Update em registro base
- Captura estado anterior e novo
- Cálculo de diff (campos alterados)
- Gravação em tabela de auditoria
- Incremento de versão
- Sincronização com BigQuery (current + history)

**C. Duplicate Response**
- Trigger: Tentativa de cadastro duplicado
- Retorna registro canônico existente
- Alerta usuário com link do original
- Registra tentativa para análise de padrões

**D. Graveyard Purge**
- Trigger: Agendado (semanal)
- Identifica registros obsoletos/inativos
- Move para lista de "suspensas" com motivo
- Aplica TTL respeitando auditoria

### 3. Validação em Camadas

**Camada 1: Power Apps (tempo real)**
- Campos obrigatórios visuais
- Dropdowns de valores permitidos
- Validação de formato (datas, emails)
- Presets e templates (reduz digitação)

**Camada 2: Power Automate (pré-persistência)**
- Regras de negócio complexas
- Deduplicação por chave
- Validação cruzada entre campos
- Normalização final

**Por que duas camadas?**
- UX: erro cedo é melhor que erro tarde
- Segurança: usuário pode bypassar frontend (API direta)
- Auditoria: log de tentativas inválidas para análise

### 4. Aprovações no Teams

[DIAGRAMA 3: Fluxo de aprovação com Adaptive Card]

**Trigger:** Registro criado/editado por usuário sem permissão total

**Adaptive Card contém:**
- Resumo do registro (campos principais)
- Diff visual (se for edição)
- Botões: Aprovar / Rejeitar / Solicitar ajuste
- Campo de comentário obrigatório

**Após decisão:**
- Registro atualizado com status, aprovador, timestamp
- Notificação ao solicitante
- Event log gravado no history
- Sincronização com BigQuery

**Timeout:** 48h sem resposta → escala para owner do processo

### 5. Export para BigQuery

**Estrutura de Tabelas:**

**lexicon_current** (estado canônico):
```sql
-- Schema simplificado (campos de exemplo)
canonical_key STRING PRIMARY KEY
cliente STRING
veiculo STRING
frente STRING
estrategia STRING
... (demais dimensões)
version INT64
created_at TIMESTAMP
updated_at TIMESTAMP
created_by STRING
updated_by STRING
approval_status STRING
approval_at TIMESTAMP
approval_by STRING
source STRING (sempre 'lexicon')
```

**lexicon_history** (auditoria):
```sql
-- Mesmos campos + metadados de mudança
event_id STRING PRIMARY KEY
canonical_key STRING
event_type STRING (create|update|approve|reject)
changed_fields ARRAY<STRING>
change_reason STRING
event_timestamp TIMESTAMP
event_user STRING
```

**Particionamento:**
- current: sem partição (sempre 1 linha por chave)
- history: PARTITION BY DATE(event_timestamp)

**Cluster:**
- CLUSTER BY frente, veiculo, estrategia (padrão de consulta)

**Sincronização:**
- Incremental por evento (Power Automate → BigQuery API)
- Fallback: full refresh diário (recuperação de falhas)

---

## Desafios Técnicos Resolvidos

### 1. Concorrência em Submits Simultâneos

**Problema:** 2 usuários criando registro idêntico ao mesmo tempo.

**Solução:**
- Request ID único por tentativa
- Lock otimista: checagem de duplicata com timestamp
- Retry com backoff exponencial
- Último check antes do commit

### 2. Late Arrivals e Dados Retroativos

**Problema:** Usuário corrige registro de semana passada, como refletir no BI?

**Solução:**
- Lookback window de 7 dias nas queries Gold
- Recálculo automático de agregações afetadas
- Flag de "dado atualizado retroativamente" no dashboard

### 3. Limite de 500 Chamadas/Dia por Fluxo

**Problema:** Power Automate tem rate limit agressivo.

**Solução:**
- Paralelização: 1 fluxo por tipo de operação
- Batch de exports para BigQuery (a cada 15 min em vez de tempo real)
- Child flows para operações repetitivas

---

## Resultados

### Métricas de Adoção

**Antes do Lexicon:**
- 12 planilhas compartilhadas
- 30% das horas de BI em correção de dados
- Erro descoberto em média 3 dias depois
- 0% de rastreabilidade de mudanças

**Depois (3 meses após rollout):**
- 0 planilhas (migração completa)
- 85% de redução em erros de taxonomia
- 95% dos registros validados em < 10 segundos
- 100% de trilha de auditoria

### Tempo de Aprovação

| Métrica | Antes | Depois |
|---------|-------|--------|
| Tempo médio de aprovação | 3 dias | 4 horas |
| Taxa de aprovação primeiro envio | 45% | 92% |
| Registros duplicados/mês | ~80 | <5 |

### Impacto no BI

**Qualidade de Dados:**
- Campos nulos: de 12% para <1%
- Inconsistências de nomenclatura: de 200+ variações para 0
- Confiabilidade de joins: 100% (chave determinística)

**Produtividade:**
- Horas economizadas/mês: ~120h
- Retrabalho evitado: ~R$ 15k/mês (estimativa)

---

## Observabilidade e Governança

### Dashboard de Qualidade (Looker Studio)

**Métricas monitoradas:**
- Taxa de aprovação por usuário/frente
- Tempo médio de aprovação
- Tentativas de duplicação (padrões)
- Uso de presets vs preenchimento manual
- Campos com maior taxa de erro

**Alertas no Teams:**
- Resumo semanal de métricas
- Spike de rejeições (>10% acima da média)
- Duplicatas frequentes (mesmo usuário/chave)
- Timeout de aprovações (>48h)

### Auditoria Completa

Cada ação registrada com:
- Quem (usuário)
- Quando (timestamp com timezone)
- O quê (campos alterados com diff)
- Por quê (motivo quando aplicável)
- Onde (origem: app, API, import)

**Retenção:**
- Current: sempre disponível
- History: 24 meses em hot storage, 60 meses em cold storage

---

## Aprendizados

### O que funcionou muito bem

1. **Validação dupla (UX + backend)**
   - Reduziu fricção (usuário corrige cedo)
   - Manteve segurança (não confia no frontend)

2. **Chave canônica determinística**
   - Resolveu 90% dos problemas de duplicata
   - Permitiu idempotência natural

3. **Aprovações via Teams**
   - Zero fricção de adoção (time já vivia no Teams)
   - Adaptive Cards são visualmente claras

4. **BigQuery como destino final**
   - Eliminou necessidade de ETL adicional
   - BI consumiu direto sem transformação

### O que faria diferente

1. **Definir política de versionamento antes**
   - Demoramos 2 sprints para decidir quando incrementar versão
   - Causou ruído inicial no history

2. **Piloto mais longo**
   - 2 semanas foi curto, ideal seria 4 semanas
   - Algumas regras de negócio só apareceram no rollout

3. **Documentação desde o dia 1**
   - Gastamos 1 sprint inteira documentando no final
   - Deveria ter sido incremental

### Próximos Passos

**Curto prazo:**
- Integração com ferramentas de mídia (API Meta, Google Ads)
- Auto-sugestão de taxonomia (ML básico)
- App mobile para aprovações

**Médio prazo:**
- Expansão para outros times (não só mídia)
- Templates por vertical de cliente
- Integração com CRM para enriquecimento

---

## Tecnologias e Ferramentas

**Desenvolvimento:**
- Power Apps (Canvas)
- Power Automate (Cloud Flows)
- SharePoint Lists
- Azure AD (grupos e permissões)

**Integração:**
- Microsoft Teams (Adaptive Cards)
- BigQuery API (Python connector em Cloud Function)
- Looker Studio (BI)

**Observabilidade:**
- Application Insights (logs de fluxos)
- BigQuery (auditoria)
- Custom dashboard (Looker Studio)

---

[Voltar para Projetos](/projects/)
