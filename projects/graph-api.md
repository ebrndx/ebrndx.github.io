---
layout: page
title: "Pipelines de Integração de APIs"
subtitle: "Meta Graph API & AppsFlyer → BigQuery"
permalink: /projects/graph-api/
---

[← Voltar para Projetos](/projects)

---

## Visão Geral

**Problema:** Dados de social media e atribuição de app chegavam por exportações manuais ou conectores pagos que não cobriam tudo, custavam caro e quebravam sem aviso.

**Solução:** Pipelines próprios em Google Apps Script que puxam dados direto das APIs da Meta e da AppsFlyer e entregam no BigQuery, prontos para consumo.

**Stack:** Google Apps Script · Meta Graph API · AppsFlyer Master API · BigQuery · AI-assisted coding

---

## O Problema

A operação dependia de dados de duas fontes críticas — social media (Meta) e atribuição de app (AppsFlyer) — mas nenhuma das duas tinha um pipeline confiável:

- **Meta:** Conectores como Supermetrics não cobriam todas as métricas (reações por tipo, vídeo por threshold, Stories, Reels) e cobravam por volume. Exportar manualmente era frágil e inconsistente
- **AppsFlyer:** Métricas de instalação, compra e registro ficavam presas na plataforma ou dependiam de exports pontuais, sem histórico granular no data warehouse
- **Dados isolados:** Sem as duas fontes no mesmo lugar, não havia como cruzar performance de mídia com atribuição de conversão

O resultado: decisões baseadas em dados parciais, defasados e espalhados em ferramentas diferentes.

---

## A Solução

Dois pipelines construídos no mesmo padrão — Apps Script consumindo API, BigQuery como destino — com agendamento automático e sem dependência de ferramentas externas.

**Meta Graph API:**
- Facebook Pages e Instagram coletados com granularidade total: 50+ métricas por publicação, incluindo Stories, Reels, breakdowns de reação e vídeo
- Agregações mensais de página e conta (alcance, seguidores, visitas)
- Paginação cursor-based para contas com 1000+ mídias

**AppsFlyer:**
- Eventos de conversão (purchases, registros, ativações) e métricas de instalação (cost, clicks, installs, sessions) por campanha, adset e ad
- Janela com lag de 2 dias e lookback de 14 dias para capturar atribuições tardias
- O script se adapta sozinho quando a API muda nomes de colunas entre versões

**Padrão comum:**
- Autenticação por token, staging temporário, deduplicação, particionamento por data
- Re-executar não gera duplicatas — idempotência garantida
- Falha em um item não derruba a coleta dos demais

---

## Resultados

- Dados de Meta e AppsFlyer disponíveis no BigQuery para dashboards e análises cruzadas
- Conectores pagos eliminados — coleta própria com controle total sobre métricas e granularidade
- Exportações manuais substituídas por execução diária automática
- Cruzamento de mídia + atribuição viabilizado pela primeira vez no mesmo warehouse

---

[← Voltar para Projetos](/projects)
