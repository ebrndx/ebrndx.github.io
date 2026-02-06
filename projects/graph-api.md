---
layout: page
title: "Pipeline Graph API"
subtitle: "Coleta Automatizada de Dados Meta"
permalink: /projects/graph-api/
---

[← Voltar para Projetos](/projects)

---

## Visão Geral

**Problema:** Métricas de performance de Facebook Pages e Instagram eram coletadas manualmente ou dependiam de conectores pagos com limitações de granularidade, métricas disponíveis e controle sobre o pipeline.

**Solução:** Pipeline automatizado em Google Apps Script que coleta métricas detalhadas via Meta Graph API (v24.0) e armazena diretamente no BigQuery com deduplicação e particionamento.

**Stack:** Google Apps Script · Meta Graph API · BigQuery · AI-assisted coding

---

## O Problema

A coleta de dados de social media apresentava desafios operacionais concretos:

- **Granularidade limitada:** Conectores padrão não expunham todas as métricas disponíveis na API (ex: breakdown de reações, métricas de vídeo por threshold)
- **Custo de conectores:** Ferramentas como Supermetrics cobravam por volume e não cobriam todos os endpoints necessários
- **Fragilidade manual:** Exportações manuais dependiam de quem executava e quando, sem garantia de consistência
- **Cobertura parcial:** Stories do Instagram e métricas de Reels exigiam coleta separada, sem consolidação

---

## A Solução

Pipeline modular com scripts dedicados por plataforma e tipo de conteúdo:

**Facebook Pages:**
- Coleta de posts (imagens, carousels, vídeos, reels) com 50+ métricas por publicação
- Métricas de alcance (orgânico, pago, total), engajamento detalhado (reações por tipo, cliques por categoria), e video insights (views por threshold, watch time, completion rate)
- Agregações mensais de página (alcance, seguidores, visitas ao perfil)

**Instagram:**
- Feed (imagens, carousels, vídeos, reels) com métricas de alcance, engajamento, saves e shares
- Stories com views, replies, shares e follows
- Métricas de conta (alcance mensal, profile views, seguidores)

**Arquitetura do pipeline:**
- Autenticação via Bearer token com retry automático e backoff exponencial
- Paginação cursor-based para volumes grandes (100+ páginas, 1000+ mídias)
- Cache de respostas da API por execução para otimizar chamadas
- Staging table temporária no BigQuery com MERGE (upsert) para deduplicação
- Particionamento por data de snapshot para queries eficientes

---

## Destaques Técnicos

- **Idempotente:** Re-execuções não geram duplicatas — o MERGE garante consistência
- **Graceful degradation:** Falha em um post não interrompe a coleta dos demais
- **Rate limiting:** Tratamento automático do erro 80004 (rate limit) com espera adaptativa
- **Detecção de formato:** Classificação automática do tipo de mídia (image, carousel, video, reel) para aplicar métricas corretas
- **AI-assisted coding:** Desenvolvimento acelerado com apoio de IA para construção dos scripts e mapeamento de métricas da API

---

## Resultados

- Coleta automatizada de métricas de todas as páginas e contas gerenciadas
- Substituição de exportações manuais e dependência de conectores pagos
- Dados granulares disponíveis no BigQuery para análise e dashboards
- Pipeline resiliente com retry, deduplicação e monitoramento de execução

---

[← Voltar para Projetos](/projects)
