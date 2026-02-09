---
layout: page
title: "Crawler de Preços"
subtitle: "Inteligência de Preços e Competitividade via Google Shopping"
permalink: /projects/crawler-precos/
---

[← Voltar para Projetos](/projects)

---

## Visão Geral

**Problema:** O cliente precisava entender se seus preços eram competitivos frente ao mercado, mas não tinha visibilidade sistematizada dos preços praticados por concorrentes — nem por tipo de varejista (grandes e-commerces, lojas menores, marketplaces internacionais).

**Solução:** Crawler em Python com Selenium headless que coleta preços do Google Shopping automaticamente, alimentando um dashboard de competitividade que cruza a base de produtos do cliente com os preços médios coletados, segmentados por tipo de varejista.

**Stack:** Python · Selenium · Pandas · Google Sheets (gspread) · WebDriver Manager

---

## O Problema

A necessidade central era responder: **estamos competitivos ou não?**

- **Sem visão de mercado:** O cliente conhecia seus próprios preços, mas não tinha como compará-los sistematicamente com o que concorrentes praticavam
- **Tipos de concorrente diferentes:** Um preço abaixo do seu em um grande e-commerce (Magazine Luiza, Amazon) tem peso diferente de um preço em uma loja pequena ou em um marketplace internacional como AliExpress — era preciso segmentar
- **Decisão por produto:** Para cada SKU da base do cliente, a pergunta era: vale a pena competir nesse preço? Estamos acima ou abaixo da média? Quem está mais barato?
- **Sem histórico:** Não havia base temporal para entender tendências de preço ou sazonalidade por concorrente

O processo manual não escalava: cada consulta exigia abrir o Google Shopping, buscar o produto, copiar dados um a um e repetir para cada termo da lista.

---

## A Solução

**Coleta automatizada (este projeto):**

- Termos de busca lidos de uma Google Sheet (aba `list`), cada linha representando um produto do catálogo do cliente
- Selenium headless navega ao Google Shopping, busca cada termo e extrai até 2 páginas de resultados
- Para cada resultado: nome do produto, preço normalizado, vendedor, URL direta e imagem
- Resultados persistidos na aba `result` da mesma planilha com timestamp por execução

**Dashboard de competitividade (consumo dos dados):**

Os dados coletados alimentavam um dashboard que cruzava a base de produtos do cliente com os preços obtidos:

- **Preço médio de mercado por produto:** Média dos preços coletados para cada SKU
- **Posicionamento do cliente:** Visualização de onde o preço do cliente ficava em relação à média (acima, abaixo, alinhado)
- **Segmentação por tipo de varejista:** Classificação dos vendedores em categorias — grande e-commerce, loja pequena/nicho, marketplace internacional — para entender contra quem se estava competindo
- **Análise de oportunidade:** Destaque dos produtos onde o cliente era mais barato (oportunidade de margem) e onde era mais caro (risco de perda de venda)

---

## Destaques Técnicos

- **Headless:** Execução sem interface gráfica, viabilizando uso em servidores e agendamento
- **Normalização de preço:** Tratamento de formato brasileiro (R$ 1.299,00) para float numérico
- **URL decoding:** Decodificação de URLs de redirect do Google Shopping para obter link direto do vendedor
- **Paginação automática:** Navega para segunda página de resultados quando disponível
- **WebDriver Manager:** Gerenciamento automático do ChromeDriver, sem necessidade de download manual
- **Resiliência:** Tratamento de exceções por item — falha em um produto não interrompe a coleta dos demais

---

## Resultados

- Visão clara de competitividade por produto, respondendo onde o cliente estava acima ou abaixo do mercado
- Segmentação de concorrentes por tipo de varejista, permitindo decisões mais qualificadas sobre posicionamento
- Base histórica de preços com timestamp para análise de tendências e sazonalidade
- Eliminação do processo manual de consulta, substituído por coleta automatizada e dashboard consolidado
- Dados acionáveis para equipes de e-commerce e pricing ajustarem estratégia por produto

---

[← Voltar para Projetos](/projects)
