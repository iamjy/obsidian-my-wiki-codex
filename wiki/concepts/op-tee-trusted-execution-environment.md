---
title: "OP-TEE Trusted Execution Environment"
date: 2026-05-22
tags:
  - op-tee
  - trusted-execution-environment
  - trustzone
  - embedded-security
type: concept
status: active
source: "wiki/ingests/2026-05-22-op-tee.md"
related:
  - "[[2026-05-22-op-tee]]"
  - "[[nvidia-jetson-secure-boot-chain]]"
  - "[[embedded-platform-cpu-selection]]"
  - "[[platform-enabling-environment]]"
---

# OP-TEE Trusted Execution Environment

## Definition

OP-TEE Trusted Execution Environment는 Arm A-Profile 시스템에서 TrustZone을 사용해 비보안 Linux와 분리된 보안 실행 환경을 제공하는 플랫폼이다. 일반 Linux 환경은 클라이언트 역할을 하고, 보안 영역에서는 Trusted Application이 실행된다.

## Architecture Role

OP-TEE는 하드웨어 TrustZone 기능, 비보안 Linux 커널, TEE 런타임, Trusted Application API가 결합된 보안 실행 기반이다. 이 구조에서 보안 민감 연산은 일반 애플리케이션 프로세스나 커널 공간에만 의존하지 않고 TEE 내부로 분리될 수 있다.

## API Surface

- TEE Internal Core API: Trusted Application이 TEE 내부 기능을 사용하기 위한 GlobalPlatform API.
- TEE Client API: 비보안 실행 환경의 클라이언트가 TEE와 통신하기 위한 GlobalPlatform API.

## Design Implications

OP-TEE 적용은 소프트웨어 패키지 선택만의 문제가 아니다. CPU가 TrustZone을 지원해야 하고, 부트 체인과 펌웨어가 Secure World를 초기화해야 하며, Linux 쪽 드라이버와 클라이언트 라이브러리, Trusted Application 개발 환경이 함께 준비되어야 한다. 따라서 OP-TEE는 [[platform-enabling-environment|플랫폼으로서의 활성화 환경]]처럼 여러 참여 코드와 도구가 올라가는 보안 실행 기반으로 이해할 수 있다.

## Related Patterns

[[embedded-platform-cpu-selection|임베디드 플랫폼 CPU 선정]]에서 CPU Core와 Peripherals가 소프트웨어 구조를 제한하듯, OP-TEE도 TrustZone 지원 여부와 Arm A-Profile 환경에 의해 사용 가능성이 결정된다. 보안 실행 환경이 제품 요구사항에 들어간다면 CPU, 펌웨어, OS, API, 개발 도구를 함께 검토해야 한다.

[[nvidia-jetson-secure-boot-chain|NVIDIA Jetson Secure Boot Chain]]은 OP-TEE가 독립 실행 계층이 아니라 Secure Boot, OEM 키, EKB, UEFI 변수 보호 같은 부트·키 관리 흐름과 함께 배치될 수 있음을 보여준다. TEE 통합은 부팅 신뢰 체인과 키 파생 경계를 함께 설계해야 한다.

## Source Notes

- [[2026-05-22-op-tee]]: OP-TEE를 Arm TrustZone 기반 TEE로 소개하고 GlobalPlatform TEE API 호환성을 정리한 프로젝트 페이지 ingest.
- [[2026-05-22-nvidia-jetson-secure-boot]]: Jetson Secure Boot 문서에서 EKB와 OEM 키가 OP-TEE 관련 보안 기능과 연결되는 지점을 정리한 ingest.
