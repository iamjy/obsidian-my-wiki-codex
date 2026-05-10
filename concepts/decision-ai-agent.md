---
title: "의사결정 AI Agent"
date: 2026-05-10
tags:
  - ai-agent
  - decision-support
  - kpi
type: concept
status: active
source: "ingests/2026-05-10-decision-ai-agent-kpi.md"
related:
  - "[[2026-05-10-decision-ai-agent-kpi]]"
  - "[[semantic-mcp]]"
---

# 의사결정 AI Agent

## Definition

의사결정 AI Agent는 단순 질의응답이나 데이터 조회를 넘어, KPI 변화의 원인을 분석하고 실행 가능한 개선안을 제안하는 decision support agent다. 목표는 의사결정을 대체하는 것이 아니라, 지표 기반 의사결정에 필요한 정량 근거와 통제 가능한 개선 수단을 제공하는 것이다.

## Operating Model

의사결정 AI Agent는 사용자 질문을 이해하고, 하위 분석 계획을 세우며, 시멘틱 레이어에서 검증된 지표와 차원을 사용해 정량 근거를 확보한다. 이후 KPI의 driver를 탐색하고, 조직이 통제 가능한 lever를 중심으로 개선안을 구성한다.

## KPI-Driver-Lever

- KPI: 개선하거나 감시하려는 핵심 지표.
- Driver: KPI에 영향을 주는 직간접 요인.
- Lever: 조직이 실제로 개입할 수 있는 통제 가능한 변수 또는 실행 수단.

## Design Constraints

- 모르는 지표나 정의되지 않은 차원은 추정해 답하지 않아야 한다.
- 시멘틱 레이어에 없는 분석은 fallback context layer를 쓰더라도 불확실성을 고지해야 한다.
- 정형 데이터에는 의사결정 당시의 맥락이 빠져 있을 수 있으므로 decision log가 필요하다.
- 최종 개선안은 데이터상 패턴과 현업 맥락을 함께 검증해야 한다.

## Source Notes

- [[2026-05-10-decision-ai-agent-kpi]]: Semantic MCP와 KPI-Driver-Lever 기반 의사결정 Agent 설계 웨비나 ingest.
