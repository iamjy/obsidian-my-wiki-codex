---
title: "보안 부팅: NVIDIA Jetson Linux 개발자 가이드"
date: 2026-05-22
tags:
  - clippings
  - nvidia-jetson
  - secure-boot
  - embedded-security
  - uefi
  - fuse
type: ingest
status: stable
source: "raw/보안 부팅 — NVIDIA Jetson Linux 개발자 가이드.md"
related:
  - "[[nvidia-jetson-secure-boot-chain]]"
  - "[[op-tee-trusted-execution-environment]]"
  - "[[embedded-platform-cpu-selection]]"
  - "[[platform-enabling-environment]]"
  - "[[index]]"
---

# 보안 부팅: NVIDIA Jetson Linux 개발자 가이드

## Source

- 원문 제목: `보안 부팅 — NVIDIA Jetson Linux 개발자 가이드`
- 원문 URL: https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Security/SecureBoot.html#prerequisites-secure-boot
- 수집일: 2026-05-22
- 대상 문서: NVIDIA Jetson Linux Developer Guide r38.4 Secure Boot

## Processed Summary

이 문서는 NVIDIA Jetson Linux에서 Secure Boot를 구성하는 절차와 보안 부팅 체인을 설명한다. Secure Boot의 목적은 신뢰 체인을 통해 승인되지 않은 부팅 코드 실행을 막는 것이다. 신뢰 루트는 SoC에 내장된 BootROM이며, BootROM은 쓰기 전용 퓨즈에 저장된 공개 키 암호화(PKC) 관련 값을 사용해 BCT, 부트로더, 웜 부팅 벡터 같은 부팅 코드를 인증한다. SBK를 지원하는 Jetson 플랫폼에서는 부트로더 이미지 암호화도 함께 사용할 수 있다.

핵심 개념은 [[nvidia-jetson-secure-boot-chain|NVIDIA Jetson Secure Boot Chain]]이다. Jetson의 하드웨어 퓨즈 기반 신뢰 루트는 부트로더까지 이어지고, 이후 현재 부트로더인 UEFI가 UEFI 보안 키 체계로 페이로드를 인증한다. 즉 Jetson 보안 부팅은 BootROM/퓨즈/PKC/SBK의 하드웨어 기반 검증과 UEFI Secure Boot의 키 체계가 이어지는 다단계 체인이다.

운영 절차는 대략 PKC 키 쌍 생성, SBK 준비, SoC별 OEM 키 준비, EKB 준비, 퓨즈 구성 XML 작성, `fskp_fuseburn.py`를 통한 퓨즈 소손, `l4t_initrd_flash.sh`를 통한 보안 이미지 플래싱으로 이어진다. Jetson Thor는 최대 16개 OEM PKC 키를 지원하고 `PublicKeyHash` 퓨즈에는 16개 PKC 해시 전체의 해시값이 들어간다. 처음 15개 PKC 키는 취소 가능하며, 이미지 서명에 사용할 키는 PKC 키 목록의 active index로 관리된다. Jetson Orin은 RSA-3K, ECDSA P-256, ECDSA P-521 기반 예시를 제공하고, `tegrasign_v3.py`로 `PublicKeyHash`를 생성한다.

퓨즈는 보안상 되돌릴 수 없는 운영 경계다. 퓨즈 비트가 1로 설정되면 0으로 되돌릴 수 없고, `SecurityMode` 또는 `odm_production_mode`가 0x1로 소손된 뒤에는 일반적인 추가 퓨즈 쓰기가 차단된다. 문서는 퓨즈 구성 XML의 순서도 중요하다고 경고한다. `fskp_fuseburn.py`는 퓨즈 간 의존성을 확인하지 않으므로, 의존 퓨즈를 선행 퓨즈보다 먼저 소손하면 장치가 작동 불능 상태가 될 수 있다.

문서는 Secure Boot 외에도 UEFI Secure Boot, UEFI 페이로드 암호화, UEFI 변수 보호, 플랫폼 벤더 키, 커널 모듈 서명까지 이어지는 보안 기능을 다룬다. EKB는 OP-TEE 관련 보안 흐름과 연결되며, Thor의 `PscOemKdk1` 또는 Orin의 `OemK1/OemK2` 같은 OEM 키가 소손된 경우 UEFI 변수 인증 키를 EKB에 설정해야 한다. 이 지점은 [[op-tee-trusted-execution-environment|OP-TEE Trusted Execution Environment]]와 같은 보안 실행 환경이 부트 체인, 키 파생, EKB, UEFI 인증과 함께 설계되어야 함을 보여준다.

## Key Claims

- Jetson Secure Boot는 BootROM을 신뢰 루트로 삼아 퓨즈에 저장된 PKC 관련 값으로 부팅 코드를 인증한다.
- 하드웨어 퓨즈 기반 신뢰 체인은 부트로더에서 끝나고, 이후 UEFI가 UEFI 보안 키 체계로 페이로드를 인증한다.
- 보안 부팅 구성에는 PKC 키, SBK, SoC별 OEM 키, EKB, 퓨즈 구성 XML, 퓨즈 소손, 보안 이미지 플래싱이 함께 필요하다.
- 퓨즈 비트는 되돌릴 수 없으며, `SecurityMode`가 활성화되면 추가 퓨즈 쓰기가 제한된다.
- `fskp_fuseburn.py`는 퓨즈 의존성을 검증하지 않으므로 XML의 퓨즈 소손 순서가 장치 안정성에 직접 영향을 준다.
- Jetson Thor는 최대 16개 PKC 키와 키 취소 정책을 지원하며, `PublicKeyHash`는 전체 PKC 키 목록의 해시로 구성된다.
- Jetson Orin은 RSA-3K, ECDSA P-256, ECDSA P-521 키 기반 보안 부팅 예시를 제공하고 RSA-2K는 지원하지 않는다.
- SBK는 부트로더 구성요소 암호화에 사용되며 PKC와 함께 사용하는 SBKPKC 모드로 설명된다.
- OEM KDK/K1/K2 키와 EKB는 secure storage, fTPM, UEFI 변수 인증 같은 추가 보안 기능과 연결된다.
- 키 파일 보안과 HSM 기반 난수 생성은 장치 보안의 전제 조건으로 반복 강조된다.

## Entities

- BootROM: SoC 내부에 내장된 최초 신뢰 루트 코드.
- PKC: 부팅 코드 인증에 쓰이는 공개 키 암호화 키 체계.
- SBK: 부트로더 구성요소 암호화에 쓰이는 Secure Boot Key.
- `PublicKeyHash`: PKC 공개 키 또는 키 목록 해시를 저장하는 퓨즈 값.
- `SecurityMode`: 활성화 후 추가 퓨즈 쓰기를 제한하는 생산 보안 모드 관련 퓨즈.
- `fskp_fuseburn.py`: 퓨즈 구성 XML을 사용해 Jetson SoC 퓨즈를 소손하는 도구.
- `l4t_initrd_flash.sh`: 보안 이미지 서명 및 플래싱 흐름에 사용되는 Jetson Linux 스크립트.
- EKB: OEM 키에서 파생된 보안 기능과 UEFI 변수 인증 키를 담는 Encrypted Key Blob.
- UEFI Secure Boot: 부트로더 이후 페이로드 인증을 담당하는 UEFI 키 기반 보안 부팅 체계.
- Jetson Thor: 16개 PKC 키 목록, 취소 정책, KDK 키 흐름이 강조되는 Jetson 계열.
- Jetson Orin: RSA-3K, ECDSA P-256, ECDSA P-521, OemK1/OemK2, RPMB 예시가 제공되는 Jetson 계열.

## Open Questions

- 실제 제품별로 어떤 퓨즈 값을 소손해야 하는지는 Jetson Thor/Orin 세부 모듈의 Fuse Specification과 보안 요구사항에 따라 별도 검토가 필요하다.
- Secure Boot, UEFI Secure Boot, 페이로드 암호화, secure storage, fTPM, 커널 모듈 서명을 어느 조합으로 적용할지는 제품 위협 모델에 따라 달라진다.
- 퓨즈 소손은 되돌릴 수 없으므로 개발 보드, 생산 보드, 키 관리 절차, HSM 사용 여부, 키 백업 정책을 분리해 운영해야 한다.
- 원문은 r38.4 문서이므로 다른 Jetson Linux 릴리스에서는 도구 옵션, 지원 키 타입, 보안 아티팩트 이름이 달라질 수 있다.

## Links

- [[nvidia-jetson-secure-boot-chain]]
- [[op-tee-trusted-execution-environment]]
- [[embedded-platform-cpu-selection]]
- [[platform-enabling-environment]]
- [[index]]
