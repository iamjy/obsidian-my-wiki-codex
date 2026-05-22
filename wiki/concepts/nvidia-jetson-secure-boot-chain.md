---
title: "NVIDIA Jetson Secure Boot Chain"
date: 2026-05-22
tags:
  - nvidia-jetson
  - secure-boot
  - chain-of-trust
  - embedded-security
  - uefi
type: concept
status: active
source: "wiki/ingests/2026-05-22-nvidia-jetson-secure-boot.md"
related:
  - "[[2026-05-22-nvidia-jetson-secure-boot]]"
  - "[[op-tee-trusted-execution-environment]]"
  - "[[embedded-platform-cpu-selection]]"
  - "[[platform-enabling-environment]]"
---

# NVIDIA Jetson Secure Boot Chain

## Definition

NVIDIA Jetson Secure Boot Chain은 SoC BootROM과 퓨즈에 저장된 키 자료를 신뢰 루트로 삼아 부팅 코드의 무결성과 출처를 검증하고, 부트로더 이후에는 UEFI Secure Boot 키 체계로 페이로드 인증을 이어가는 다단계 부팅 신뢰 체인이다.

## Chain Structure

- BootROM: SoC 내부의 변경 불가능한 신뢰 루트로 시작한다.
- Fuse-backed keys: `PublicKeyHash`, `SecurityMode`, SBK, OEM 키 같은 퓨즈 값으로 인증과 암호화 정책을 고정한다.
- Boot code authentication: BCT, 부트로더, 웜 부팅 벡터 같은 초기 부팅 코드를 검증한다.
- UEFI handoff: BootROM/퓨즈 기반 신뢰 체인이 부트로더에서 끝난 뒤 UEFI가 자체 Secure Boot 키 체계로 페이로드를 인증한다.
- Extended security: EKB, UEFI 변수 보호, 페이로드 암호화, fTPM, secure storage, 커널 모듈 서명 같은 기능이 추가 보안 경계를 형성한다.

## Operational Risks

퓨즈는 되돌릴 수 없는 운영 경계다. `SecurityMode` 활성화 후에는 추가 퓨즈 쓰기가 제한되고, 잘못된 퓨즈 값이나 잘못된 소손 순서는 장치를 복구 불가능한 상태로 만들 수 있다. 따라서 개발용 키, 생산용 키, HSM 기반 키 생성, 키 보관, 퓨즈 XML 검토, 사전 플래싱 검증을 별도 운영 절차로 분리해야 한다.

## Platform Differences

Jetson Thor는 최대 16개 PKC 키 목록과 키 취소 정책을 중심으로 동작하며, `PublicKeyHash`가 전체 키 목록 해시를 반영한다. Jetson Orin은 RSA-3K, ECDSA P-256, ECDSA P-521 기반 예시와 `OemK1/OemK2`, RPMB, secure storage 흐름이 강조된다. 같은 Jetson Secure Boot라도 SoC 계열별 퓨즈 이름, 키 흐름, 지원 알고리즘이 다르므로 모듈별 Fuse Specification을 기준으로 검토해야 한다.

## Related Patterns

[[op-tee-trusted-execution-environment|OP-TEE Trusted Execution Environment]]와 Jetson Secure Boot는 모두 하드웨어 신뢰 기능, 펌웨어, OS, 키 관리, 개발 도구가 결합되어야 작동한다. [[embedded-platform-cpu-selection|임베디드 플랫폼 CPU 선정]] 단계에서 보안 부팅과 TEE 요구사항이 있다면 Core 성능뿐 아니라 TrustZone, 퓨즈, Secure Boot 도구, UEFI 정책, BSP 지원까지 함께 확인해야 한다.

## Source Notes

- [[2026-05-22-nvidia-jetson-secure-boot]]: Jetson Linux r38.4 Secure Boot 문서에서 BootROM 신뢰 루트, 퓨즈 소손, PKC/SBK/OEM 키, EKB, UEFI Secure Boot 흐름을 정리한 ingest.
