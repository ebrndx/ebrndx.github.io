---
layout: page
title: "O Guardião"
subtitle: "Extensão Chrome para Validação de Budgets de Mídia"
permalink: /projects/guardiao/
---

[← Voltar para Projetos](/projects)

---

## Visão Geral

**Problema:** Erros de preenchimento de budget em plataformas de mídia (Meta Ads e Google Ads) geravam gastos acima do planejado, com impacto financeiro direto para a operação.

**Solução:** Extensão Chrome que monitora campos de orçamento em tempo real e alerta o operador quando o valor ultrapassa o limite diário definido.

**Stack:** Chrome Extension (Manifest v3) · JavaScript · Content Scripts

**Resultado:** Zerou os erros de budget de mídia dentro da agência.

---

## O Problema

Na operação de mídia digital, um dígito a mais no campo de budget pode transformar R$ 2.000 em R$ 20.000 de investimento diário. Esse tipo de erro:

- Só era percebido no dia seguinte, quando o valor já tinha sido gasto
- No Meta Ads, operadores confundiam orçamento diário com vitalício, multiplicando o gasto
- Dependia de conferência manual, sujeita a falha humana e esquecimento
- Acontecia com frequência em equipes com múltiplos analistas e contas

---

## A Solução

Extensão leve que roda direto no navegador e intercepta a entrada de valores nas plataformas:

**No Meta Ads:**
- Monitora campos de orçamento diário e vitalício em tempo real
- Distingue entre os dois tipos de budget para evitar confusão
- Alerta quando o valor ultrapassa o limite diário definido
- Exibe aviso sobre impostos (PIS/Cofins e ISS) não refletidos no Ads Manager

**No Google Ads:**
- Varre budgets de todas as campanhas visíveis na conta
- Detecta valores acima do limite com deduplicação por campanha
- Acompanha mudanças de paginação para garantir cobertura completa

**Interface:**
- Popup com status ativo e botões de acesso rápido para Meta e Google Ads
- Alertas via popup e notificações do navegador
- Indicador visual de status (ativo/inativo)

---

## Resultados

- Erros de budget de mídia na agência: **zero** após implantação
- Cobertura em todas as contas de Meta Ads e Google Ads da operação
- Adoção pelo time sem necessidade de treinamento

---

[← Voltar para Projetos](/projects)
