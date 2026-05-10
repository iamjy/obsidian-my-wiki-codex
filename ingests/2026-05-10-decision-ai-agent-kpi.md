---
title: "질의응답 챗봇을 넘어, KPI를 움직이는 의사결정 AI Agent"
date: 2026-05-10
tags:
  - clippings
  - ai-agent
  - semantic-mcp
  - kpi
  - decision-support
type: ingest
status: stable
source: "raw/질의응답 챗봇을 넘어, KPI를 움직이는 의사결정 AI Agent.md"
related:
  - "[[decision-ai-agent]]"
  - "[[semantic-mcp]]"
  - "[[index]]"
---

# 질의응답 챗봇을 넘어, KPI를 움직이는 의사결정 AI Agent

## Source

- 원문 제목: `질의응답 챗봇을 넘어, KPI를 움직이는 의사결정 AI Agent`
- 원문 URL: https://www.youtube.com/watch?v=Zb3zv-TaMbU
- 발표 주체: HEARTCOUNT
- 게시일: 2026-05-08
- 수집일: 2026-05-10

## Processed Summary

이 웨비나는 AI의 역할이 질의응답 챗봇에서 [[decision-ai-agent|KPI를 움직이는 의사결정 AI Agent]]로 확장되어야 한다고 주장한다. 핵심 문제의식은 LLM이 질문 의도와 분석 계획 수립은 잘하지만, 정형 데이터에서 정확한 정량 근거를 확보하고 비즈니스 맥락에 맞는 개선안을 내는 데는 기존 text-to-SQL 방식만으로 한계가 있다는 점이다.

해결 구조로 제시된 것은 [[semantic-mcp|Semantic MCP]]다. Semantic MCP는 지표 계산식, 차원, 필터, 조인 규칙, KPI별 분석 경로를 AI Agent가 사용할 수 있는 인터페이스로 노출한다. Agent는 직접 SQL을 자유 생성하기보다 시멘틱 레이어에 정의된 지표와 차원으로 pseudo query plan을 만들고, 시멘틱 레이어가 이를 SQL로 컴파일해 실행한다.

하트카운트의 고유 접근은 KPI-Driver-Lever 모델이다. KPI는 개선하려는 핵심 지표, Driver는 KPI에 영향을 주는 직간접 요인, Lever는 조직이 통제 가능한 개선 수단이다. 예시 데모에서는 보험 과지급률 KPI를 분석하면서 비급여 항목, 심사자, 병원 등급 같은 driver를 확인하고, 특정 심사자의 peer group comparison을 lever 분석으로 수행한다.

웨비나는 시멘틱 레이어의 구축이 완전 자동화되기 어렵다고 본다. 기존 DW, DDL, BI 대시보드, SQL history, 지표 정의서 등에서 초안을 만들 수 있지만, 최종 승격에는 현업과 데이터 팀의 확인이 필요하다. 에이전트가 모르는 지표를 추정해 답하기보다 사용자에게 확인하거나 fallback context layer를 활용하는 human-in-the-loop 설계가 권장된다.

## Key Claims

- 의사결정 AI Agent는 질문 답변을 넘어 정량 근거와 실행 가능한 개선안을 제공해야 한다.
- 복잡한 정형 데이터 분석에서는 text-to-SQL 단독 접근이 정확도와 보안 측면에서 한계가 있다.
- 시멘틱 레이어는 회사의 업무 언어와 물리 데이터 구조를 연결하는 의미 계층이다.
- Semantic MCP는 시멘틱 레이어의 정의와 실행 기능을 AI Agent가 사용할 수 있는 도구 인터페이스로 제공한다.
- KPI-Driver-Lever 구조는 KPI 분석을 통제 가능한 개선 수단까지 이어주는 경량 온톨로지 역할을 한다.
- 시멘틱 레이어는 반자동으로 초안을 만들 수 있지만, 정확한 운영을 위해 human-in-the-loop 검증이 필요하다.
- 기존 대시보드, SQL, DW, 마트, BI 자산은 폐기 대상이 아니라 의사결정 Agent가 활용할 semantic asset의 원천이다.

## Entities

- HEARTCOUNT: 웨비나 발표 주체이자 ODA/AI 데이터 분석 도구 제공사
- Semantic MCP: 시멘틱 레이어를 AI Agent에 노출하는 MCP 기반 인터페이스
- ODA: 하트카운트의 의사결정 Agent 맥락에서 언급된 제품/접근
- Cube: 오픈소스 시멘틱 레이어 도구 예시
- dbt Labs: text-to-SQL과 시멘틱 레이어 기반 SQL 생성 정확도 비교 실험 출처로 언급
- KPI-Driver-Lever: KPI, 영향 요인, 통제 가능한 개선 수단을 연결하는 분석 구조

## Open Questions

- Semantic MCP가 실제 운영 환경에서 권한, 감사 로그, 데이터 lineage를 어떻게 보장하는지는 추가 자료가 필요하다.
- KPI-Driver-Lever 정의를 누가 소유하고 어떤 변경 승인 절차를 거치는지가 조직별 핵심 설계 문제가 된다.
- Decision log와 context layer를 결합할 때 잘못된 과거 의사결정 맥락이 재사용되는 위험을 어떻게 통제할지 정리해야 한다.

## Links

- [[decision-ai-agent]]
- [[semantic-mcp]]
- [[index]]
