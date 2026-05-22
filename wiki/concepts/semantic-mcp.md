---
title: "Semantic MCP"
date: 2026-05-10
tags:
  - semantic-layer
  - mcp
  - ai-agent
  - analytics
type: concept
status: active
source: "ingests/2026-05-10-decision-ai-agent-kpi.md"
related:
  - "[[2026-05-10-decision-ai-agent-kpi]]"
  - "[[decision-ai-agent]]"
---

# Semantic MCP

## Definition

Semantic MCP는 시멘틱 레이어에 정의된 지표, 차원, 필터, 조인 규칙, 분석 경로를 AI Agent가 사용할 수 있는 MCP 도구 인터페이스로 노출하는 구조다. Agent가 물리 DB 구조를 직접 추론해 SQL을 생성하는 대신, 검증된 의미 계층을 통해 분석 계획과 실행을 수행하게 한다.

## Components

- 지표 정의: KPI와 계산식.
- 차원 정의: 시간, 고객, 지역, 상품, 조직 등 분석 축.
- 필터 정의: Agent가 안전하게 사용할 수 있는 조건.
- 조인 규칙: 테이블 간 연결 경로.
- 분석 경로: KPI별 driver와 lever.
- 실행 계층: pseudo query plan을 SQL로 컴파일하고 결과를 검증하는 로직.

## Why It Matters

Semantic MCP는 text-to-SQL의 자유 생성 위험을 줄이고, Agent가 회사의 업무 언어와 데이터 구조를 일관되게 연결하도록 돕는다. 특히 현업 사용자를 대상으로 할 때는 모르면 틀린 답을 내는 방식보다, 정의된 semantic asset 안에서 답하거나 사용자 확인을 받는 방식이 더 적합하다.

## Operating Pattern

1. Agent가 사용자 질문을 KPI, dimension, filter, driver, lever로 분해한다.
2. Semantic MCP를 통해 사용 가능한 semantic definition을 탐색한다.
3. Agent가 pseudo query plan을 만든다.
4. 시멘틱 레이어가 plan을 SQL로 컴파일하고 실행한다.
5. Agent가 결과를 해석해 개선안과 후속 분석을 제안한다.

## Source Notes

- [[2026-05-10-decision-ai-agent-kpi]]: Semantic MCP가 의사결정 AI Agent의 정량 근거 확보 계층으로 제시된 웨비나 ingest.
