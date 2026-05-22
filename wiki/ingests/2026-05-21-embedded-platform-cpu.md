---
title: "임베디드 플랫폼: CPU"
date: 2026-05-21
tags:
  - clippings
  - embedded-systems
  - cpu
  - mcu
  - rtos
type: ingest
status: stable
source: "raw/01 임베디드 플랫폼  CPU.md"
related:
  - "[[embedded-platform-cpu-selection]]"
  - "[[automotive-risc-v-shift-left-development]]"
  - "[[edge-ai-inference-semiconductors]]"
  - "[[index]]"
---

# 임베디드 플랫폼: CPU

## Source

- 원문 제목: `01 임베디드 플랫폼 : CPU`
- 원문 URL: https://devsophia.tistory.com/entry/01-%EC%9E%84%EB%B2%A0%EB%94%94%EB%93%9C-%ED%94%8C%EB%9E%AB%ED%8F%BC-CPU
- 작성자: devsophia
- 발행일: 2019-05-03
- 수집일: 2026-05-21
- 참고 문헌: 유명환, `Fun + Fun 뻔뻔하게 배우는 임베디드 리눅스`, 지앤선, 2010년

## Processed Summary

이 글은 임베디드 플랫폼을 하드웨어(CPU), 소프트웨어(Firmware 또는 OS), 개발환경(Tool)의 조합으로 정의한다. 플랫폼 선택은 기능만이 아니라 비용 중심의 균형 문제이며, 특히 CPU 선택은 어떤 소프트웨어 실행 모델을 사용할지에 직접 영향을 준다. Task 수가 적고 자원 공유나 우선순위 요구가 약하면 Firmware가 적합하고, Task 수는 적지만 우선순위 보장이 필요하면 RTOS가 필요하다. 같은 자원에 여러 Task가 접근해야 하는 경우에는 일반 OS 또는 Non-RTOS 계열의 운영체제 선택지가 등장한다.

핵심 개념은 [[embedded-platform-cpu-selection|임베디드 플랫폼 CPU 선정]]이다. CPU는 CPU Core와 CPU Peripherals로 구성되며, Core는 연산 능력과 소프트웨어 실행 범위를 결정하고 Peripherals는 USB, Ethernet MAC 같은 외부 장치와 Core 사이의 하드웨어 제어 회로를 제공한다. 하드웨어 관점에서는 필요한 주변장치 컨트롤러가 칩 안에 내장되어 있는지가 비용과 회로 복잡도를 줄인다. 소프트웨어 관점에서는 8/16/32bit 같은 연산 능력과 OS 지원 여부가 중요하며, 예를 들어 임베디드 리눅스는 32bit 연산을 지원하는 Core를 전제로 한다.

글은 MPU와 MCU의 차이도 Core 중심성과 Peripheral 중심성으로 설명한다. MPU는 CPU Core의 연산 처리 능력이 중심인 장치이고, MCU는 주변장치 제어 회로가 CPU 내부에 잘 통합된 장치로 볼 수 있다. 이 구분은 실무 채용 공고 등에서 혼용되기도 하지만, 시스템 설계에서는 연산 중심 제품인지 제어 중심 제품인지 판단하는 기준으로 쓸 수 있다.

임베디드 소프트웨어의 본질은 C 같은 언어로 하드웨어 레지스터 값을 설정하는 실행 코드라는 설명으로 정리된다. 주변장치가 자체 레지스터를 갖고 있으면 그 값을 직접 설정하고, 그렇지 않으면 I/O Controller 같은 CPU Peripheral의 레지스터를 통해 외부 하드웨어를 제어한다. 따라서 임베디드 개발자는 CPU 데이터시트, 메모리 맵, Peripheral 레지스터, 인터럽트, Task 스케줄링 요구를 함께 읽어야 한다.

## Key Claims

- 임베디드 플랫폼은 CPU, 소프트웨어, 개발환경의 조합이며 비용과 요구사항에 맞춰 균형 있게 선택해야 한다.
- CPU 선택은 Firmware, RTOS, 일반 OS 중 어떤 소프트웨어 구조를 채택할지에 영향을 준다.
- Firmware는 Task 수가 적고 자원 공유와 우선순위 요구가 낮은 경우에 적합하다.
- RTOS는 Task 수가 적더라도 우선순위 보장이 필요한 경우에 필요하다.
- 여러 Task가 같은 자원에 접근하는 경우에는 더 일반적인 운영체제 구조와 자원 관리가 필요하다.
- CPU Core는 연산 능력과 OS 지원 범위를 결정하고, CPU Peripherals는 외부 장치 제어 회로를 제공한다.
- MPU는 Core 중심의 연산 처리 능력, MCU는 Peripheral 중심의 제어 통합성이 상대적으로 강조된다.
- 임베디드 소프트웨어는 하드웨어 레지스터 값을 설정해 주변장치를 제어하는 실행 코드로 볼 수 있다.

## Entities

- CPU Core: 명령 실행과 연산 처리 능력을 담당하며 소프트웨어 실행 범위를 결정하는 CPU 구성요소.
- CPU Peripherals: USB Controller, Ethernet MAC Controller, I/O Controller처럼 Core와 주변 하드웨어 사이를 연결하는 제어 회로.
- Firmware: 단순 Task와 낮은 자원 공유 요구에 적합한 직접 제어형 소프트웨어.
- RTOS: Task 우선순위와 시간 결정성이 필요한 임베디드 시스템용 운영체제.
- Non-RTOS: 원문에서 여러 Task의 자원 접근 관리가 필요한 경우의 운영체제 선택지로 언급된 범주.
- MPU: CPU Core의 연산 처리 능력을 중심으로 보는 Micro Processor Unit.
- MCU: CPU Core와 주변장치 제어 회로의 통합성을 중심으로 보는 Micro Controller Unit.

## Open Questions

- 원문의 Firmware, RTOS, Non-RTOS 구분은 입문용 기준이므로 실제 제품에서는 인터럽트 구조, 응답 시간, 메모리 보호, 드라이버 생태계, 인증 요구까지 함께 비교해야 한다.
- "Non-RTOS"라는 표현은 문맥상 Linux 같은 범용 OS를 가리키는 것으로 보이지만, 용어 자체는 추가 정리가 필요하다.
- MPU와 MCU 구분은 시장과 벤더 문서에서 자주 혼용되므로 실제 칩 선정 시에는 데이터시트의 Core, 메모리, Peripheral, MMU/MPU, OS 지원 여부를 직접 확인해야 한다.

## Links

- [[embedded-platform-cpu-selection]]
- [[automotive-risc-v-shift-left-development]]
- [[edge-ai-inference-semiconductors]]
- [[index]]
