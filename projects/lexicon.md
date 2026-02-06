---
layout: page
title: "Lexicon"
subtitle: "Plataforma de Governança de Taxonomia"
permalink: /projects/lexicon/
---

[← Voltar para Projetos](/projects)

---

## Visão Geral

**Problema:** Times de mídia perdiam horas corrigindo erros de nomenclatura de campanhas. Planilhas compartilhadas geravam inconsistências, duplicatas e retrabalho constante no BI.

**Solução:** Plataforma de governança com validação na origem, fluxos automatizados de aprovação e registro canônico centralizado.

**Stack:** Power Apps · Power Automate · SharePoint · Microsoft Teams

---

## O Problema

Em operações de mídia digital com múltiplos clientes, frentes e canais, a padronização de nomenclaturas é crítica para análise e otimização. Mas o processo estava quebrado:

- Cada analista preenchia campanhas de um jeito diferente
- Erros descobertos apenas no BI, dias depois
- Planilhas compartilhadas travavam e geravam versões conflitantes
- 30% das horas de BI eram gastas corrigindo dados na base
- Impossível rastrear quem mudou o quê e quando

---

## A Solução

Plataforma low-code construída sobre o ecossistema Microsoft 365, com três pilares:

1. **Validação na origem** — interface com campos controlados, listas mestres e validação em tempo real, eliminando erros antes de persistir
2. **Automação de fluxos** — orquestração de aprovações, deduplicação por chave canônica e versionamento automático
3. **Aprovações no Teams** — Adaptive Cards com resumo, diff visual e histórico de decisões, sem fricção de adoção

A escolha por Power Platform foi pragmática: o time já operava no Microsoft 365, o deploy era rápido e o custo de manutenção baixo.

---

## Resultados

| Métrica | Antes | Depois |
|---------|-------|--------|
| Planilhas compartilhadas | 12 | 0 |
| Erros de taxonomia | baseline | -85% |
| Tempo médio de aprovação | 3 dias | 4 horas |
| Aprovação no primeiro envio | 45% | 92% |
| Registros duplicados/mês | ~80 | <5 |
| Rastreabilidade de mudanças | 0% | 100% |

**Impacto operacional:**
- ~120 horas economizadas por mês
- Onboarding de novos analistas: de 2 semanas para 3 dias
- Fonte única de verdade para 80+ usuários

---

[← Voltar para Projetos](/projects)
