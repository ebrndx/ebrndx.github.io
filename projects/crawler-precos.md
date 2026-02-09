---
layout: page
title: "Crawler de Preços"
subtitle: "Inteligência de Preços e Competitividade via Google Shopping"
permalink: /projects/crawler-precos/
---

[← Voltar para Projetos](/projects)

---

## Visão Geral

**Problema:** O cliente não sabia se seus preços eram competitivos. Não havia comparação estruturada com o mercado, nem visão de quem estava mais barato e que tipo de concorrente era.

**Solução:** Crawler em Python que coleta preços do Google Shopping automaticamente e alimenta um dashboard de competitividade, cruzando a base do cliente com preços de mercado segmentados por tipo de varejista.

**Stack:** Python · Selenium · Pandas · Google Sheets (gspread) · WebDriver Manager

---

## O Problema

A pergunta central era simples: **estamos competitivos ou não?**

Mas responder isso exigia contexto que não existia:

- O cliente conhecia seus próprios preços, mas não tinha como compará-los com o mercado de forma sistemática
- Nem todo concorrente tem o mesmo peso — perder para a Amazon é diferente de perder para uma loja de nicho ou um marketplace internacional como AliExpress
- Para cada produto a decisão era diferente: onde vale competir, onde há margem para subir, onde estamos perdendo venda
- Sem histórico, não dava para entender se uma diferença de preço era pontual ou tendência

O processo manual — abrir Google Shopping, buscar produto, copiar dados — não escalava para dezenas de SKUs.

---

## A Solução

**Coleta:** Script Python com Selenium headless que lê a lista de produtos de uma Google Sheet, busca cada um no Google Shopping e extrai preço, vendedor, link e imagem. Roda sem interface gráfica, pode ser agendado.

**Dashboard:** Os dados alimentavam uma visão de competitividade por produto que cruzava os preços do cliente com os preços médios coletados:

- Posicionamento por produto — acima, abaixo ou alinhado com a média de mercado
- Segmentação por tipo de varejista — grande e-commerce, loja pequena/nicho, marketplace internacional
- Destaque de oportunidades — onde o cliente era mais barato (margem para subir) e onde era mais caro (risco de perda de venda)

---

## Resultados

- Visão clara de competitividade por produto, algo que não existia antes
- Decisões de pricing passaram a considerar o tipo de concorrente, não apenas o preço absoluto
- Base histórica com timestamp para acompanhar tendências e sazonalidade
- Processo manual eliminado — coleta automatizada e dashboard consolidado

---

[← Voltar para Projetos](/projects)
