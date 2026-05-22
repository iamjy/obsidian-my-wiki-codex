---
title: "임베디드 플랫폼 CPU 선정"
date: 2026-05-21
tags:
  - embedded-systems
  - cpu
  - mcu
  - mpu
  - rtos
type: concept
status: active
source: "ingests/2026-05-21-embedded-platform-cpu.md"
related:
  - "[[2026-05-21-embedded-platform-cpu]]"
  - "[[automotive-risc-v-shift-left-development]]"
  - "[[edge-ai-inference-semiconductors]]"
---

# 임베디드 플랫폼 CPU 선정

## Definition

임베디드 플랫폼 CPU 선정은 제품 요구사항에 맞춰 CPU Core의 연산 능력, CPU Peripherals의 주변장치 통합성, Firmware/RTOS/OS 실행 모델, 개발 툴체인과 비용을 함께 고르는 설계 판단이다. CPU는 단순 부품이 아니라 하드웨어 구조와 소프트웨어 구조를 동시에 제한하는 플랫폼 기준점이다.

## Selection Axes

- Core 역량: bit 폭, 명령어 아키텍처, MMU/MPU, 성능, 전력, OS 지원 범위를 확인한다.
- Peripheral 통합성: USB, Ethernet MAC, I/O Controller, 타이머, ADC, 통신 인터페이스처럼 제품에 필요한 제어 회로가 칩 안에 있는지 본다.
- 소프트웨어 실행 모델: 단순 제어는 Firmware, 우선순위와 시간 결정성이 필요하면 RTOS, 복잡한 자원 공유와 고수준 애플리케이션이 필요하면 일반 OS를 검토한다.
- 개발환경: 컴파일러, 디버거, 보드 지원 패키지, 드라이버, 예제 코드, 커뮤니티 또는 벤더 지원을 일정 리스크로 본다.
- 비용과 회로 복잡도: 외부 칩과 별도 회로를 줄일 수 있는 Peripheral 통합성은 BOM, PCB, 검증 비용에 영향을 준다.

## MPU And MCU

MPU는 CPU Core 중심의 연산 처리 능력이 강조되는 선택지이고, MCU는 CPU Peripheral과 제어 회로 통합성이 강조되는 선택지다. 실제 시장에서는 용어가 혼용되므로 이름보다 데이터시트의 Core, 메모리 구조, Peripheral, MMU/MPU, OS 지원 여부가 더 중요하다.

## Software Implication

임베디드 소프트웨어는 추상 애플리케이션 로직뿐 아니라 레지스터 설정을 통해 주변장치를 제어하는 실행 코드다. 주변장치가 자체 레지스터를 제공하면 해당 레지스터를 설정하고, 그렇지 않으면 I/O Controller 같은 CPU Peripheral의 레지스터를 통해 제어한다. 그래서 CPU 선정은 드라이버 작성 방식, 인터럽트 처리, Task 스케줄링, 디버깅 방식까지 영향을 준다.

## Related Patterns

[[automotive-risc-v-shift-left-development|차량용 RISC-V 시프트-레프트 개발]]은 CPU 아키텍처 선택이 툴체인, 드라이버, RTOS/AUTOSAR 포팅, 검증 일정까지 확장된다는 점을 보여준다. [[edge-ai-inference-semiconductors|엣지 AI 추론 반도체]]도 마찬가지로 칩 스펙만이 아니라 실제 소프트웨어 스택과 개발 생태계가 채택 가능성을 좌우한다.

## Source Notes

- [[2026-05-21-embedded-platform-cpu]]: 임베디드 플랫폼을 CPU, 소프트웨어, 개발환경의 조합으로 보고 CPU Core와 Peripherals 기준의 선정 관점을 정리한 글 ingest.
