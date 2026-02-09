---
layout: page
title: "Crawler de Precos"
subtitle: "Inteligencia de Precos e Competitividade via Google Shopping"
permalink: /projects/crawler-precos/
---

[← Voltar para Projetos](/projects)

---

## Visao Geral

**Problema:** O cliente precisava entender se seus precos eram competitivos frente ao mercado, mas nao tinha visibilidade sistematizada dos precos praticados por concorrentes — nem por tipo de varejista (grandes e-commerces, lojas menores, marketplaces internacionais).

**Solucao:** Crawler em Python com Selenium headless que coleta precos do Google Shopping automaticamente, alimentando um dashboard de competitividade que cruza a base de produtos do cliente com os precos medios coletados, segmentados por tipo de varejista.

**Stack:** Python · Selenium · Pandas · Google Sheets (gspread) · WebDriver Manager

---

## O Problema

A necessidade central era responder: **estamos competitivos ou nao?**

- **Sem visao de mercado:** O cliente conhecia seus proprios precos, mas nao tinha como compara-los sistematicamente com o que concorrentes praticavam
- **Tipos de concorrente diferentes:** Um preco abaixo do seu em um grande e-commerce (Magazine Luiza, Amazon) tem peso diferente de um preco em uma loja pequena ou em um marketplace internacional como AliExpress — era preciso segmentar
- **Decisao por produto:** Para cada SKU da base do cliente, a pergunta era: vale a pena competir nesse preco? Estamos acima ou abaixo da media? Quem esta mais barato?
- **Sem historico:** Nao havia base temporal para entender tendencias de preco ou sazonalidade por concorrente

O processo manual nao escalava: cada consulta exigia abrir o Google Shopping, buscar o produto, copiar dados um a um e repetir para cada termo da lista.

---

## A Solucao

**Coleta automatizada (este projeto):**

- Termos de busca lidos de uma Google Sheet (aba `list`), cada linha representando um produto do catalogo do cliente
- Selenium headless navega ao Google Shopping, busca cada termo e extrai ate 2 paginas de resultados
- Para cada resultado: nome do produto, preco normalizado, vendedor, URL direta e imagem
- Resultados persistidos na aba `result` da mesma planilha com timestamp por execucao

**Dashboard de competitividade (consumo dos dados):**

Os dados coletados alimentavam um dashboard que cruzava a base de produtos do cliente com os precos obtidos:

- **Preco medio de mercado por produto:** Media dos precos coletados para cada SKU
- **Posicionamento do cliente:** Visualizacao de onde o preco do cliente ficava em relacao a media (acima, abaixo, alinhado)
- **Segmentacao por tipo de varejista:** Classificacao dos vendedores em categorias — grande e-commerce, loja pequena/nicho, marketplace internacional — para entender contra quem se estava competindo
- **Analise de oportunidade:** Destaque dos produtos onde o cliente era mais barato (oportunidade de margem) e onde era mais caro (risco de perda de venda)

---

## Destaques Tecnicos

- **Headless:** Execucao sem interface grafica, viabilizando uso em servidores e agendamento
- **Normalizacao de preco:** Tratamento de formato brasileiro (R$ 1.299,00) para float numerico
- **URL decoding:** Decodificacao de URLs de redirect do Google Shopping para obter link direto do vendedor
- **Paginacao automatica:** Navega para segunda pagina de resultados quando disponivel
- **WebDriver Manager:** Gerenciamento automatico do ChromeDriver, sem necessidade de download manual
- **Resiliencia:** Tratamento de excecoes por item — falha em um produto nao interrompe a coleta dos demais

---

## Resultados

- Visao clara de competitividade por produto, respondendo onde o cliente estava acima ou abaixo do mercado
- Segmentacao de concorrentes por tipo de varejista, permitindo decisoes mais qualificadas sobre posicionamento
- Base historica de precos com timestamp para analise de tendencias e sazonalidade
- Eliminacao do processo manual de consulta, substituido por coleta automatizada e dashboard consolidado
- Dados acionaveis para equipes de e-commerce e pricing ajustarem estrategia por produto

---

[← Voltar para Projetos](/projects)
