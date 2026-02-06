---
layout: page
title: "Lexicon: Plataforma de Governança de Taxonomia"
subtitle: "85% de redução em erros e eliminação de planilhas compartilhadas"
---

[← Voltar para Projetos](/projects/)

---

# Lexicon: Plataforma de Governança de Taxonomia

## Visão Geral

**Problema:** Times de mídia perdiam horas corrigindo erros de nomenclatura de campanhas. Planilhas compartilhadas geravam inconsistências, duplicatas e retrabalho constante no BI.

**Solução:** Plataforma de governança com validação na origem, fluxos automatizados de aprovação e registro canônico em SharePoint.

**Impacto:**
- 85% de redução em erros de taxonomia
- Tempo de aprovação: de 3 dias para 4 horas
- Eliminação de 12 planilhas compartilhadas
- Fonte única de verdade para 50+ usuários

**Stack:**
Power Apps · Power Automate · SharePoint Lists · Microsoft Teams

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
> "Quinta-feira, 18h: descobrimos que 2 semanas de dados estavam errados porque alguém digitou 'Meta' em vez de 'meta'. Perdemos o final de semana corrigindo."

**Por que não escalava:**
- Google Sheets tem limite de 5M células (atingido mensalmente)
- Sem validação em tempo real
- Sem controle de concorrência
- Sem trilha de auditoria estruturada
- Múltiplas versões conflitantes da "verdade"

---

## A Solução

### Arquitetura de Alto Nível
```
Power Apps (validação UX + interface)
    ↓
Power Automate (orquestração + regras + validação)
    ↓
SharePoint Lists (armazenamento + versionamento)
    ↓
Teams (aprovações via Adaptive Cards + notificações)
```

### Decisões de Arquitetura

**1. Por que Power Platform em vez de app custom?**

Prós da escolha:
- Time de negócio já usava Microsoft 365
- Integração nativa com Teams (aprovações já aconteciam lá)
- Low-code permitiu iteração rápida com feedback do usuário
- Menor custo de manutenção
- Deploy rápido sem envolver time de engenharia

Trade-offs aceitos:
- Menos flexibilidade que código puro
- Limite de 500 chamadas/dia por fluxo (contornado com paralelização)
- Dependência do ecossistema Microsoft

**2. Por que SharePoint Lists como repositório?**

Prós:
- Permissões integradas com Azure AD
- Auditoria nativa (created_by, modified_by, modified_at)
- Versionamento automático de itens
- API REST para integrações futuras
- Backup e disaster recovery gerenciados

Trade-offs:
- Limite de 30M itens por lista (não foi problema na escala atual)
- Performance degrada acima de 5k itens (resolvido com arquivamento mensal)
- Menos flexível que banco de dados relacional

**3. Por que Teams para aprovações?**

Teams como canal de aprovação porque:
- Time já operava 100% no Teams
- Adaptive Cards oferecem UX rica sem sair do contexto
- Notificações instantâneas
- Histórico de conversas associado ao registro
- Zero fricção de adoção

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
- Join confiável com sistemas externos

### 2. Fluxos Principais (Power Automate)

**A. Save and Export**
- Trigger: Submit no Power Apps
- Validação de campos obrigatórios e domínios permitidos
- Geração de chave canônica
- Busca de duplicata (por chave)
- Decisão: inserir novo ou retornar existente
- Gravação no SharePoint
- Retorno de status para o usuário

**B. History Save Export**
- Trigger: Update em registro base
- Captura estado anterior e novo
- Cálculo de diff (campos alterados)
- Gravação em lista de auditoria (history)
- Incremento de versão
- Atualização do registro current

**C. Duplicate Response**
- Trigger: Tentativa de cadastro duplicado
- Retorna registro canônico existente
- Alerta usuário com link do original no SharePoint
- Registra tentativa para análise de padrões

**D. Graveyard Purge**
- Trigger: Agendado (semanal)
- Identifica registros obsoletos/inativos
- Move para lista de "suspensas" com motivo
- Aplica TTL respeitando auditoria

### 3. Validação em Camadas

**Camada 1: Power Apps (tempo real)**
- Campos obrigatórios visuais
- Dropdowns de valores permitidos (listas mestres)
- Validação de formato (datas, padrões)
- Presets e templates (reduz digitação e erro)

**Camada 2: Power Automate (pré-persistência)**
- Regras de negócio complexas
- Deduplicação por chave canônica
- Validação cruzada entre campos
- Normalização final antes de gravar

**Por que duas camadas?**
- UX: erro cedo é melhor que erro tarde
- Segurança: usuário pode bypassar frontend (chamada direta à API)
- Auditoria: log de tentativas inválidas para análise

### 4. Aprovações no Teams

**Trigger:** Registro criado/editado por usuário sem permissão total

**Adaptive Card contém:**
- Resumo do registro (campos principais)
- Diff visual (se for edição - campos alterados destacados)
- Botões: Aprovar / Rejeitar / Solicitar ajuste
- Campo de comentário obrigatório

**Após decisão:**
- Registro atualizado com status, aprovador, timestamp
- Notificação ao solicitante (Teams + email)
- Event log gravado na lista de history
- Thread do Teams linkada ao registro

**Timeout:** 48h sem resposta → escala para owner do processo

### 5. Estrutura no SharePoint

**Lista: lexicon_current** (estado canônico)
- ID (auto)
- canonical_key (indexed, unique)
- cliente, veiculo, frente, estrategia, objetivo, produto, pilar, target, data_versao
- version (número inteiro)
- status (rascunho, aprovado, rejeitado, obsoleto)
- created, created_by
- modified, modified_by
- approval_status, approval_date, approval_by
- comments (texto longo)

**Lista: lexicon_history** (auditoria)
- ID (auto)
- canonical_key (indexed)
- event_type (create, update, approve, reject, archive)
- changed_fields (múltiplas linhas de texto)
- change_reason
- event_date, event_user
- link_to_current (lookup)

**Listas auxiliares:**
- Clientes (lista mestre)
- Veículos (lista mestre)
- Frentes (lista mestre)
- Estratégias (lista mestre)
- Presets (templates pré-configurados)

---

## Desafios Técnicos Resolvidos

### 1. Concorrência em Submits Simultâneos

**Problema:** 2 usuários criando registro idêntico ao mesmo tempo.

**Solução:**
- Request ID único por tentativa (GUID)
- Lock otimista: checagem de duplicata com timestamp
- Retry com backoff exponencial
- Último check antes do commit no SharePoint

### 2. Late Arrivals e Dados Retroativos

**Problema:** Usuário corrige registro de semana passada, como rastrear?

**Solução:**
- Versionamento automático no SharePoint
- Lista de history captura TODAS as mudanças
- Flag de "atualizado retroativamente" visível no registro
- Notificação no Teams quando registro antigo é alterado

### 3. Limite de 500 Chamadas/Dia por Fluxo

**Problema:** Power Automate tem rate limit agressivo.

**Solução:**
- Paralelização: 1 fluxo por tipo de operação
- Child flows para operações repetitivas
- Batch de notificações (agrupa em vez de enviar 1 por 1)
- Otimização de loops (usar "Apply to each" com filtros)

### 4. Performance em Listas Grandes

**Problema:** SharePoint degrada com muitos itens.

**Solução:**
- Índices em canonical_key, status e modified
- Arquivamento mensal de registros antigos
- Views filtradas por default (só registros ativos)
- Paginação no Power Apps (lazy loading)

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

### Impacto em Qualidade de Dados

**Campos vazios ou incorretos:**
- Antes: 12% de registros com problema
- Depois: <1%

**Inconsistências de nomenclatura:**
- Antes: 200+ variações do mesmo conceito
- Depois: 0 (listas mestres + validação)

**Confiabilidade:**
- Chave canônica determinística = 100% de confiabilidade em joins
- Versionamento completo = rastreabilidade total

### Impacto Operacional

**Produtividade:**
- Horas economizadas/mês: ~120h
- Retrabalho evitado: ~R$ 15k/mês (estimativa)
- Onboarding de novos analistas: de 2 semanas para 3 dias

**Governança:**
- 100% dos registros com dono identificado
- 100% das mudanças rastreadas (quem, quando, por quê)
- 0 versões conflitantes

---

## Observabilidade e Governança

### Monitoramento via SharePoint Views

**Views customizadas criadas:**
- Registros pendentes de aprovação
- Registros editados nos últimos 7 dias
- Top usuários criadores
- Taxa de rejeição por usuário
- Registros obsoletos (candidatos a arquivo)

### Alertas no Teams

**Configurados via Power Automate:**
- Resumo semanal de métricas (envio automático segunda 9h)
- Spike de rejeições (>10% acima da média)
- Duplicatas frequentes (mesmo usuário tentando 3+ vezes)
- Timeout de aprovações (>48h sem resposta)
- Registro crítico alterado (flag de impacto alto)

### Auditoria Completa

Cada ação registrada com:
- Quem (usuário do Azure AD)
- Quando (timestamp com timezone São Paulo)
- O quê (campos alterados com diff)
- Por quê (motivo quando aplicável)
- Onde (origem: Power Apps, API, import)

**Retenção:**
- Current: sempre disponível
- History: mantido indefinidamente (baixo volume)
- Logs de fluxo: 28 dias (padrão Power Automate)

---

## Aprendizados

### O que funcionou muito bem

1. **Validação dupla (UX + backend)**
   - Reduziu fricção (usuário corrige cedo)
   - Manteve segurança (não confia no frontend)
   - 92% de aprovação no primeiro envio

2. **Chave canônica determinística**
   - Resolveu 90% dos problemas de duplicata
   - Permitiu idempotência natural
   - Base para integrações futuras

3. **Aprovações via Teams**
   - Zero fricção de adoção (time já vivia no Teams)
   - Adaptive Cards são visualmente claras
   - Histórico de decisões no contexto

4. **SharePoint como fonte única**
   - Backup e disaster recovery gerenciados
   - Permissões granulares por grupo
   - Versionamento nativo

### O que faria diferente

1. **Definir política de versionamento antes**
   - Demoramos 2 sprints para decidir quando incrementar versão
   - Causou ruído inicial no history
   - Tivemos que fazer limpeza retroativa

2. **Piloto mais longo**
   - 2 semanas foi curto, ideal seria 4 semanas
   - Algumas regras de negócio só apareceram no rollout
   - Ajustes quebraram fluxos já configurados

3. **Documentação desde o dia 1**
   - Gastamos 1 sprint inteira documentando no final
   - Deveria ter sido incremental
   - Onboarding de novos usuários foi mais lento

4. **Mais atenção aos limites do SharePoint**
   - Esbarramos em limite de 5k itens em view
   - Tivemos que criar índices retroativamente
   - Causou lentidão temporária

### Próximos Passos

**Curto prazo:**
- App mobile (Power Apps Mobile) para aprovações
- Integração via API com sistemas externos
- Auto-sugestão de taxonomia baseada em histórico

**Médio prazo:**
- Expansão para outros times (não só mídia)
- Templates por vertical de cliente
- Dashboard de métricas de governança (Power BI)

**Longo prazo:**
- ML para detecção de anomalias em cadastros
- Integração com plataformas de mídia (validação cruzada)
- Export estruturado para sistemas analíticos

---

## Tecnologias e Ferramentas

**Desenvolvimento:**
- Power Apps (Canvas App)
- Power Automate (Cloud Flows)
- SharePoint Lists
- Azure AD (grupos e permissões)

**Comunicação:**
- Microsoft Teams (Adaptive Cards)
- Outlook (notificações de backup)

**Observabilidade:**
- SharePoint Views customizadas
- Power Automate Run History
- Logs de auditoria nativos do SharePoint

---

[← Voltar para Projetos](/projects/)
