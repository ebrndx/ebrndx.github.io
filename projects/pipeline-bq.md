---
layout: page
title: "Pipeline BigQuery"
subtitle: "Migração e Arquitetura Medallion"
permalink: /projects/pipeline-bq/
---

[← Voltar para Projetos](/projects/)

---

## Visão Geral

**Problema:** Dados de mídia digital operavam em Google Sheets como camada principal, com limitações de confiabilidade, performance, governança e escala.

**Solução:** Migração para BigQuery com arquitetura Medallion (Bronze/Silver/Gold), eliminando dependência de planilhas e criando um pipeline robusto para analytics.

**Stack:** BigQuery · Supermetrics · SQL · Looker Studio

---

## O Problema

Planilhas como camada operacional de dados traziam riscos crescentes:

- **Confiabilidade:** alterações manuais quebravam o pipeline sem aviso, sem rastreabilidade de quem mudou o quê
- **Performance:** joins e agregações pesadas travavam os dashboards no Looker, com carregamento lento por full scan
- **Governança:** fórmulas escondidas viravam "regras de negócio fantasma", sem versionamento de schema
- **Escala:** mais clientes e frentes significavam mais planilhas, mais risco e dependência de quem "conhecia a planilha"

---

## A Solução

Arquitetura em três camadas no BigQuery, cada uma com responsabilidade clara:

**Bronze (raw)** — ingestão dos dados brutos, mantendo granularidade original com metadados de carga (timestamp, origem, batch). Particionamento por data e clusterização por dimensões de consulta frequente.

**Silver (cleaned)** — normalização de nomenclaturas, joins com tabelas de referência (taxonomia), deduplicação e tratamento de nulos. Regras de negócio que antes viviam em fórmulas de planilha passaram a ser SQL documentado e versionado.

**Gold (aggregated)** — visão consolidada pronta para consumo nos dashboards, com métricas calculadas (CTR, CPC, CPA) e agregações por dimensões de negócio. Materialização agendada para garantir performance.

---

## Estratégia de Migração

A migração seguiu uma abordagem incremental com convivência temporária entre fontes:

1. **Mapeamento** — inventário de todas as planilhas ativas, suas transformações, consumidores e criticidade
2. **Implementação Bronze/Silver/Gold** — criação das camadas por fonte, começando pelas menos críticas
3. **Validação paralela** — queries de comparação diária entre planilha e BigQuery, com critério de delta < 1%
4. **Troca no Looker** — substituição da fonte de dados nos dashboards após validação
5. **Desligamento** — congelamento das planilhas originais após período de convivência sem divergências

---

## Resultados

- Dashboards que travavam agora carregam com performance consistente
- Schema versionado e documentado, sem "regras fantasma" em fórmulas
- Pipeline suporta volume crescente sem intervenção manual
- Auditoria de alterações e ponto único de verdade para métricas de mídia
- Redução do custo operacional de manutenção

---

[← Voltar para Projetos](/projects/)
