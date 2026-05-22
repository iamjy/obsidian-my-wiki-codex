---
title: "보안 부팅 — NVIDIA Jetson Linux 개발자 가이드"
source: "https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Security/SecureBoot.html#prerequisites-secure-boot"
author:
published:
created: 2026-05-22
description:
tags:
  - "clippings"
---
## 보안 부팅

NVIDIA® Jetson™ Linux는 부팅 보안을 제공합니다. 보안 부팅(Secure Boot) <sup><small><font dir="auto">은</font></small></sup> 신뢰 체인을 통해 승인되지 않은 부팅 코드의 실행을 방지합니다. 루트 오브 트러스트는 칩에 내장된 BootROM 코드로, 쓰기 전용 퓨즈 장치에 저장된 공개 키 암호화(PKC) 키를 사용하여 BCT, 부트로더, 웜 부팅 벡터와 같은 부팅 코드를 인증합니다. 보안 부팅 키(SBK)를 지원하는 Jetson 플랫폼에서는 이를 사용하여 부트로더 이미지를 암호화할 수 있습니다.

NVIDIA SoC에는 보안 및 부팅과 관련된 다양한 항목을 제어하는 여러 개의 퓨즈가 포함되어 있습니다.

Jetson BSP 패키지에는 부팅 중에 보안 서비스를 제공하기 위한 프로그램 스크립트/도구 및 지침이 포함되어 있습니다.

NVIDIA SoC의 퓨즈를 사용하여 부팅 코드를 인증하는 신뢰 루트는 부트로더에서 끝납니다. 그 후, 현재의 부트로더(UEFI)는 UEFI의 보안 키 체계를 사용하여 페이로드를 인증합니다.

UEFI 보안 부팅을 활성화하려면 참조하십시오.

> [!note] 메모
> 제트슨 토르 시리즈 주요 내용:
> 
> - 16개의 PKC 키를 지원합니다. `PublicKeyHash` 퓨즈는 16개 PKC 해시값 전체의 해시값입니다.
> - 처음 15개의 PKC 키는 취소할 수 있습니다.
> - 16개의 키 중 어느 것이든 부트 이미지에 서명하는 데 사용할 수 있습니다.
> - 인증 방식은 더 이상 `BootSecurityInfo` 퓨즈에 명시되지 않습니다. 이미지 서명 시 서명 키 유형에 따라 결정되어 PCP에 저장됩니다.

## 바이너리 융합 및 서명 전체 흐름

PKC 및 SBK를 사용하는 보안 부팅 프로세스를 위해서는 다음 사항이 필요합니다.

- PKC 키 쌍을 생성합니다.
- SBK 키를 준비하세요.
- Kdk0/Kdk1 키를 준비하세요. (Jetson Thor 시리즈에만 해당)
- K1/K2 키를 준비하세요. (Jetson Orin 시리즈에만 해당)
- EKB를 준비하세요.
- 퓨즈 구성 파일을 준비합니다.
- `fskp_fuseburn.py` 퓨즈 구성 파일을 사용하여 스크립트로 퓨즈를 소손합니다.
- `l4t_initrd_flash.sh` ( 및 옵션 `-u` 을 사용하여) 보안 이미지로 장치를 플래싱합니다 `-v`.

## 필수 조건: 보안 부팅

- `libftdi-dev` USB 디버그 포트 지원을 위해.
- `openssh-server` OpenSSL용 패키지입니다.
- 호스트에 최신 Jetson Linux 버전을 완전히 설치했습니다.
- 젯슨 장치와 호스트를 연결하는 USB 케이블.
- 필요한 경우, Jetson 장치의 디버그 직렬 포트를 호스트에 연결하는 USB 케이블이 필요합니다.

## 퓨즈 및 보안

NVIDIA SoC에는 보안 및 부팅과 관련된 다양한 항목을 제어하는 여러 개의 퓨즈가 포함되어 있습니다. 퓨즈 비트가 1로 설정되면 0으로 되돌릴 수 없습니다. 예를 들어, 퓨즈 값 1(0x01)은 3(0x03) 또는 5(0x05)로 변경할 수 있지만, 비트 0이 이미 1로 프로그래밍되어 있기 때문에 4(0x4)로는 변경할 수 없습니다.

`SecurityMode` 퓨즈 (또는 퓨즈 `odm_production_mode`)가 0x1 값으로 소손된 후에 는 모든 추가 퓨즈 쓰기 요청이 차단됩니다.

하지만 일부 ODM 퓨즈는 여전히 쓰기 가능합니다. 자세한 내용은 해당 SoC 퓨즈 관련 문서를 참조하십시오.

`fskp_fuseburn.py` 퓨즈를 태우려면 퓨즈 설정 파일과 함께 스크립트를 사용할 수 있습니다.

퓨즈 설정 파일은 기록될 퓨즈 데이터를 포함하는 XML 파일입니다.

## 퓨즈 구성 파일

퓨즈 구성 파일은 XML 파일이며, 퓨즈 데이터, 퓨즈 목록 및 각 퓨즈에 기록될 값을 포함합니다.

이 `fskp_fuseburn.py` 도구는 이 XML 파일을 사용하여 퓨즈를 프로그래밍합니다.

퓨즈 구성 파일에는 태그 쌍이 포함되어 있으며, 각 태그 쌍에는 연소될 각 퓨즈에 대한 태그가 하나씩 있습니다.`<genericfuse> </genericfuse>` `<fuse/>`

다음 템플릿은 파일 형식을 보여줍니다.

```
<genericfuse MagicId="0x45535546" version="1.0.0">
    <fuse name="<name>" size="<size>" value="<value>"/>
    <fuse name="<name>" size="<size>" value="<value>"/>
    . . .
</genericfuse>

 - \`\`<name>\`\` is the name of a fuse. Supported fuse names are listed in the SoC's Reference Fuse Configuration File. For the Jetson Orin SoC, refer to :ref:\`secure-boot.jetson-orin-reference-fuse-configuration-file\`.
 - \`\`<size>\`\` is the size of the fuse in bytes.
 - \`\`<value>\`\` is the value to be burned into the fuse, with two hexadecimal digits per byte.
```

해당 `MagicId` 값은 `0x45535546` 대상 바이너리에서 사용되므로 변경해서는 안 됩니다.

이 `fskp_fuseburn.py` 스크립트는 퓨즈 설정 파일에 나타나는 순서대로 퓨즈를 태웁니다. 두 개 이상의 퓨즈 값이 서로 의존적인 경우, 독립적인 퓨즈를 의존적인 퓨즈 *보다 먼저* 지정해야 독립적인 퓨즈가 먼저 태워집니다. 즉, 퓨즈 Y에 기록할 수 있는 값이 퓨즈 X의 값에 따라 달라지는 경우, 퓨즈 설정 파일에 퓨즈 X를 먼저 지정한 다음 Y를 지정해야 합니다. 이렇게 하면 스크립트는 `fskp_fuseburn.py` 퓨즈 X를 먼저 태웁니다.

> [!caution] 주의
> 퓨즈 `fskp_fuseburn.py` 소각 도구는 종속성을 확인하지 않으므로, 종속 퓨즈를 해당 퓨즈가 의존하는 퓨즈보다 먼저 지정하면 대상 장치가 작동 불능 상태가 될 수 있습니다. 퓨즈를 소각하기 전에 퓨즈 목록의 순서를 주의 깊게 확인하십시오.

> [!note] 메모
> 퓨즈 설정 파일에는 XML 태그가 포함되어 있지만, XML 표준에서 정의하는 프롤로그는 필요하지 않습니다 . 퓨즈 설정 파일에는 프롤로그가 없을 수도 있습니다. 이러한 파일에서 일반적인 XML 유틸리티를 실행하려면 프롤로그를 추가해야 할 수도 있습니다.`<?xml... ?>`

각 SoC는 고유한 퓨즈와 퓨즈 이름을 가지고 있습니다.

[각 SoC의 퓨즈 및 퓨즈 이름에 대한 자세한 내용은 Jetson 다운로드 센터](https://developer.nvidia.com/embedded/downloads#?search=fuse) 에서 해당 사양 (‘퓨즈’로 필터링)을 참조하십시오.

다음 섹션에서는 각 SoC에 대한 Fuse 구성 파일에 대해 설명합니다.

- Jetson Thor SoC에 대한 자세한 내용은 참조하십시오.
- Jetson Orin SoC에 대한 자세한 내용은 참조하십시오.

### Jetson Thor 퓨즈 구성 파일

퓨즈를 프로그래밍하려면 다음 단계를 따르십시오.

1. PKC 키를 사용하여 이미지 서명을 활성화합니다.
	1. PKC 키 목록을 생성합니다. 자세한 내용은 를 참조하십시오.
		2. PKC 키 목록에서 값을 생성합니다 `PublicKeyHash`. 자세한 내용은 를 참조하십시오.
2. SBK 키를 생성하세요. 자세한 내용은 참조하세요.
3. OEM 프로그래밍 가능 키를 생성합니다. 자세한 내용은 참조하십시오.
4. Create a Fuse Configuration file. For more information, refer to.

For more information about fuses and fuse names for the Jetson Thor SoC, refer to the *Jetson Thor Series Modules Fuse Specification* document.

#### Generate a PKC Key List for Jetson Thor

Jetson Thor series support up to 16 OEM PKC keys. The digest of all 16 public key digests are burned to the `PublicKeyHash` fuse. To support this, Jetson Thor devices use a PKC key list stored in form of a xml file that holds the information for up to 16 keys.

The following template shows the format of the PKC key list:

```
<?xml version="1.0"?>
<entry_list>
    <bct active_index="<active_key_id>" chip_id="0x260" pcp_file="nv_combo.pcp" pcps_file="nv_combo.pcps" pcps_hash_file="nv_combo.pcps.hash" />
    <entry hash_file="<public_key_digest_filename>"  key="<private_key_filename>"  key_id="0"  mode="<key_mode>"  pub_file="<public_key_filename>" />
    <entry hash_file="<public_key_digest_filename>"  key="<private_key_filename>"  key_id="1"  mode="<key_mode>"  pub_file="<public_key_filename>" />
    <entry hash_file="<public_key_digest_filename>"  key="<private_key_filename>"  key_id="2"  mode="<key_mode>"  pub_file="<public_key_filename>" />
    <entry hash_file="<public_key_digest_filename>"  key="<private_key_filename>"  key_id="3"  mode="<key_mode>"  pub_file="<public_key_filename>" />
    . . .
    <entry hash_file="<public_key_digest_filename>"  key="<private_key_filename>"  key_id="15" mode="<key_mode>"  pub_file="<public_key_filename>" />
</entry_list>
```

Replace the placeholders as follows:

- `<active_key_id>` is the index of the active key that will be used to sign images.
- `<public_key_digest_filename>` is the name of the output public key digest file.
- `<private_key_filename>` is the full path and name of the input private key file.
- `<public_key_filename>` is the name of the output public key file.
- `<key_mode>` is the mode of the key: `pkc` for RSA-3K keys, `ec` for ECDSA P-256 keys, `ec521` for ECDSA P-521 keys, or `xmss` for XMSS keys.

Default fields in the PKC key list:

- `pcp_file` is the output public cryptography parameter file of the active key.
- `pcps_file` is the output file of the concatenation of all public key digests.
- `pcps_hash_file` is the output digest of the `pcps_file`.

> [!note] Note
> We suggest that you generate all 16 keys. For key generation, refer to.

> [!note] Note
> The value of `<private_key_filename>` must include the full path to the private key file.

#### Examples of Jetson Thor Fuse Configuration Files

##### Example Jetson Thor Fuse Configuration File to Program an RSA-3K Key

Example PKC key list with only one RSA-3K key:

```
<?xml version="1.0"?>
<entry_list>
    <bct active_index="0" chip_id="0x260" pcp_file="nv_combo.pcp" pcps_file="nv_combo.pcps" pcps_hash_file="nv_combo.pcps.hash" />
    <entry hash_file="rsa3k-0.hash"  key="/path/to/rsa3k-0.pem"  key_id="0"  mode="pkc"  pub_file="rsa3k-0.pubkey" />
</entry_list>
```

Example fuse configuration file to enable Secure Boot with an RSA-3K key:

```
<genericfuse MagicId="0x45535546" version="1.0.0">
    <fuse name="PublicKeyHash" size="64" value="0x18e984f7d79f7a185039ec413ed2ff86227c8f0be639edde0cf23ab1f7910b759ede8fb0c20d02c68deb04a75226d632f9fe24c71dad4b302acdba13db658130"/>
    <fuse name="OptInEnable" size="4" value="0x00000001"/>
    <!--  To enable revocation policy, use <fuse name="BootSecurityInfo" size="4" value="0x220"/>  -->
    <fuse name="BootSecurityInfo" size="4" value="0x200"/>
    <fuse name="SecurityMode" size="4" value="0x1"/>
</genericfuse>
```

> [!note] Note
> The value above for `PublicKeyHash` is for demonstrations only. It is generated from the PKC key list shown above. You must prepare a RSA-3K key pair (`.pem` file). If you plan to support more than one PKC key for signing images, you must prepare all the key pairs now so that the `PublicKeyHash` can be generated from all supported PKC public keys. For key generation, refer to.
> 
> For more information about generating the `PublicKeyHash` fuse value, refer to.

##### Example Jetson Thor Fuse Configuration File to Program an ECDSA P-521 Key + SBK Key

Example ECDSA P-521 PKC key list with only one ECDSA P-521 key:

```
<?xml version="1.0"?>
<entry_list>
    <bct active_index="0" chip_id="0x260" pcp_file="nv_combo.pcp" pcps_file="nv_combo.pcps" pcps_hash_file="nv_combo.pcps.hash" />
    <entry hash_file="ecp521-0.hash"  key="/path/to/ecp521-0.pem"  key_id="0"  mode="ec"  pub_file="ecp521-0.pubkey" />
</entry_list>
```

The following sample configuration file is used to enable the Secure Boot with an ECDSA P-521 key and a SBK key:

```
<genericfuse MagicId="0x45535546" version="1.0.0">
    <fuse name="PublicKeyHash" size="64" value="0x9f0ebf0aec1e2bb30c0838096a6d9de5fb86b1277f182acf135b081e345970167a88612b916128984564086129900066255a881948ab83bebf78c7d627f8fe84"/>
    <fuse name="OptInEnable" size="4" value="0x00000001"/>
    <fuse name="PscSecureBootKey" size="32" value="0x123456789abcdef0fedcba987654321000112233445566778899aabbccddeeff"/>
    <fuse name="OespSecureBootKey" size="32" value="0x123456789abcdef0fedcba987654321000112233445566778899aabbccddeeff"/>
    <fuse name="SbSecureBootKey" size="32" value="0x123456789abcdef0fedcba987654321000112233445566778899aabbccddeeff"/>
    <!--  To enable revocation policy, use <fuse name="BootSecurityInfo" size="4" value="0x228"/>  -->
    <fuse name="BootSecurityInfo" size="4" value="0x208"/>
    <fuse name="SecurityMode" size="4" value="0x1"/>
</genericfuse>
```

> [!note] Note
> The values of `PublicKeyHash` and `PscSecureBootKey` above are for demonstrations only. The value of `PublicKeyHash` is generated from the PKC key list shown above. If you plan to support more than one PKC key for signing images, you must prepare all the key pairs now so that the `PublicKeyHash` can be generated from all supported PKC public keys. For key generation, refer to.
> 
> For more information about generating the `PublicKeyHash` fuse value, refer to.
> 
> For more information about generating the `PscSecureBootKey`, `OespSecureBootKey` and `SbSecureBootKey` fuse values, refer to.

##### Example Jetson Thor Fuse Configuration File to Program an ECDSA P-521 Key + SBK Key + OemKdk0 Key + OemKdk1 Key + Enable Revocation Policy + Enable fTPM

The following sample configuration file is used to enable the Secure Boot with an ECDSA P-521 key, a SBK key, an OemKdk0 key, an OemKdk1 key, revocation policy, and fTPM:

```
<genericfuse MagicId="0x45535546" version="1.0.0">
    <fuse name="PublicKeyHash" size="64" value="0xaa7ab9bf57a5747af4af080b86c1b7a651f839141da6ee80ebe3c562a5a8fb34e981d964bee653e93cd56e6b0896753d191ded586a12503fc8797d3d62ac3c08"/>
    <fuse name="OptInEnable" size="4" value="0x00000001"/>
    <fuse name="PscOemKdk0" size="32" value="0x6208e3cd81ed0cd77b214db0c875ade40c26bca09382ad82cd0e24046cc8c64e"/>
    <fuse name="OespOemKdk0" size="32" value="0xac88dead695089ed2aee491b180264873e966a61b609db4977f073aea41b132b"/>
    <fuse name="SbOemKdk0" size="32" value="0x35e174674e5ab8168023b83063886ca252c018bca1015d86cfd7c0d1d09b6659"/>
    <fuse name="PscOemKdk1" size="32" value="0x112233445566778899aabbccddeeff00112233445566778899aabbccddeeff00"/>
    <fuse name="OespOemKdk1" size="32" value="0xae4052f4e52541b238d6c63123e768ef519a3cdc8ff2e7cbd759e03d68a8775e"/>
    <fuse name="SbOemKdk1" size="32" value="0x881b43f0d8985031df039339d4eff6b30513e272633b527513b358195821ee37"/>
    <fuse name="PscSecureBootKey" size="32" value="0x123456789abcdef0fedcba987654321023456789abcdef01edcba9876543210f"/>
    <fuse name="OespSecureBootKey" size="32" value="0x123456789abcdef0fedcba987654321023456789abcdef01edcba9876543210f"/>
    <fuse name="SbSecureBootKey" size="32" value="0x123456789abcdef0fedcba987654321023456789abcdef01edcba9876543210f"/>
    <fuse name="BootSecurityInfo" size="4" value="0x2228"/>
    <fuse name="SecurityMode" size="4" value="0x1"/>
</genericfuse>
```

> [!note] Note
> The values of `PublicKeyHash`, `PscSecureBootKey`, `OemKdk0`, and `OemKdk1` above are for demonstrations only. You must prepare a PKC key pair (`.pem` file) or prepare all PKC key pairs you plan to use. For key generation, refer to.
> 
> For more information about generating the `PublicKeyHash` fuse value, refer to.
> 
> For more information about generating the `PscSecureBootKey`, `OespSecureBootKey` and `SbSecureBootKey` fuse values, refer to.
> 
> For more information about generating the `PscOemKdk0`, `OespOemKdk0` and `SbOemKdk0` fuse values, refer to [Firmware TPM](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Security/FirmwareTPM.html#sd-security-firmwaretpm).
> 
> For more information about generating the `PscOemKdk1`, `OespOemKdk1`, and `SbOemKdk1` fuse values, refer to.

#### Jetson Thor Reference Fuse Configuration File

The Jetson Thor Reference Fuse Configuration file lists all fuses that are supported by the Jetson Thor SoC.

All fuse values in the reference configuration file are enclosed in XML comments. To adapt the reference file for fusing, uncomment them and replace their `0xFFFF` placeholder values with the actual values for your target.

Here is the Reference Fuse Configuration File for Jetson Thor devices:

```
<genericfuse MagicId="0x45535546" version="1.0.0">
    <!-- <fuse name="OdmId" size="8" value="0xFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="OdmInfo" size="4" value="0xFFFF"/> -->
    <!-- <fuse name="ReservedOdm0" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="ReservedOdm1" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="ReservedOdm2" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="ReservedOdm3" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="ReservedOdm4" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="ReservedOdm5" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="ReservedOdm6" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="ReservedOdm7" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="OptInEnable" size="4" value="0x00000001"/> -->
    <!-- <fuse name="PublicKeyHash" size="64" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="PscOemKdk0" size="32" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="OespOemKdk0" size="32" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="SbOemKdk0" size="32" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="PscOemKdk1" size="32" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="OespOemKdk1" size="32" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="SbOemKdk1" size="32" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="PscSecureBootKey" size="32" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="OespSecureBootKey" size="32" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="SbSecureBootKey" size="32" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="BootSecurityInfo" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="SecurityMode" size="4" value="0x1"/> -->
</genericfuse>
```

> [!note] Note
> Although the size of the `OdmInfo` fuse is 4, only the last two bytes are programmable. `OptInEnable` and `SecurityMode` are both 1-bit fuses despite their sizes are 4.

#### Generate a PKC Key Pair for Jetson Thor

Jetson Thor series targets support the PKC of RSA 3K, ECDSA P-256, and ECDSA P-521. **XMSS is currently not supported**.

1. Enter one of the following commands to generate a PKC key pair:
	- To generate an ECDSA P-256 key:
		```
		$ openssl ecparam -name prime256v1 -genkey -noout -out ecp256.pem
		```
		- To generate an ECDSA P-521 key:
		```
		$ openssl ecparam -name secp521r1 -genkey -noout -out ecp521.pem
		```
		- To generate an RSA 3K key:
		```
		$ openssl genrsa -out rsa_priv.pem 3072
		```
		- To generate an XMSS key:
		```
		$ ./bootloader/xmss-sign generate --privkey xmss_v3.key -pubkey xmss_v3.pub
		```
2. Rename and save the key file.
	The key file is used to burn fuses and sign boot files for Jetson devices.

> [!caution] Caution
> Avoid generating and storing the key pair under `bootloader/` directory but a secure location instead.

> [!note] Note
> There will be 3 files generated by `xmss-sign` that need to be saved:
> 
> > - xmss\_v3.key
> > - xmss\_v3.key.cache
> > - xmss\_v3.pub

> [!note] Note
> To generate a truly random number key, use the Hardware Security Module (HSM).

#### Generate PublicKeyHash Value From a PKC Key List for Jetson Thor

Instead of fusing 16 public keys of each key pair, only the digest of all 16 public key digests is fused to the `PublicKeyHash` fuse.

The following steps show how to generate the `PublicKeyHash` value from a PKC key list:

1. Generate a PKC key list. For more information, refer to.
2. Run the following command to generate the `PublicKeyHash` value:
	```
	$ sudo ./tegrasign_v3.py --key <pkc_key_list> --pubkeyhash <active_pkc.pcp> <pkc_key_list.hash>
	```

Where:

> - `<pkc_key_list>` is the name of the PKC key list file generated in the previous step.
> - `<active_pkc.pcp>` is the name of the output public cryptography parameter file of the active key.
> - `<pkc_key_list.hash>` is the name of the output public key digest of all the public keys in the PKC key list.

The hexadecimal value shown on the screen after tegra-fuse format (big-endian): can be used directly as the `PublicKeyHash` fuse data of a Fuse Configuration file.

Here are some sample outputs after running `tegrasign_v3.py` to generate `PublicKeyHash` for a key list with 16 keys:

```bash
$ sudo ./tegrasign_v3.py --key key_list.xml --pubkeyhash pub_key.pcp pcps.hash
Key size is 384 bytes
Key size is 384 bytes
Key size is 384 bytes
Key size is 384 bytes
Valid ECC key
Valid ECC key
Valid ECC key
Valid ECC key
Valid ECC key. Key size is 521
Valid ECC key. Key size is 521
Valid ECC key. Key size is 521
Valid ECC key. Key size is 521
WARNING: Can't create EVPKey object from ECKey object
WARNING: Can't create EVPKey object from ECKey object
Warning: Can't create EVPKey object from EDKey object
Warning: xmss_v3_0.key is not valid ed25519 key in Open SSL format
Warning: Can not extract key from xmss_v3_0.key
Assuming XMSS key
WARNING: Can't create EVPKey object from ECKey object
WARNING: Can't create EVPKey object from ECKey object
Warning: Can't create EVPKey object from EDKey object
Warning: xmss_v3_1.key is not valid ed25519 key in Open SSL format
Warning: Can not extract key from xmss_v3_1.key
Assuming XMSS key
WARNING: Can't create EVPKey object from ECKey object
WARNING: Can't create EVPKey object from ECKey object
Warning: Can't create EVPKey object from EDKey object
Warning: xmss_v3_2.key is not valid ed25519 key in Open SSL format
Warning: Can not extract key from /xmss_v3_2.key
Assuming XMSS key
WARNING: Can't create EVPKey object from ECKey object
WARNING: Can't create EVPKey object from ECKey object
Warning: Can't create EVPKey object from EDKey object
Warning: xmss_v3_3.key is not valid ed25519 key in Open SSL format
Warning: Can not extract key from xmss_v3_3.key
Assuming XMSS key
Saving public key in pub_key.key for LIST
pub_key.key is produced from copying nv_combo.pcp
pcps.hash is produced from copying nv_combo.pcps.hash
tegra-fuse format (big-endian): 0xaa7ab9bf57a5747af4af080b86c1b7a651f839141da6ee80ebe3c562a5a8fb34e981d964bee653e93cd56e6b0896753d191ded586a12503fc8797d3d62ac3c08
```

> [!note] Note
> The actual output may vary from the examples shown here based on the BSP version and the key type.

### Jetson Orin Fuse Configuration File

For more information about fuses and fuse names for the Jetson Orin SoC, refer to *Jetson Orin Fuse Specification (Covers Jetson AGX Orin Series, Jetson Orin NX Series, and Jetson Orin Nano Series modules)*.

#### Examples of Jetson Orin Fuse Configuration Files

##### Example Jetson Orin Fuse Configuration File to Program an RSA-3K Key

Example fuse configuration file to enable Secure Boot with an RSA-3K key:

```
<genericfuse MagicId="0x45535546" version="1.0.0">
    <fuse name="PublicKeyHash" size="64" value="0x18e984f7d79f7a185039ec413ed2ff86227c8f0be639edde0cf23ab1f7910b759ede8fb0c20d02c68deb04a75226d632f9fe24c71dad4b302acdba13db658130"/>
    <fuse name="BootSecurityInfo" size="4" value="0x1"/>
    <fuse name="SecurityMode" size="4" value="0x1"/>
</genericfuse>
```

> [!note] Note
> The preceding value for `PublicKeyHash` is for demonstration only. You must prepare a PKC key pair (`.pem` file). For key generation, refer to.
> 
> For more information about generating the `PublicKeyHash` fuse value, refer to.

##### Example Jetson Orin Fuse Configuration File to Program an ECDSA P-256 Key

Example fuse configuration file to enable Secure Boot with an ECDSA P-256 key:

```
<genericfuse MagicId="0x45535546" version="1.0.0">
    <fuse name="PublicKeyHash" size="64" value="0x3c67c6446176bab0a35c09fa77c77c14f2c690dad4f5afcbc6a5ac3c39a0231e192eea1aab469e086ffd42eded658d2317583d6b39bedb2e2ca3c5d0d09bcbea"/>
    <fuse name="BootSecurityInfo" size="4" value="0x2"/>
    <fuse name="SecurityMode" size="4" value="0x1"/>
</genericfuse>
```

> [!note] Note
> The preceding value for `PublicKeyHash` is for demonstration only. You must prepare a PKC key pair (`.pem` file). For key generation, refer to.
> 
> For more information about generating the `PublicKeyHash` fuse value, refer to.

##### Example Jetson Orin Fuse Configuration File to Program an ECDSA P-521 Key + SBK Key + OemK1 Key

The following sample configuration file is used to enable the Secure Boot with an ECDSA P-521 key, an SBK key, and an OemK1 key:

```
<genericfuse MagicId="0x45535546" version="1.0.0">
    <fuse name="PublicKeyHash" size="64" value="0x9f0ebf0aec1e2bb30c0838096a6d9de5fb86b1277f182acf135b081e345970167a88612b916128984564086129900066255a881948ab83bebf78c7d627f8fe84"/>
    <fuse name="SecureBootKey" size="32" value="0x123456789abcdef0fedcba987654321000112233445566778899aabbccddeeff"/>
    <fuse name="OemK1" size="32" value="0xf3bedbff9cea44c05b08124e8242a71ec1871d55ef4841eb4e59a56b5f88fb2b"/>
    <fuse name="BootSecurityInfo" size="4" value="0x20b"/>
    <fuse name="SecurityMode" size="4" value="0x1"/>
</genericfuse>
```

> [!note] Note
> The preceding values of `PublicKeyHash`, `SecureBootKey`, and `OemK1` are for demonstration only. You must prepare a PKC key pair (`.pem` file). For key generation, refer to.
> 
> For more information about generating the `PublicKeyHash` fuse value, refer to.
> 
> For more information about generating the `SecureBootKey` fuse value, refer to.
> 
> For more information about generating the `OemK1` fuse value, refer to.

##### Example Jetson Orin Fuse Configuration File to Program an OemK1 Key + RPMB Key

Use the following sample configuration file to enable [Secure Storage](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Security/SecureStorage.html#sd-security-securestorage) with an OemK1 key and RPMB key:

```
<genericfuse MagicId="0x45535546" version="2.0.0">
    <fuse name="OemK1" size="32" value="0xf3bedbff9cea44c05b08124e8242a71ec1871d55ef4841eb4e59a56b5f88fb2b"/>
    <fuse name="BootSecurityInfo" size="4" value="0x200"/>
    <rpmb provisioning="0x1" dev_type="0x2"/>
</genericfuse>
```

> [!note] Note
> The value `OemK1` is for demonstration purposes only. For more information about generating the `OemK1` fuse value, refer to.
> 
> To enable RPMB provisioning, the tool supports fusing the RPMB key onto eMMC storage. The version field must be 2.0.0 for RPMB. This requires the `OemK1` key to be present in the fuse configuration file or already burned.

#### Jetson Orin Reference Fuse Configuration File

The Jetson Orin Reference Fuse Configuration file lists all fuses that are supported by the Jetson Orin SoC.

All fuse values in the reference configuration file are enclosed in XML comments. To adapt the reference file for fusing, uncomment them and replace their `0xFFFF` placeholder values with the actual values for your target.

Here is the Reference Fuse Configuration File for Jetson Orin devices:

```
<genericfuse MagicId="0x45535546" version="1.0.0">
    <!-- <fuse name="OdmId" size="8" value="0xFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="OdmInfo" size="4" value="0xFFFF"/> -->
    <!-- <fuse name="ArmJtagDisable" size="4" value="0x1"/> -->
    <!-- <fuse name="ReservedOdm0" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="ReservedOdm1" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="ReservedOdm2" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="ReservedOdm3" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="ReservedOdm4" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="ReservedOdm5" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="ReservedOdm6" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="ReservedOdm7" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="OptInEnable" size="4" value="0x1"/> -->
    <!-- <fuse name="PublicKeyHash" size="64" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="PkcPubkeyHash1" size="64" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="PkcPubkeyHash2" size="64" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="SecureBootKey" size="32" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="Kdk0" size="32" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="PscOdmStatic" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="OemK1" size="32" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="OemK2" size="32" value="0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"/> -->
    <!-- <fuse name="BootSecurityInfo" size="4" value="0xFFFFFFFF"/> -->
    <!-- <fuse name="SecurityMode" size="4" value="0x1"/> -->
</genericfuse>
```

> [!note] Note
> Although the size of the `OdmInfo` fuse is 4, only the last two bytes are programmable.

#### Generate a PKC Key Pair for Jetson Orin

Jetson Orin series targets support the PKC of RSA 3K, ECDSA P-256, and ECDSA P-521.

> [!note] Note
> The 2048-bit RSA key option is no longer supported on Jetson Orin series.

1. Enter one of the following commands to generate a PKC key pair:
	- To generate an ECDSA P-256 key:
		```
		$ openssl ecparam -name prime256v1 -genkey -noout -out ecp256.pem
		```
		- To generate an ECDSA P-521 key:
		```
		$ openssl ecparam -name secp521r1 -genkey -noout -out ecp521.pem
		```
		- To generate an RSA 3K key:
		```
		$ openssl genrsa -out rsa_priv.pem 3072
		```
2. Rename and save the key file.
	The key file is used to burn fuses and sign boot files for Jetson devices.

> [!caution] Caution
> The security of your device depends on how securely you keep the key file.

> [!note] Note
> To generate a truly random number key, use the Hardware Security Module (HSM).

#### Generate PublicKeyHash Value from a PKC Key Pair for Jetson Orin

Instead of fusing the public key of a PKC key pair, only the hash of the public key is burned to the `PublicKeyHash` fuse field.

To generate the `PublicKeyHash` value, use the `tegrasign_v3.py` program:

```
$ ./tegrasign_v3.py --pubkeyhash <pkc.pubkey> <pkc.hash> --key <pkc.pem>
```

Where:

> - `<pkc.pem>` is the input PKC key pair (.pem file) file.
> - `<pkc.pubkey>` is the output public key of the `<pkc.pem>` key pair
> - `<pkc.hash>` is the output public key hash of the `<pkc.pem>` key pair

The hexadecimal value shown on the screen after `tegra-fuse format (big-endian):` can be used directly as the `PublicKeyHash` fuse data of a Fuse Configuration file.

Here are some sample outputs after running `tegrasign_v3.py` to generate `PublicKeyHash` for an ECDSA P-521 key:

```
$ ./tegrasign_v3.py --pubkeyhash ecp521.pubkey ecp521.hash --key ecp521.pem
  Valid ECC key. Key size is 521
  Valid ECC key. Key size is 521
  Saving public key in ecp521.pubkey for ECC
  Sha saved in pcp.sha
  tegra-fuse format (big-endian): 0x9f0ebf0aec1e2bb30c0838096a6d9de5fb86b1277f182acf135b081e345970167a88612b916128984564086129900066255a881948ab83bebf78c7d627f8fe84
```

Here are some sample outputs after running `tegrasign_v3.py` to generate `PublicKeyHash` for an RSA 3k key:

```
$ ./tegrasign_v3.py --pubkeyhash rsa3k.pubkey rsa3k.hash --key rsa3k.pem
Key size is 384 bytes
Key size is 384 bytes
Saving pkc public key in rsa3k.pubkey
Sha saved in pcp.sha
tegra-fuse format (big-endian): 0xad2474627c14e3f7f4944a832bd15d0640938a3dc162f558692458f3d12f9453e11bea2ec75df3f83e8b29c47fc3d2483d528d3e94a5469c4ba1ec61f1584b23
```

> [!note] Note
> - `tegrasign_v3.py` can only be used to generate `PublicKeyHash` for the Jetson AGX Orin series, the Jetson Orin NX, and the Nano series.
> - RSA2K is not supported on the Jetson AGX Orin series, the Jetson Orin NX, and the Nano series.
> - `tegrakeyhash` has been deprecated. Please use `tegrasign_v3.py` to generate the `PublicKeyHash` now.

## Prepare an SBK key

An SBK key is used to encrypt Bootloader components. The same SBK key has to be fused to the Jetson’s SoC fuses, so the key can be used to decrypt the Bootloader components when the Jetson device boots up.

> [!note] Note
> You can only use the SBK key with the PKC key. The encryption mode that uses these two keys together is called SBKPKC.

Both the Thor and Orin SoCs require an SBK key be of eight 32-bit words (32 bytes).

The SBK key file is stored in big-endian hexadecimal format.

Here is an example of a 32-byte SBK key file:

```
0x12345678 0x9abcdef0 0xfedcba98 0x76543210 0x23456789 0xabcdef01 0xedcba987 0x6543210f
```

This type of file format is used in `l4t_initrd_flash.sh` command with `-v` option.

The same SBK representation used in the “SecureBootKey” fuse value field of a Fuse Configuration XML file is:

```
0x123456789abcdef0fedcba987654321023456789abcdef01edcba9876543210f
```

> [!caution] Caution
> For the Jetson Thor series, the three SBK fuses, `PscSecureBootKey`, `OespSecureBootKey` and `SbSecureBootKey`, must be programmed with the same value.

> [!note] Note
> Hexadecimal numbers must be presented in big-endian format. The leading 0x or 0X can be omitted. The Jetson Secure Boot software converts the big-endian hexadecimal format to the format that the Jetson device expects. All standard OpenSSL utilities output in big-endian format.
> 
> We recommend that you use the Hardware Security Module (HSM) to generate a truly random number for an SBK key.

> [!caution] Caution
> The security of your device depends on how securely you keep the key file.

## Prepare K1/K2/KDK1 Keys

### For the Jetson Thor series

The KDK1 key is the fuse key used for other security applications, such as deriving other usage root keys. The fuse key name is `PscOemKdk1`, and the key length is 32 bytes.

For example, you can use the KDK1 key as the EKB fuse key. (For details, refer to [EKB Generation](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Security/OpTee.html#sd-security-optee-ekb-ekbgeneration).) You must prepare this key and other ODM fuse bits as described in the documentation for the other security applications.

### For the Jetson Orin series

The K1/K2 keys are the fuse keys used for other security applications. The fuse key names are `OemK1` and `OemK2`, and the key length is 32 bytes.

For example, you can use the K1 key as the EKB fuse key. (For details, refer to [EKB Generation](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Security/OpTee.html#sd-security-optee-ekb-ekbgeneration).) You must prepare these keys and other ODM fuse bits as described in the documentation for the other security applications.

### Sample Fuse Key

A key consists of eight 32-bit words stored in a file in the big-endian hexadecimal format.

> [!note] Note
> You can omit the leading `0x` or `0X` of a hexadecimal number can be omitted. The Jetson Secure Boot software converts the big-endian hexadecimal format to the format that the Jetson device expects.

The following is an example of a fuse key file:

```
0x11223344 0x55667788 0x99aabbcc 0xddeeff00 0xffeeddcc 0xbbaa9988 0x77665544 0x33221100
```

The same key representation in the fuse value field in the Fuse Configuration XML file is as follows:

```
0x112233445566778899aabbccddeeff00ffeeddccbbaa99887766554433221100
```

> [!note] Note
> We recommend that you use the HSM to generate a truly random number for K1/K2/KDK1 keys.

> [!caution] Caution
> The security of your device depends on how securely you keep these key files.

## Prepare EKB

When the `PscOemKdk1` fuse (for the Jetson Thor series) or the `OemK1/OemK2` fuse (for the Jetson Orin series) is burned, you must set your own UEFI variable authentication key (the *auth* key) in the EKB. Without the auth key, UEFI fails to authenticate the UEFI variable and fails to boot. For more details, refer to [EKB Generation](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Security/OpTee.html#sd-security-optee-ekb-ekbgeneration).

The UEFI variable authentication key is a 128-bit key stored in a file in the big-endian hexadecimal format.

Here is an example of an `auth` key file:

```
0x00000000000000000000000000000000
```

> [!note] Note
> We recommend that you use the HSM to generate a truly random number for the `auth` key.

### For the Jetson Thor series

When the `PscOemKdk1` fuse is burned, generate your own EKB authenticated and encrypted with keys derived from `PscOemK1`.

The following example shows you how to generate the EKB for the Jetson Thor series:

```
$ python3 gen_ekb.py -chip t264
                     -oem_kdk1_key <oem_kdk1.key> \
                     -in_auth_key <auth_t264.key> \
                     -out <eks_t264.img>
```

### For the Jetson Orin series

When the `OemK1` fuse is burned, generate your own EKB authenticated and encrypted with keys derived from `OemK1`.

The following example shows you how to generate the EKB for the Jetson Orin series:

```
$ python3 gen_ekb.py -chip t234
                     -oem_k1_key <oem_k1.key> \
                     -in_auth_key <auth_t234.key> \
                     -out <eks_t234.img>
```

## Prepare the Fuse Configuration file

To modify the SoC’s Reference Fuse Configuration file, uncomment the fuses you need, and enter information in the correct fuse data fields for your target Jetson device.

The next section provides information about how to burn fuses with the prepared Fuse Configuration file.

> [!caution] Caution
> The security of your device depends on how securely you keep the Fuse Configuration file.

## Burn Fuses with the Fuse Configuration file

After the Fuse Configuration file is prepared, you can burn fuses using `odmfuse.sh` (-X option) script with the Fuse Configuration file:

```
sudo ./odmfuse.sh -X <fuse_config> -i <chip_id> <target_config>
```

If a Jetson board was previously burned with a PKC key <pkc.pem>, and the board needs to have additional fuses burned, run the following `odmfuse.sh` command with -k option:

```
sudo ./odmfuse.sh -X <fuse_config> -i 0x23 -k <pkc.pem> <target_config>
```

Where:

> - `<fuse_config>` is the fuse configuration XML file.
> - `<pkc.pem>` is the PKC key pair (.pem file) that was fused to the board before.
> - `<target_config>` is the name of the configuration for your Jetson device and carrier board; see the table in [Jetson Modules and Configurations](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/IN/QuickStart.html#in-quickstart-jetsonmodulesandconfigurations).

> [!note] Note
> Fuse burning operations are high-risk because they cannot be reversed. NVIDIA strongly recommends that you use the `--test` option to verify fuse burning operations before you perform them.
> 
> When you add `--test` to an `odmfuse.sh` command, the command performs pre-burn processing and verification, but it does not actually burn the fuse. If the command reports the results you want, you can re-enter the command without `--test` and burn the fuse with greater confidence that you are doing it correctly.
> 
> NVIDIA recommends burning all the fuses you need in a single operation. While partial fuse burning is possible if `SecurityMode` is not burned, it may lead to issues not described in this document. If you are determined to proceed with partial fuse burning, contact NVIDIA technical support for further assistance.

> [!caution] Caution
> `odmfuse.sh` applies to the Jetson AGX Orin series only. For the Jetson AGX Thor series, use `fskp_fuseburn.py` to burn fuses.

## Read Fuses through the Linux kernel

To read the fuse values through the Linux kernel, run the `/usr/sbin/nv_fuse_read.sh` script.

To display the script usage, run the following command:

```
sudo nv_fuse_read.sh -h
```

To list the supported fuses in this script, run the following command:

```
sudo nv_fuse_read.sh -l
```

To read the value of a fuse, run the following command:

```
sudo nv_fuse_read.sh <fuse name>
```

For example, the following command can be used to get the ECID of the Jetson board:

```
sudo nv_fuse_read.sh ecid
```

To read all fuse values, run the following command:

```
sudo nv_fuse_read.sh
```

## Sign and Flash Secured Images

The procedures described in this section use the following placeholders in their commands:

- `<pkc_keyfile>`:
	- For Orin series, it is a RSA 3K, ECDSA P-256, or ECDSA P-521 key file.
	- For Thor series, it is a PKC key list with active key ID. For more information, refer to.
- `<sbk_keyfile>` is an SBK key file.
- `<target_config>` is the name of the configuration for your Jetson device and carrier board; see the table in [Jetson Modules and Configurations](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/IN/QuickStart.html#in-quickstart-jetsonmodulesandconfigurations).

### Sign and Flash with initrd Using the l4t\_initrd\_flash.sh Script

1. Place the Jetson device into Force Recovery mode.
2. To sign the image, run the following command:
	```
	$ sudo ./l4t_initrd_flash.sh --no-flash -u <pkc_keyfile|pkc_keylist> [-v <sbk_keyfile>] <target_config> <rootdev>
	```
	Where:
	- `<target_config>` is the name of the configuration for that Jetson device and carrier board, specified by the `BOARD` environment variable. (Refer to the table in [Jetson Modules and Configurations](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/IN/QuickStart.html#in-quickstart-jetsonmodulesandconfigurations).)
		- `<rootdev>` specifies the device on which the root file system is located, as described in [Basic Flashing Script Usage](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/FlashingSupport.html#sd-flashingsupport-basicflashingscriptusage).
3. To flash the signed images, run the following command:
	```
	$ sudo ./l4t_initrd_flash.sh --flash-only <target_config> <rootdev>
	```
	Where `<target_config>` and `<rootdev>` mean the same as the variables in step 2.

For example:

> To flash an SBKPKC-fused Jetson AGX Thor target using l4t\_initrd\_flash.sh:
> 
> 1. Sign the images:
> 	```
> 	$ sudo ./l4t_initrd_flash.sh --no-flash -u pkc_keylist.xml -v sbk.key jetson-agx-thor-devkit internal
> 	```
> 2. Flash the signed images:
> 	```
> 	$ sudo ./l4t_initrd_flash.sh --flash-only jetson-agx-thor-devkit internal
> 	```
> 
> For more information about PKC key lists, refer to.
> 
> For more information about SBK keys, refer to.

### Sign and Flash Secured Images in One Step

> [!note] Note
> `flash.sh` is not supported for the Jetson Thor series.

1. Navigate to the directory where you installed Jetson Linux.
2. Place the Jetson device into Recovery mode.
3. Enter the following command:
	```
	$ sudo ./flash.sh -u <pkc_keyfile|pkc_keylist> [-v <sbk_keyfile>] <target_config> internal
	```

> [!note] Note
> If the `-v` command option is specified, the `-u` command option also must be specified.
> 
> If the `-v` command option is omitted, all images flashed to the Jetson device are not encrypted.
> 
> If the `-u` command option is omitted, all images flashed to the Jetson device are not signed.

> [!caution] Caution
> None of the PKC key file and SBK key file can be placed under the bootloader directory.

For example,

> To flash a PKC-fused Jetson AGX Orin target:
> 
> ```
> $ sudo ./flash.sh -u <pkc_keyfile> jetson-agx-orin-devkit internal
> ```
> 
> To flash an SBKPKC-fused Jetson AGX Orin target:
> 
> ```
> $ sudo ./flash.sh -u <pkc_keyfile> -v <sbk_keyfile> jetson-agx-orin-devkit internal
> ```

### Sign and Flash Secured Images in Separate Steps

> [!note] Note
> `flash.sh` is not supported for the Jetson Thor series.

1. Sign/encrypt the boot files:
	```
	$ sudo ./flash.sh --no-flash -u <pkc_keyfile> [-v <sbk_keyfile>] <target_config> internal
	```
	> [!note] Note
	> If `-v` command option is specified, `-u` command option must be specified also.
	> 
	> If `-v` command option is omitted, all images flashed to the Jetson device are not encrypted.
	> 
	> If `-u` command option is omitted, all images flashed to the Jetson device are not signed.
2. Flash the generated encrypted/signed images:
	```
	$ cd bootloader
	$ sudo bash ./flashcmd.txt
	```

> [!note] Note
> Ensure that you place the Jetson device into Recovery mode before executing `flashcmd.txt` command.

## Revocation of PKC Keys for Jetson Thor

**Applies only to** the Jetson Thor SoC.

The Jetson Thor SoC supports 16 PKC public keys and provides a revoking mechanism if keys are compromised after the product is shipped. The Jetson Thor SoC is capable of revoking multiple keys at the same time. Only the first 15 PKC keys are revocable. The last PKC key (`key_id="15"`) cannot be revoked.

If a key used in an inactive boot chain is listed for revocation, the Jetson Thor SoC will not revoke it. This prevents the inactive boot chain from becoming unbootable.

To revoke the comprised PKC keys (`key_id` 0 through 14):

1. Determine a revoke bitmap of keys to be revoked. Each bit in the revoke bitmap represents a `key_id`: bit 0 for `key_id="0"`, bit 1 for `key_id="1"`, and so on.
2. Add `u16_fuse_revoke_bitmap = <revoke_bitmap>` to the `brbct` section of the `<br_bct.dts>` file of your target board.
3. In the `<pkc_key_list>` file, edit the `active_index` field to use a `key_id` that is bigger than the largest `key_id` in `<revoke_bitmap>`.
4. Update the bootloader using the `<pkc_key_list>` file for the revocation to take effect. Perform one of the following actions:
	- Flash the device. For details, refer to.
	> [!note] Note
	> Flashing the device will update the bootloader of both boot chains.
	or
	- Perform an Over-the-Air (OTA) update by triggering a UEFI Capsule update **twice**. For details, refer to [Manually Trigger the Capsule Update](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Bootloader/UpdateAndRedundancy.html#sd-bootloader-updateandredundancy-manuallytriggerthecapsuleupdate).
	> [!note] Note
	> Triggering the UEFI Capsule update **twice** updates both boot chains. This is required because key revocation cannot occur while either boot chain still uses a key that is scheduled for revocation.
	> 
	> First, perform the OTA update on the **inactive** boot chain. After it completes and the system successfully boots from the newly updated chain, the previously active chain becomes inactive. Then perform the second OTA update on the new inactive chain. Once the second update completes and the system reboots from it, both boot chains are updated, and key revocation will be executed during boot by MB2.

> [!note] Note
> To find the `<br_bct.dts>` file of your target board, look for the `DEV_PARAMS=` entry in your target board config file.

### An Example: Revoke PKC keys 0, 1, and 5

This example shows how to revoke three PKC keys—0, 1, and 5—on a `jetson-agx-thor-devkit` target.

For these three keys, the revoke bitmap is `0x23`. The entry `DEV_PARAMS="tegra264-br-bct-common-l4t.dts"` is set in `jetson-agx-thor-devkit.conf`.

1. In `<LDK_DIR>/bootloader/generic/BCT/tegra264-br-bct-common-l4t.dts`, after `preprod_dev_sign = <1>;`, add the following entry:
	```
	u16_fuse_revoke_bitmap = <0x23>;
	```
2. In the `<pkc_key_list>` file, change the `active_index=` field to 6 (or a greater value, but no higher than 14):
	```
	active_index="6"
	```
3. Execute the following flash command:
	```
	sudo ./l4t_initrd_flash.sh -u <pkc_key_list> jetson-agx-thor-devkit internal
	```
4. Reset the `jetson-agx-thor-devkit` target to enable the key revocation.
5. Verify that the target can boot successfully.

> [!note] Note
> If the `active_index=` field is set to 2 (or 3 or 4), only PKC keys 0 and 1 are revoked. This happens because, if revocation policy is enabled (bit 5) in the `BootSecurityInfo` fuse, only PKC keys whose `key_id` are less than `active_index` are revoked.

## Revocation of PKC Keys for Jetson Orin

**Applies only to** the Jetson Orin NX series, the Jetson Orin Nano series, and the Jetson AGX Orin series.

The Jetson Orin SoC supports three PKC public keys and provides a revoking mechanism if a key is compromised after the product is shipped.

Here is some information about these keys:

- These PKC keys must be of the same type and strength.
- These keys are OEM programmable and the SHA2-512 hashes of the keys are burned into the fuses (FUSE\_PUBLIC\_KEY, FUSE\_PK\_H1 and FUSE\_PK\_H2) by the OEM during the manufacturing process. (Use the corresponding fuse name of PublicKeyHash, PkcPubkeyHash1, PkcPubkeyHash2 in the Fuse Configuration XML file.)
- To enable ratchet, FUSE\_OPT\_CUSTOMER\_OPTIN\_FUSE (Fuse Configuration file XML entry: <fuse name=”OptInEnable” size=”4” value=”0x1” />) must be burned. This is to prevent running an earlier versions of the software, which compromises the revocation effect.
- The keys are always active until they are revoked, and SoC will accept images signed with any of the non-revoked keys.
- The last key (FUSE\_PK\_H2) is not revocable, and the system can always boot with images signed with the private key of the last key.
- To revoke the first PKC (FUSE\_PUBLIC\_KEY) key:
	1. Add **revoke\_pk\_h0 = <1>** to the **brbct** section of the **<br\_bct.dts>** file of your target board.
		2. Use the second PKC private key or the last PKC private key as the sign key in -u option in flash.sh.
- To revoke the second PKC (FUSE\_PK\_H1) key:
	1. Add **revoke\_pk\_h1 = <1>** to the **brbct** section of the **<br\_bct.dts>** file of your target board.
		2. Use the last PKC private key as the sign key in -u option in flash.sh.
- After a key is revoked, it is permanently unusable. It can not be restored even the revoke\_pk\_h0 or revoke\_pk\_h1 is set to <0>.
- To support PKC keys revocation, all **three PKC keys must be fused** at device provision.

> [!note] Note
> To find the **<br\_bct.dts>** file of your target board, look for “DEV\_PARAMS=” entry of your target board config file.

### An Example: Fusing the Three PKC keys

1. Generate the rsa3k-0.pem, rsa3k-1.pem, and rsa3k-2.pem PKC keys:

```
$ openssl genrsa -out rsa3k-0.pem 3072

$ openssl genrsa -out rsa3k-1.pem 3072

$ openssl genrsa -out rsa3k-2.pem 3072
```

2. Generate the Hash values from the PKC keys:

```
$ ./tegrasign_v3.py --pubkeyhash rsa3k-0.pubkey rsa3k-0.hash --key rsa3k-0.pem

$ ./tegrasign_v3.py --pubkeyhash rsa3k-1.pubkey rsa3k-1.hash --key rsa3k-1.pem

$ ./tegrasign_v3.py --pubkeyhash rsa3k-2.pubkey rsa3k-2.hash --key rsa3k-2.pem
```

3. Create a Fuse Configuration file (fuse\_rsa3k.xml):
	1. Enter the hexadecimal public key hash that was generated from rsa3k-0.pem to the value field of the “PublicKeyHash” fuse name.
		2. Enter the hexadecimal public key hash that was generated from rsa3k-1.pem to the value field of the “PkcPubkeyHash1” fuse name.
		3. Enter the hexadecimal public key hash that was generated from rsa3k-2.pem to the value field of the “PkcPubkeyHash2” fuse name.

> [!note] Note
> For more information about generating the `PublicKeyHash` fuse value, refer to.

Here is an example Fuse Configuration file:

```
<genericfuse MagicId="0x45535546" version="1.0.0">
   <fuse name="SecureBootKey" size="32" value="0x123456789abcdef0fedcba987654321023456789abcdef01edcba9876543210f"/>
   <fuse name="PublicKeyHash" size="64" value="0xad2474627c14e3f7f4944a832bd15d0640938a3dc162f558692458f3d12f9453e11bea2ec75df3f83e8b29c47fc3d2483d528d3e94a5469c4ba1ec61f1584b23"/>
   <fuse name="PkcPubkeyHash1" size="64" value="0xd87796fb510d79738f8509c98511be0bb79dcc17d204a2f0f0bea9680b91bd1273ee2ae7a8a6bdb8b95deb0f421e72404939ae20d12c82649712283027201f39"/>
   <fuse name="PkcPubkeyHash2" size="64" value="0x99a5b6eac64dfb29698cb684165529e5d8650c1aab0e18b677c5d5f0998af53f8a8a1f09ad1d79368bc500e57eb199e9108fc7b1499995d869b028fec3f367db"/>
   <fuse name="OptInEnable" size="4" value="0x1"/>
   <fuse name="BootSecurityInfo" size="4" value="0x9"/>
   <fuse name="SecurityMode" size="4" value="0x1"/>
</genericfuse>
```

4. Burn the fuses with the Fuse Configuration file (fuse\_rsa3k.xml):

```
$ sudo ./odmfuse.sh -X fuse_rsa3k.xml -i 0x23 jetson-agx-orin-devkit
```

> [!note] Note
> In the following examples, the sbk key is stored in file sbk-32.key with content:
> 
> > `0x12345678 0x9abcdef0 0xfedcba98 0x76543210 0x23456789 0xabcdef01 0xedcba987 0x6543210f`

### An Example: Revoking the First PKC key (rsa3k-0.pem)

1. Add **revoke\_pk\_h0 = <1>** to **tegra234-br-bct-p3767-0000-l4t.dts**:

```
/dts-v1/;

/ {
    brbct {
        . . .
        revoke_pk_h0 = <1>;
        bf_bl_allbits {
            . . .
        }
    };
};
```

2. Flash with rsa3k-1.pem or rsa3k-2.pem:

Option 1: rsa3k-1.pem

```
$ sudo ./flash.sh -u rsa3k-1.pem -v sbk-32.key jetson-agx-orin-devkit internal
```

Option 2: rsa3k-2.pem

```
$ sudo ./flash.sh -u rsa3k-2.pem -v sbk-32.key jetson-agx-orin-devkit internal
```

3. Use the UEFI Capsule update to revoke the first PKC key.
	1. Generate the Capsule payload with the modified dts file. For more information, refer to [Generating the Capsule Update Payload](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Bootloader/UpdateAndRedundancy.html#sd-bootloader-updateandredundancy-generatingthecapsuleupdatepayload).
		Option 1: Generate the Capsule payload signed by the second PKC key rsa3k-1.pem.
		```
		$ sudo ./l4t_generate_soc_bup.sh -u rsa3k-1.pem -v sbk-32.key -e t23x_agx_bl_spec t23x
		$ ./generate_capsule/l4t_generate_soc_capsule.sh -i bootloader/payloads_t23x/bl_only_payload -o ./TEGRA_BL.Cap t234
		```
		Option 2: Generate the Capsule payload signed by the third PKC key rsa3k-2.pem.
		```
		$ sudo ./l4t_generate_soc_bup.sh -u rsa3k-2.pem -v sbk-32.key -e t23x_agx_bl_spec t23x
		$ ./generate_capsule/l4t_generate_soc_capsule.sh -i bootloader/payloads_t23x/bl_only_payload -o ./TEGRA_BL.Cap t234
		```
		2. Trigger a Capsule update. For more information, refer to [Use the Helper Script to Trigger the Capsule Update](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Bootloader/UpdateAndRedundancy.html#sd-bootloader-updateandredundancy-usethehelperscripttotriggerthecapsuleupdate).

### An Example: Revoking the Second PKC key (rsa3k-1.pem)

1. Add **revoke\_pk\_h1 = <1>** to **tegra234-br-bct-p3767-0000-l4t.dts**:

```
/dts-v1/;

/ {
    brbct {
        . . .
        revoke_pk_h1 = <1>;
        bf_bl_allbits {
            . . .
        }
    };
};
```

2. Flash with rsa3k-2.pem:

```
$ sudo ./flash.sh -u rsa3k-2.pem -v sbk-32.key jetson-agx-orin-devkit internal
```

3. Use the UEFI Capsule update to revoke the second PKC key.
	1. Generate the Capsule payload with the modified dts file. For more information, refer to [Generating the Capsule Update Payload](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Bootloader/UpdateAndRedundancy.html#sd-bootloader-updateandredundancy-generatingthecapsuleupdatepayload).
	> ```
	> $ sudo ./l4t_generate_soc_bup.sh -u rsa3k-2.pem -v sbk-32.key -e t23x_agx_bl_spec t23x
	> $ ./generate_capsule/l4t_generate_soc_capsule.sh -i bootloader/payloads_t23x/bl_only_payload -o ./TEGRA_BL.Cap t234
	> ```
	2. Trigger a Capsule update. For more information, refer to [Use the Helper Script to Trigger the Capsule Update](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Bootloader/UpdateAndRedundancy.html#sd-bootloader-updateandredundancy-usethehelperscripttotriggerthecapsuleupdate).

## UEFI Secure Boot

UEFI Secure Boot uses digital signatures (RSA) to validate the authenticity and integrity of the codes that it loads.

UEFI Secure Boot implementations use PK, KEK, and db keys:

> - Platform Key (PK): Top-level key, is used to sign KEK.
> - Key Exchange Key (KEK): Keys used to sign Signatures Database.
> - Signature Database (db): Contains keys to sign UEFI payloads.

Before enabling UEFI Secure Boot, users have to prepare their own PK, KEK and db keys.

Then, users can enable Secure Boot either:

> - At flashing time; Or,
> - At the target from Ubuntu prompt.

The following diagram illustrates how PK/KEK/db keys are used to sign and validate UEFI’s payloads:

![UEFI 보안 부팅에서 PK/KEK/db 키는 어떻게 사용되나요?](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/_images/UEFI_Secureboot.png)

UEFI 보안 부팅에서 PK/KEK/db 키는 어떻게 사용되나요?

1. Enroll PK, KEK, and db keys in the form of UEFI authenticated variable.
2. Sign UEFI payloads such as L4tLauncher (as OS Loader), kernel, kernel-dtb with private key and flash signed images (on Host).
3. UEFI loads signed images.
4. UEFI Verifies image signature by using the associated certificate/public key, and verifies the certificate/public key existing in db but not in dbx.

Here is a high-level process to enable UEFI Secure Boot:

1. Generate the secure boot artifacts: the PK, KEK, and db key pairs, their certificates, and the EFI signature list files.
2. Enable UEFI Secure Boot using one of the following three methods:
	- **During flashing**, using a command option:
		> 1. Create a UEFI keys config file.
		> 2. Generate `UefiDefaultSecurityKeys.dtbo` and the auth files.
		> 3. Use option `--uefi-keys <keys_conf>` to provide signing keys and enable UEFI secure boot.
		- **After flashing**, using Capsule update:
		> 1. Create a UEFI keys config file.
		> 2. Generate `UefiDefaultSecurityKeys.dtbo`.
		> 3. Generate the Capsule payload with `UefiDefaultSecurityKeys.dtbo`.
		> 4. Generate signed UEFI payloads on the host.
		> 5. Download and install the Secure Boot artifacts.
		> 6. Trigger a Capsule update.
		> 7. Check UEFI Secure Boot status.
		- **After flashing**, using UEFI utilities from an Ubuntu prompt:
		> 1. Generate the auth files.
		> 2. Generate signed UEFI payloads on the host.
		> 3. Download the PK, KEK, and db auth files from the host.
		> 4. Enroll the KEK and db keys.
		> 5. Download and write the signed UEFI payloads.
		> 6. Enroll the PK key.
3. Update the db/dbx Keys with a Capsule update
	> 1. Prepare the update keys.
	> 2. Generate the Capsule payload with UEFI Secure Boot enabled.
	> 3. Trigger a Capsule update.
	> 4. Check and verify the update keys.

> [!note] Note
> When UEFI Secure Boot is enabled during the flashing process, it cannot be disabled unless you flash again.
> 
> However, if UEFI Secure Boot is enabled through UEFI utilities running from the Ubuntu prompt, you can disable it by accessing the UEFI Menu and selecting **Reset Secure Boot Keys**, provided you have the necessary permissions. UEFI Secure Boot can also be disabled by enrolling `noPK.auth` at runtime. Assuming only an admin has access to `noPK.auth`, they can disable UEFI Secure Boot on next boot by running the kernel utility `efi-updatevar` with `noPK.auth`.

### Prerequisites

Ensure that the following utilities are installed in your host:

- openssl
- device-tree-compiler
- efitools
- uuid-runtime

### Prepare the PK, KEK, db Keys

#### Generate the PK, KEK, and DB RSA Key Pairs, Certificates and EFI Signature List Files

To generate the PK, KEK, and db RSA key pairs, their certificates, and the EFI signature list files, run the following commands:

```
$ cd to <LDK_DIR>
$ mkdir uefi_keys
$ cd uefi_keys
$ GUID=$(uuidgen)

### Generate PK RSA Key Pair, Certificate, and EFI Signature List File
$ openssl req -newkey rsa:2048 -nodes -keyout PK.key  -new -x509 -sha256 -days 3650 -subj "/CN=my Platform Key/" -out PK.crt
$ cert-to-efi-sig-list -g "${GUID}" PK.crt PK.esl

### Generate KEK RSA Key Pair, Certificate, and EFI Signature List File
$ openssl req -newkey rsa:2048 -nodes -keyout KEK.key  -new -x509 -sha256 -days 3650 -subj "/CN=my Key Exchange Key/" -out KEK.crt
$ cert-to-efi-sig-list -g "${GUID}" KEK.crt KEK.esl

### Generate db_1 RSA Key Pair, Certificate, and EFI Signature List File
$ openssl req -newkey rsa:2048 -nodes -keyout db_1.key  -new -x509 -sha256 -days 3650 -subj "/CN=my Signature Database key/" -out db_1.crt
$ cert-to-efi-sig-list -g "${GUID}" db_1.crt db_1.esl

### Generate db_2 RSA Key Pair, Certificate, and EFI Signature List File
$ openssl req -newkey rsa:2048 -nodes -keyout db_2.key  -new -x509 -sha256 -days 3650 -subj "/CN=my another Signature Database key/" -out db_2.crt
$ cert-to-efi-sig-list -g "${GUID}" db_2.crt db_2.esl
```

> [!caution] Caution
> The generated.crt files are self-signed certificates and are used for demonstration purposes only. For production, follow your official certificate generation procedure.

### Generate the UEFI Secure Boot DTBO

The figure shows the process of generating the UEFI Secure Boot DTBO:

> - Generate `UefiDefaultSecurityKeys.dtbo` to enable UEFI Secure Boot and enroll keys at flashing time or with a Capsule update.
> - Generate `UefiUpdateSecurityKeys.dtbo` to update the UEFI Secure Boot keys with a Capsule update.

![UEFI 보안 부팅을 위해 DTBO가 생성되고 사용되는 방법](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/_images/UEFI_Secureboot_Gen_DTBO.png)

UEFI 보안 부팅을 위해 DTBO가 생성되고 사용되는 방법

### Enable the UEFI Secure Boot

#### Method One: Enable UEFI Secure Boot at Flashing Time

> [!note] Note
> We strongly recommend to use this method for production devices.

##### Create a UEFI Keys Config File

To create a UEFI keys config file with the generated keys, run the following command:

```
$ vim uefi_keys.conf
```

Insert the following lines to uefi\_keys.conf file:

```
UEFI_DB_1_KEY_FILE="db_1.key";  # UEFI payload signing key
UEFI_DB_1_CERT_FILE="db_1.crt"; # UEFI payload signing key certificate

UEFI_DEFAULT_PK_ESL="PK.esl"
UEFI_DEFAULT_KEK_ESL_0="KEK.esl"

UEFI_DEFAULT_DB_ESL_0="db_1.esl"
UEFI_DEFAULT_DB_ESL_1="db_2.esl"
```

> [!note] Note
> - The minimum number of `UEFI_DEFAULT_DB_ESL` is 1. The `UEFI_DEFAULT_DB_ESL_1` entry shown above is optional.
> - `UEFI_DB_1_KEY_FILE` and `UEFI_DB_1_CERT_FILE` are used to sign UEFI payloads, such as kernel, kernel-dtb, and initrd. As a result, `UEFI_DB_1_KEY_FILE` and `UEFI_DB_1_CERT_FILE` must be specified in `uefi_keys.conf`.
> - Microsoft has two DB certificates and one KEK certificate, and these certificates can be used based on your requirement. Refer to [Microsoft’s certificates](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#Microsoft_Windows) for more information.
> - The UEFI revocation list file, which is used to update the Secure Boot Forbidden Signature Database (dbx). Download the revocation list file from [UEFI Revocation List File for arm64](https://uefi.org/revocationlistfile).
> - For a system installed with a UEFI option ROM, and that is signed with a Microsoft db, you must enroll the Microsoft db, the KEK, and the UEFI dbx certificates. Assign the corresponding key esl files to the variables `UEFI_DEFAULT_KEK_ESL_X` (up to 3), `UEFI_DEFAULT_DB_ESL_X` (update to 3), and `UEFI_DEFAULT_DBX_ESL_X` (up to 3) in the `uefi_keys.conf` file.

##### Generate UefiDefaultSecurityKeys.dtbo

To enable UEFI Secure Boot at flashing time, the UEFI default security keys need to be flashed to target. The UEFI default security keys are embedded in `UefiDefaultSecurityKeys.dtbo` and are used during flashing.

`UefiDefaultSecurityKeys.dtbo` and the auth files are generated by using the `gen_uefi_keys_dts.sh` script.

Run the following commands:

```
$ cd ..
$ sudo tools/gen_uefi_keys_dts.sh uefi_keys/uefi_keys.conf
```

##### Use the –uefi-keys Option to Provide Signing Keys and Enable UEFI Secure Boot

> [!note] Note
> Although UEFI Secure Boot can be independently enabled from a low-level bootloader secure boot, we strongly recommended that users enable bootloader secure boot so that the root-of-trust can start from the BootROM.

Issue the following commands with the `--uefi-keys <keys.conf>` option:

- For the Jetson AGX Thor series:
	```
	$ sudo ./l4t_initrd_flash.sh -u <pkc_keyfile> [-v <sbk_keyfile>] --uefi-keys uefi_keys/uefi_keys.conf jetson-agx-thor-devkit internal
	```
- For the Jetson AGX Orin series:
	- Use eMMC as rootfs storage:
		```
		$ sudo ./l4t_initrd_flash.sh -u <pkc_keyfile> [-v <sbk_keyfile>] --uefi-keys uefi_keys/uefi_keys.conf jetson-agx-orin-devkit internal
		```
		- Use NVMe as rootfs storage:
		```
		$ sudo ./l4t_initrd_flash.sh --external-device nvme0n1p1 -u <pkc_keyfile> [-v <sbk_keyfile>] --uefi-keys uefi_keys/uefi_keys.conf -p "-c ./bootloader/generic/cfg/flash_t234_qspi.xml" -c ./tools/kernel_flash/flash_l4t_t234_nvme.xml jetson-agx-orin-devkit external
		```
- For the Jetson Orin NX series and the Orin Nano series:
	```
	$ sudo ./l4t_initrd_flash.sh --external-device nvme0n1p1 -u <pkc_keyfile> [-v <sbk_keyfile>] --uefi-keys uefi_keys/uefi_keys.conf -p "-c ./bootloader/generic/cfg/flash_t234_qspi.xml" -c ./tools/kernel_flash/flash_l4t_t234_nvme.xml jetson-orin-nano-devkit external
	```

For more information about `pkc_keyfile` and `sbk_keyfile`, refer to.

Once flashing is finished, your target has UEFI Secure Boot enabled.

#### Method Two: Enable UEFI Secure Boot Using Capsule Update

This section is for the targets that were not flashed with UEFI Secure Boot enabled.

##### Generate UefiDefaultSecurityKeys.dtbo

Refer to and to generate a `UefiDefaultSecurityKeys.dtbo` file.

Copy the generated `UefiDefaultSecurityKeys.dtbo` to the bootloader directory:

```
$ cd to <LDK_DIR>
$ cp uefi_keys/UefiDefaultSecurityKeys.dtbo bootloader/
```

##### Generate a Capsule Payload with UEFI Secure Boot Keys

The `uefi_keys/UefiDefaultSecurityKeys.dtbo` generated earlier is packed into Capsule payload with cpu-bootloader. Refer to [Generating the Capsule Update Payload](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Bootloader/UpdateAndRedundancy.html#sd-bootloader-updateandredundancy-generatingthecapsuleupdatepayload) for more information.

To generate a Capsule payload, complete one of the following tasks:

- Generate a Capsule payload for the Jetson AGX Thor devkits:
	```
	$ sudo ADDITIONAL_DTB_OVERLAY="UefiDefaultSecurityKeys.dtbo" ./l4t_generate_soc_bup.sh -u <pkc_keyfile> [-v <sbk_keyfile>] -e t26x_3834_bl_spec t26x
	$ ./generate_capsule/l4t_generate_soc_capsule.sh -i bootloader/payloads_t26x/bl_only_payload -o ./TEGRA_AGX_THOR.Cap t264
	```
- Generate a Capsule payload for the Jetson AGX Orin devkits:
	```
	$ sudo ADDITIONAL_DTB_OVERLAY="UefiDefaultSecurityKeys.dtbo" ./l4t_generate_soc_bup.sh -u <pkc_keyfile> [-v <sbk_keyfile>] -e t23x_agx_bl_spec t23x
	$ ./generate_capsule/l4t_generate_soc_capsule.sh -i bootloader/payloads_t23x/bl_only_payload -o ./TEGRA_AGX.Cap t234
	```
- Generate a Capsule payload for the Jetson AGX Orin Industrial:
	```
	$ sudo ADDITIONAL_DTB_OVERLAY="UefiDefaultSecurityKeys.dtbo" ./l4t_generate_soc_bup.sh -u <pkc_keyfile> [-v <sbk_keyfile>] -e t23x_agx_ind_bl_spec t23x
	$ ./generate_capsule/l4t_generate_soc_capsule.sh -i bootloader/payloads_t23x/bl_only_payload -o ./TEGRA_AGX_IND.Cap t234
	```
- Generate a Capsule payload for the Jetson Orin Nano devkits:
	```
	$ sudo ADDITIONAL_DTB_OVERLAY="UefiDefaultSecurityKeys.dtbo" ./l4t_generate_soc_bup.sh -u <pkc_keyfile> [-v <sbk_keyfile>] -e t23x_3767_bl_spec t23x
	$ ./generate_capsule/l4t_generate_soc_capsule.sh -i bootloader/payloads_t23x/bl_only_payload -o ./TEGRA_Nano.Cap t234
	```

##### Generate Signed UEFI Payloads for Jetson Thor

All UEFI payloads must be signed using UEFI security keys. If the `--uefi-keys` option is specified during flashing, the UEFI payloads are signed during flashing images generation. The UEFI secure boot is enabled after flashing. To enable UEFI Secure Boot via Capsule update, the UEFI payloads must be signed from the host. Before starting Capsule update, the signed payloads must be preinstalled to the target.

The UEFI payloads are

- extlinux.conf,
- initrd,
- kernel images (in rootfs, and in recovery partitions),
- kernel-dtb images (in rootfs and in recovery-dtb partition), and
- BOOTAA64.efi.

Create a directory named `<LDK_DIR>/uefi_signed/` to store the signed UEFI payloads:

```
$ cd <LDK_DIR>
$ mkdir -p uefi_signed
$ cd uefi_signed
```

> [!note] Note
> The following steps assume that you have copied the required unsigned UEFI payloads to the `<LDK_DIR>/uefi_signed/` folder. You can replace `db.crt` and `db.key` with the `db_1.*` or `db_2.*` key.

1. To sign extlinux.conf using db:
	```
	$ openssl cms -sign -signer ../uefi_keys/db.crt -inkey ../uefi_keys/db.key -binary -in extlinux.conf -outform der -out extlinux.conf.sig
	```
2. To sign initrd using db:
	```
	$ openssl cms -sign -signer ../uefi_keys/db.crt -inkey ../uefi_keys/db.key -binary -in initrd -outform der -out initrd.sig
	```
3. To sign the kernel Image in rootfs using db:
	```
	$ cp Image Image.unsigned
	$ sbsign --key ../uefi_keys/db.key --cert ../uefi_keys/db.crt --output Image Image
	```
4. To sign kernel-dtb in rootfs using db:
	```
	$ openssl cms -sign -signer ../uefi_keys/db.crt -inkey ../uefi_keys/db.key -binary -in kernel_tegra264-p4071-0000+p3834-0008-nv.dtb -outform der -out kernel_tegra264-p4071-0000+p3834-0008-nv.dtb.sig
	```
	> [!note] Note
	> The preceding command uses the Jetson Thor SKU 8 kernel-dtb filename. Replaced it with the appropriate kernel-dtb filename of your target.
5. To sign recovery.img flashed to recovery partition using db:
	```
	$ ../bootloader/mkbootimg --kernel Image --ramdisk ../bootloader/recovery.ramdisk --output recovery.img --cmdline <rec_cmdline_string>
	$ cp recovery.img recovery.img.unsigned
	$ openssl cms -sign -signer ../uefi_keys/db.crt -inkey ../uefi_keys/db.key -binary -in recovery.img -outform der -out recovery.img.sig
	$ truncate -s %4096 recovery.img
	$ cat recovery.img.sig >> recovery.img
	```
	In the first command, `<rec_cmdline_string>` for the Jetson Thor series is as follows:
	```
	root=/dev/initrd rw rootwait mminit_loglevel=4 console=ttyTCU0,115200 firmware_class.path=/etc/firmware fbcon=map:0 net.ifnames=0
	```
	> [!note] Note
	> The kernel `Image` inside `recovery.img` must also be signed. Use the `Image` signed earlier in step 3.
6. To sign recovery kernel dtb flashed to recovery-dtb partition using db:
	```
	$ cp tegra264-p4071-0000+p3834-0008-nv.dtb.rec tegra264-p4071-0000+p3834-0008-nv.dtb.rec.unsigned
	$ openssl cms -sign -signer ../uefi_keys/db.crt -inkey ../uefi_keys/db.key -binary -in tegra264-p4071-0000+p3834-0008-nv.dtb.rec -outform der -out tegra264-p4071-0000+p3834-0008-nv.dtb.rec.sig
	$ truncate -s %4096 tegra264-p4071-0000+p3834-0008-nv.dtb.rec
	$ cat tegra264-p4071-0000+p3834-0008-nv.rec.sig >> tegra264-p4071-0000+p3834-0008-nv.dtb.rec
	```
	> [!note] Note
	> The preceding commands use the Jetson Thor SKU 8 kernel-dtb filename. Replace it with the appropriate kernel-dtb filename of your target. The images signed earlier in steps 5 and 6 are stored into the partition. For those images, the signing certificate and signature are appended to the original image after being aligned to a 4-KB boundary.
7. To sign BOOTAA64.efi using db:
	```
	$ cp BOOTAA64.efi BOOTAA64.efi.unsigned
	$ sbsign --key ../uefi_keys/db.key --cert ../uefi_keys/db.crt --output BOOTAA64.efi BOOTAA64.efi
	```

##### Generate Signed UEFI Payloads for Jetson Orin

All UEFI payloads must be signed using UEFI security keys. If the `--uefi-keys` option is specified during flashing, the UEFI payloads are signed during flashing images generation. The UEFI secure boot is enabled after flashing. To enable UEFI Secure Boot via Capsule update, the UEFI payloads must be signed from the host. Before starting Capsule update, the signed payloads must be preinstalled to the target.

The UEFI payloads are

- extlinux.conf,
- initrd,
- kernel images (in rootfs, and in kernel and recovery partitions),
- kernel-dtb images (in rootfs and in kernel-dtb and recovery-dtb partitions), and
- BOOTAA64.efi.

Create a directory named `<LDK_DIR>/uefi_signed/` to store the signed UEFI payloads:

```
$ cd <LDK_DIR>
$ mkdir -p uefi_signed
$ cd uefi_signed
```

> [!note] Note
> The following steps assume that you have copied the required unsigned UEFI payloads to the `<LDK_DIR>/uefi_keys/` folder. You can replace `db.crt` and `db.key` with the `db_1.*` or `db_2.*` key.

1. To sign extlinux.conf using db:
	```
	$ openssl cms -sign -signer ../uefi_keys/db.crt -inkey ../uefi_keys/db.key -binary -in extlinux.conf -outform der -out extlinux.conf.sig
	```
2. To sign initrd using db:
	```
	$ openssl cms -sign -signer ../uefi_keys/db.crt -inkey ../uefi_keys/db.key -binary -in initrd -outform der -out initrd.sig
	```
3. To sign the kernel Image in rootfs using db:
	```
	$ cp Image Image.unsigned
	$ sbsign --key ../uefi_keys/db.key --cert ../uefi_keys/db.crt --output Image Image
	```
4. To sign kernel-dtb in rootfs using db:
	```
	$ openssl cms -sign -signer ../uefi_keys/db.crt -inkey ../uefi_keys/db.key -binary -in kernel_tegra234-p3701-0004-p3737-0000.dtb -outform der -out kernel_tegra234-p3701-0004-p3737-0000.dtb.sig
	```
	> [!note] Note
	> The preceding commands use the Concord SKU 4 kernel-dtb filename. Replaced it with the appropriate kernel-dtb filename of your target.
5. To sign boot.img of kernel partition using db:
	```
	$ ../bootloader/mkbootimg --kernel Image --ramdisk initrd --board <rootdev> --output boot.img --cmdline <cmdline_string>
	$ cp boot.img boot.img.unsigned
	$ openssl cms -sign -signer ../uefi_keys/db.crt -inkey ../uefi_keys/db.key -binary -in boot.img -outform der -out boot.img.sig
	$ truncate -s %4096 boot.img
	$ cat boot.img.sig >> boot.img
	```
	In the first command, `<cmdline_string>`, when generated in `flash.sh` to flash eMMC/SD, is as follows:
	```
	root=/dev/mmcblk0p1 rw rootwait rootfstype=ext4 mminit_loglevel=4 console=ttyTCU0,115200 console=ttyAMA0,115200 firmware_class.path=/etc/firmware fbcon=map:0 net.ifnames=0
	```
	When generated in `l4t_initrd_flash.sh` to flash NVMe, `<cmdline_string>` is as follows:
	```
	root=/dev/nvme0n1p1 rw rootwait rootfstype=ext4 mminit_loglevel=4 console=ttyTCU0,115200 console=ttyAMA0,115200 firmware_class.path=/etc/firmware fbcon=map:0 net.ifnames=0
	```
	> [!note] Note
	> The `Image` inside `boot.img` must also be signed. Use the `Image` signed earlier in step 3.
6. To sign kernel-dtb of kernel-dtb partition using db:
	```
	$ cp tegra234-p3701-0004-p3737-0000.dtb tegra234-p3701-0004-p3737-0000.dtb.unsigned
	$ openssl cms -sign -signer ../uefi_keys/db.crt -inkey ../uefi_keys/db.key -binary -in tegra234-p3701-0004-p3737-0000.dtb -outform der -out tegra234-p3701-0004-p3737-0000.dtb.sig
	$ truncate -s %4096 tegra234-p3701-0004-p3737-0000.dtb
	$ cat tegra234-p3701-0004-p3737-0000.dtb.sig >> tegra234-p3701-0004-p3737-0000.dtb
	```
	> [!note] Note
	> The preceding commands use the Concord SKU 4 kernel-dtb filename. Replaced it with the appropriate kernel-dtb filename of your target.
7. To sign recovery.img flashed to recovery partition using db:
	```
	$ ../bootloader/mkbootimg --kernel Image --ramdisk ../bootloader/recovery.ramdisk --output recovery.img --cmdline <rec_cmdline_string>
	$ cp recovery.img recovery.img.unsigned
	$ openssl cms -sign -signer ../uefi_keys/db.crt -inkey ../uefi_keys/db.key -binary -in recovery.img -outform der -out recovery.img.sig
	$ truncate -s %4096 recovery.img
	$ cat recovery.img.sig >> recovery.img
	```
	In the first command, `<rec_cmdline_string>` is as follows:
	```
	root=/dev/initrd rw rootwait mminit_loglevel=4 console=ttyTCU0,115200 firmware_class.path=/etc/firmware fbcon=map:0 net.ifnames=0
	```
	> [!note] Note
	> The kernel `Image` inside `recovery.img` must also be signed. Use the `Image` signed by the step 3 above.
8. To sign recovery kernel dtb flashed to recovery-dtb partition using db:
	```
	$ cp tegra234-p3701-0004-p3737-0000.dtb.rec tegra234-p3701-0004-p3737-0000.dtb.rec.unsigned
	$ openssl cms -sign -signer ../uefi_keys/db.crt -inkey ../uefi_keys/db.key -binary -in tegra234-p3701-0004-p3737-0000.dtb.rec -outform der -out tegra234-p3701-0004-p3737-0000.dtb.rec.sig
	$ truncate -s %4096 tegra234-p3701-0004-p3737-0000.dtb.rec
	$ cat tegra234-p3701-0004-p3737-0000.dtb.rec.sig >> tegra234-p3701-0004-p3737-0000.dtb.rec
	```
	> [!note] Note
	> The preceding commands use the Concord SKU 4 kernel-dtb filename. Replaced it with the appropriate kernel-dtb filename of your target. The images signed earlier in step 5 to 8 are stored into the partition. For those images, the signing certificate and signature are appended to the original image after being aligned to 4-KB boundary.
9. To sign BOOTAA64.efi using db:
	```
	$ cp BOOTAA64.efi BOOTAA64.efi.unsigned
	$ sbsign --key ../uefi_keys/db.key --cert ../uefi_keys/db.crt --output BOOTAA64.efi BOOTAA64.efi
	```

##### Download and Install the Secure Boot Artifacts for Jetson Thor

Download the signed UEFI payloads from the host to the payloads’ corresponding target folder as shown in the following table.

<table><thead><tr><th><p>File in Host’s <LDK_DIR>/uefi_signed/ Folder</p></th><th><p>Target Folder</p></th><th><p>Type</p></th></tr></thead><tbody><tr><td><p>extlinux.conf and extlinux.conf.sig</p></td><td><p>/boot/extlinux/</p></td><td><p>rootfs</p></td></tr><tr><td rowspan="2"><p>initrd and initrd.sig</p></td><td rowspan="2"><p>/boot/</p></td><td rowspan="2"><p>rootfs</p></td></tr><tr><td><p>kernel_tegra264-p4071-0000+p3834-0008-nv.dtb, and kernel_tegra264-p4071-0000+p3834-0008-nv.dtb.sig (for Jetson Thor SKU 8)</p></td><td><p>/boot/dtb/</p></td><td><p>rootfs</p></td></tr><tr><td><p>Image</p></td><td><p>/boot/</p></td><td><p>rootfs</p></td></tr><tr><td><p>BOOTAA64.efi</p></td><td><p>/uefi_signed/</p></td><td><p>esp partition</p></td></tr><tr><td><p>recovery.img</p></td><td><p>/uefi_signed/</p></td><td><p>recovery partition</p></td></tr><tr><td><p>tegra264-p4071-0000+p3834-0008-nv.dtb.rec (for Jetson Thor SKU 8)</p></td><td><p>/uefi_signed/</p></td><td><p>recovery-dtb partition</p></td></tr></tbody></table>

> [!note] Note
> You might want to save copies of the original files.

For the UEFI payload files with the `rootfs` type, the target folder listed in the table is their final destination. The other files need to be copied to their destination partitions. To copy a file to its destination partition, copy the file to a block device that is mapped to that partition.

> [!note] Note
> The EFI System Partition (`esp`) is mounted to the `/boot/efi/` directory automatically. To verify, use the command `df -h` in kernel. If the `esp` is not mounted, find the partition by using the command `sudo fdisk -l | grep "EFI"` and then mount the partition to the `/boot/efi/` directory manually.

To determine which block devices are mapped to a particular partition, use `blkid`:

```
$ sudo blkid | grep <part_name>
```

The value of `<part_name>` is one of the following:

- `recovery`
- `recovery-dtb`

> [!note] Note
> - If multiple block devices are mapped to a partition, choose the boot device.
> - Beginning with JetPack 7.1, the `kernel` and `kernel-dtb` partitions are removed for Jetson Thor. If UEFI fails to boot the file system kernel (`Attempting Direct Boot`) after three attempts, it boots from the `recovery` partition.

1. To write the signed BOOTAA64.efi to `esp` partition:
	```
	$ cp /uefi_signed/BOOTAA64.efi /boot/efi/EFI/BOOT/BOOTAA64.efi
	$ sync
	```
2. To write the signed recovery.img to `recovery` partition:
	```
	### Ex: recovery partition is mapped to /dev/nvme0n1p2
	$ dd if=/uefi_signed/recovery.img of=/dev/nvme0n1p2 bs=64k
	```
3. To write the signed recovery kernel-dtb to `recovery-dtb` partition:
	```
	### Ex: recovery-dtb partition is mapped to /dev/nvme0n1p3
	$ dd if=/uefi_signed/tegra264-p4071-0000+p3834-0008-nv.dtb.rec of=/dev/nvme0n1p3 bs=64k
	```

##### Download and Install the Secure Boot Artifacts for Jetson Orin

Download the signed UEFI payloads from the host to the payloads’ corresponding target folder as shown in the following table.

<table><thead><tr><th><p>File in Host’s <LDK_DIR>/uefi_signed/ Folder</p></th><th><p>Target Folder</p></th><th><p>Type</p></th></tr></thead><tbody><tr><td><p>extlinux.conf and extlinux.conf.sig</p></td><td><p>/boot/extlinux/</p></td><td><p>rootfs</p></td></tr><tr><td rowspan="2"><p>initrd and initrd.sig</p></td><td rowspan="2"><p>/boot/</p></td><td rowspan="2"><p>rootfs</p></td></tr><tr><td><p>kernel_tegra234-p3701-0004-p3737-0000.dtb, and kernel_tegra234-p3701-0004-p3737-0000.dtb.sig (for Concord SKU 4)</p></td><td><p>/boot/dtb/</p></td><td><p>rootfs</p></td></tr><tr><td><p>Image</p></td><td><p>/boot/</p></td><td><p>rootfs</p></td></tr><tr><td><p>BOOTAA64.efi</p></td><td><p>/uefi_signed/</p></td><td><p>esp partition</p></td></tr><tr><td><p>boot.img</p></td><td><p>/uefi_signed/</p></td><td><p>A/B_kernel partition</p></td></tr><tr><td><p>tegra234-p3701-0004-p3737-0000.dtb (for Concord SKU 4)</p></td><td><p>/uefi_signed/</p></td><td><p>A/B_kernel-dtb partition</p></td></tr><tr><td><p>recovery.img</p></td><td><p>/uefi_signed/</p></td><td><p>recovery partition</p></td></tr><tr><td><p>tegra234-p3701-0004-p3737-0000.dtb.rec (for Concord SKU 4)</p></td><td><p>/uefi_signed/</p></td><td><p>recovery-dtb partition</p></td></tr></tbody></table>

> [!note] Note
> You might want to save copies of the original files.

For the UEFI payload files with the `rootfs` type, the target folder listed in the table is their final destination. The other files need to be copied to their destination partitions. To copy a file to its destination partition, copy the file to a block device that is mapped to that partition.

> [!note] Note
> The EFI System Partition (`esp`) is mounted to the `/boot/efi/` directory automatically. To verify, use the command `df -h` in kernel. If the `esp` is not mounted, find the partition by using the command `sudo fdisk -l | grep "EFI"` and then mount the partition to the `/boot/efi/` directory manually.

To determine which block devices are mapped to a particular partition, use `blkid`:

```
$ sudo blkid | grep <part_name>
```

The value of `<part_name>` is one of the following:

- `A_kernel`
- `B_kernel`
- `A_kernel-dtb`
- `B_kernel-dtb`
- `recovery`
- `recovery-dtb`

> [!note] Note
> - If multiple block devices are mapped to a partition, choose the boot device.
> - If UEFI fails to boot the file system kernel, it boots from the `kernel` partition.

1. To write the signed BOOTAA64.efi to `esp` partition:
	```
	$ cp /uefi_signed/BOOTAA64.efi /boot/efi/EFI/BOOT/BOOTAA64.efi
	$ sync
	```
2. To write the signed boot.img to `A_kernel` partition:
	```
	### Ex: A_kernel partition is mapped to /dev/mmcblk0p2
	$ dd if=/uefi_signed/boot.img of=/dev/mmcblk0p2 bs=64k
	```
3. To write the signed boot.img to `B_kernel` partition:
	```
	### Ex: B_kernel partition is mapped to /dev/mmcblk0p5
	$ dd if=/uefi_signed/boot.img of=/dev/mmcblk0p5 bs=64k
	```
4. To write the signed kernel-dtb to `A_kernel-dtb` partition:
	```
	### Ex: A_kernel-dtb partition is mapped to /dev/mmcblk0p3
	$ dd if=/uefi_signed/tegra234-p3701-0004-p3737-0000.dtb of=/dev/mmcblk0p3 bs=64k
	```
5. To write the signed kernel-dtb to `B_kernel-dtb` partition:
	```
	### Ex: B_kernel-dtb partition is mapped to /dev/mmcblk0p6
	$ dd if=/uefi_signed/tegra234-p3701-0004-p3737-0000.dtb of=/dev/mmcblk0p6 bs=64k
	```
6. To write the signed recovery.img to `recovery` partition:
	```
	### Ex: recovery partition is mapped to /dev/mmcblk0p8
	$ dd if=/uefi_signed/recovery.img of=/dev/mmcblk0p8 bs=64k
	```
7. To write the signed recovery kernel-dtb to `recovery-dtb` partition:
	```
	### Ex: recovery-dtb partition is mapped to /dev/mmcblk0p9
	$ dd if=/uefi_signed/tegra234-p3701-0004-p3737-0000.dtb.rec of=/dev/mmcblk0p9 bs=64k
	```

##### Trigger a Capsule Update

To trigger a Capsule update, complete the steps in [Manually Trigger the Capsule Update](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Bootloader/UpdateAndRedundancy.html#sd-bootloader-updateandredundancy-manuallytriggerthecapsuleupdate)

> [!note] Note
> After the Capsule update is complete, the system boots from the newly updated slot.

##### Check the Enrolled Keys

After the Capsule update is complete, check the enrolled keys:

1. To install `mokutil` on the target device, run the following commands:
	```
	$ sudo apt-get update
	$ sudo apt-get install mokutil
	```
2. To check the UEFI Secure Boot status, run the following command:
	```
	$ mokutil --sb-state
	```

> [!note] Note
> The command should output “SecureBoot enabled”.

3. To check the enrolled PK, run the following command:
	```
	$ mokutil --pk
	```

> [!note] Note
> The `PK.crt` file is in the output key list.

4. To check the enrolled KEK, run the following command:
	```
	$ mokutil --kek
	```

> [!note] Note
> The `KEK.crt` file is in the output key list.

5. To check the enrolled db, run the following command:
	```
	$ mokutil --db
	```

> [!note] Note
> The `db_1.crt` and `db_2.crt` files are in the output key list.

#### Method Three: Enable UEFI Secure Boot Using UEFI Utilities from an Ubuntu Prompt

This section is for the targets that were not flashed with UEFI Secure Boot enabled.

> [!note] Note
> We recommend using this method for development only. For production devices, use Method One: Enable UEFI Secure Boot at Flashing Time.

##### Prerequisites

- Install the UEFI utilities: `efitools` and `efivar`:
	```
	$ apt update
	$ apt install efitools
	$ apt install efivar
	```
- Ensure that Secure Boot is not enabled:
	```
	$ efivar -n 8be4df61-93ca-11d2-aa0d-00e098032b8c-SecureBoot
	```

> [!note] Note
> The above command should return with a value of 0. If it returns with a value of 1, you cannot continue.

##### Generate the Auth files

The PK, KEK, and db auth files are used to enroll the PK, KEK and db keys from target when enabling UEFI Secure Boot through UEFI utilities running from an Ubuntu prompt.

Run the following commands:

```
$ cd <LDK_DIR>/uefi_keys/

### Generate PK Auth File
$ sign-efi-sig-list -k PK.key -c PK.crt PK PK.esl PK.auth

### Generate KEK Auth File
$ sign-efi-sig-list -k PK.key -c PK.crt KEK KEK.esl KEK.auth

### Generate db Auth Files
$ cat db_1.esl db_2.esl > db.esl
$ sign-efi-sig-list -k KEK.key -c KEK.crt db db.esl db.auth
```

##### Generate the Signed UEFI Payloads

For Jetson Thor, refer to.

For Jetson Orin, refer to.

##### Download and Enroll the Secure Boot Artifacts Using the Ubuntu Prompt

This section describes the following steps to be performed on the target:

1. Download the PK, KEK, and db auth files from the host.
2. Enroll the KEK and db keys.
3. Download and write the signed UEFI payloads.
4. Enroll the PK key.

1. Download the PK, KEK, and db auth files from the host.
	To get the PK, KEK, and db auth files, run the following commands:
	```
	$ mkdir /uefi_keys
	$ cd /uefi_keys
	$ scp <host_ip>:<LDK_DIR>/uefi_keys/*.auth .
	```
2. Enroll the KEK and db keys on the target.
	To enroll the KEK and db, run the following commands:
	```
	$ efi-updatevar -f /uefi_keys/db.auth db
	$ efi-updatevar -f /uefi_keys/KEK.auth KEK
	```
3. Download and write the signed UEFI payloads.
	To download and write the signed UEFI payloads:
	- For Jetson Thor, refer to.
		- For Jetson Orin, refer to.
4. Enroll the PK key.
	To enroll the PK key last to enable UEFI Secure Boot:
	```
	$ efi-updatevar -f /uefi_keys/PK.auth PK
	```

##### Check If UEFI Secure Boot Is Enabled

Reboot the target and run the `efivar` command to verify:

```
$ efivar -n 8be4df61-93ca-11d2-aa0d-00e098032b8c-SecureBoot
```

> [!note] Note
> The command above should return with a value of 01.

### Update the db/dbx Keys with a Capsule Update

This section provides information about how to use Capsule to update the KEK, the db, and the dbx keys after enabling the UEFI Secure Boot. Here is the high-level process to update UEFI Secure Boot keys:

1. Prepare the update keys.
	1. Generate the KEK, the db, and the dbx keys auth file for update.
		2. Create a UEFI update keys config file with the generated keys auth file.
		3. Generate the `UefiUpdateSecurityKeys.dtbo` file.
2. Generate the Capsule payload with UEFI Secure Boot enabled.
3. Trigger a Capsule update.
4. Check and verify update keys.
	1. Check the UEFI Secure Boot status.
		2. Check the updated KEK.
		3. Check the updated db.
		4. Check the updated dbx.

When UEFI Secure Boot is enabled, you can use Capsule update to update the KEK, the db, and the dbx keys. Refer to [Generating the Capsule Update Payload](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Bootloader/UpdateAndRedundancy.html#sd-bootloader-updateandredundancy-generatingthecapsuleupdatepayload) for more information about Capsule update.

#### Prepare the Update Key Auth Files

You must provide the KEK/db/dbx keys certificates in signed the esl (.auth) format, and you can update one, two, or all three key types.

The next section is an example that shows you how to generate self-signed certificates to test updates to the all three types of keys.

> [!note] Note
> In a production environment, complete your official certificate generation procedure.

##### Generate the KEK, the db, and the dbx Key Auth Files for an Update

To generate the KEK, the db, and the dbx key auth files:

1. Run the following commands to prepare to generate the update keys:
	```
	$ cd to <LDK_DIR>/uefi_keys
	$ GUID=$(uuidgen)
	```
2. Run the following commands to generate the KEK RSA keypair and certificate for the update:
	```
	$ openssl req -newkey rsa:2048 -nodes -keyout update_kek_0.key  -new -x509 -sha256 -days 3650 -subj "/CN=Update KEK 0/" -out update_kek_0.crt
	$ cert-to-efi-sig-list -g "${GUID}" update_kek_0.crt update_kek_0.esl
	$ sign-efi-sig-list -a -k PK.key -c PK.crt KEK update_kek_0.esl update_kek_0.auth
	```

> [!note] Note
> - This step is needed only when a KEK update is required.
> - The PK.key and PK.crt are the PK private key and PK certificate that were generated when you enrolled the default keys in.

3. Run the following commands to generate the db RSA keypair and certificate for the update:
	```
	$ openssl req -newkey rsa:2048 -nodes -keyout update_db_0.key  -new -x509 -sha256 -days 3650 -subj "/CN=Update DB 0/" -out update_db_0.crt
	$ cert-to-efi-sig-list -g "${GUID}" update_db_0.crt update_db_0.esl
	$ sign-efi-sig-list -a -k update_kek_0.key -c update_kek_0.crt db update_db_0.esl update_db_0.auth
	```

> [!note] Note
> The signing private key (`update_kek_0.key`) and the certificate (`update_kek_0.crt`) are generated by running the previous command. They can also be the KEK private key and certificate when you enroll the default keys in.

4. Run the following commands to generate another db RSA keypair and certificate for the update:
	```
	$ openssl req -newkey rsa:2048 -nodes -keyout update_db_1.key  -new -x509 -sha256 -days 3650 -subj "/CN=update DB 1/" -out update_db_1.crt
	$ cert-to-efi-sig-list -g "${GUID}" update_db_1.crt update_db_1.esl
	$ sign-efi-sig-list -a -k KEK.key -c KEK.crt db update_db_1.esl update_db_1.auth
	```
5. Run the following commands to generate db\_2 auth for the dbx update:
	```
	$ cert-to-efi-sig-list -g "${GUID}" db_2.crt db_2.esl
	$ sign-efi-sig-list -a -k update_kek_0.key -c update_kek_0.crt dbx db_2.esl dbx_db_2.auth
	```

> [!note] Note
> The db\_2 certificate is generated when you enroll the default keys in.

> [!caution] Caution
> The generated.crt files are self-signed certificates and are used for demonstration purposes only. In a production environment, complete your official certificate generation procedure.

##### Create a UEFI Update Keys Config File

To create a UEFI keys config file with the generated key auth files:

1. Run the following command to create the `uefi_update_keys.conf` file:
	```
	$ vim uefi_update_keys.conf
	```
2. Add the following lines to the `uefi_update_keys.conf` file:
	```
	UEFI_DB_1_KEY_FILE="update_db_0.key";  # UEFI payload signing key
	UEFI_DB_1_CERT_FILE="update_db_0.crt"; # UEFI payload signing key certificate
	UEFI_UPDATE_PRE_SIGNED_KEK_0="update_kek_0.auth"
	UEFI_UPDATE_PRE_SIGNED_DB_0="update_db_0.auth"
	UEFI_UPDATE_PRE_SIGNED_DB_1="update_db_1.auth"
	UEFI_UPDATE_PRE_SIGNED_DBX_0="dbx_db_2.auth"
	```

> [!note] Note
> - The UEFI\_DB\_1\_KEY\_FILE and UEFI\_DB\_1\_CERT\_FILE are used to sign UEFI payloads such as L4TLauncher, kernel, and kernel-dtb. If the UEFI payloads are resigned with update\_db\_x key, which is shown in this example, you can use the same signing key (db\_1.key and db\_1.crt) that was used when you initially enabled UEFI secure boot or the update key update\_db\_x that was defined in this update key conf.
> - Users can specify up to 50 UEFI\_UPDATE\_PRE\_SIGNED\_KEK\_n, UEFI\_UPDATE\_PRE\_SIGNED\_DBX\_n, or UEFI\_UPDATE\_PRE\_SIGNED\_DB\_n.
> - If the following standard keys are not enrolled yet, they can be used here according to your requirement:
> 	- Microsoft’s certificates (two DB certificates and one KEK certificate). Refer to the [Microsoft’s certificates](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#Microsoft_Windows) for more information.
> 		- The UEFI revocation list file, which is used to update the Secure Boot Forbidden Signature Database (dbx). Download the revocation list file from [UEFI Revocation List File for arm64](https://uefi.org/revocationlistfile). To update the revocation list file to dbx, assign the `arm64_DBXUpdate.bin` file to the UEFI\_UPDATE\_PRE\_SIGNED\_DBX\_n variable in `uefi_update_keys.conf`.

##### Generate the UefiUpdateSecurityKeys.dtbo File

To update the UEFI Secure Boot keys, the update UEFI security keys auth files are embedded in the `UefiUpdateSecurityKeys.dtbo` file, which is generated by using the `gen_uefi_keys_dts.sh` script.

Run the following commands:

```
$ cd ..
$ sudo tools/gen_uefi_keys_dts.sh uefi_keys/uefi_update_keys.conf
$ sudo chmod 644 uefi_keys/*.auth
```

Copy the generated `UefiUpdateSecurityKeys.dtbo` to the bootloader directory:

```
$ cd to <LDK_DIR>
$ cp uefi_keys/UefiUpdateSecurityKeys.dtbo bootloader/
```

#### Generate a Capsule Payload with UEFI Secure Boot Enabled

The `UefiUpdateSecurityKeys.dtbo` generated above is packed into Capsule payload with cpu-bootloader.Refer to [Generating the Capsule Update Payload](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Bootloader/UpdateAndRedundancy.html#sd-bootloader-updateandredundancy-generatingthecapsuleupdatepayload) for more information.

To generate a Capsule payload, complete one of the following tasks:

- Generate a Capsule payload for the Jetson AGX Thor devkits:
	```
	$ sudo ADDITIONAL_DTB_OVERLAY="UefiUpdateSecurityKeys.dtbo" ./l4t_generate_soc_bup.sh -u <pkc_keyfile> [-v <sbk_keyfile>] -e t26x_3834_bl_spec t26x
	$ ./generate_capsule/l4t_generate_soc_capsule.sh -i bootloader/payloads_t26x/bl_only_payload -o ./TEGRA_THOR.Cap t264
	```
- Generate a Capsule payload for the Jetson AGX Orin devkits:
	```
	$ sudo ADDITIONAL_DTB_OVERLAY="UefiUpdateSecurityKeys.dtbo" ./l4t_generate_soc_bup.sh -u <pkc_keyfile> [-v <sbk_keyfile>] -e t23x_agx_bl_spec t23x
	$ ./generate_capsule/l4t_generate_soc_capsule.sh -i bootloader/payloads_t23x/bl_only_payload -o ./TEGRA_AGX.Cap t234
	```
- Generate a Capsule payload for the Jetson AGX Orin Industrial:
	```
	$ sudo ADDITIONAL_DTB_OVERLAY="UefiUpdateSecurityKeys.dtbo" ./l4t_generate_soc_bup.sh -u <pkc_keyfile> [-v <sbk_keyfile>] -e t23x_agx_ind_bl_spec t23x
	$ ./generate_capsule/l4t_generate_soc_capsule.sh -i bootloader/payloads_t23x/bl_only_payload -o ./TEGRA_AGX_IND.Cap t234
	```
- Generate a Capsule payload for the Jetson Orin Nano devkits:
	```
	$ sudo ADDITIONAL_DTB_OVERLAY="UefiUpdateSecurityKeys.dtbo" ./l4t_generate_soc_bup.sh -u <pkc_keyfile> [-v <sbk_keyfile>] -e t23x_3767_bl_spec t23x
	$ ./generate_capsule/l4t_generate_soc_capsule.sh -i bootloader/payloads_t23x/bl_only_payload -o ./TEGRA_Nano.Cap t234
	```

#### Trigger a Capsule Update

To trigger a Capsule update, complete the steps in [Manually Trigger the Capsule Update](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Bootloader/UpdateAndRedundancy.html#sd-bootloader-updateandredundancy-manuallytriggerthecapsuleupdate)

> [!note] Note
> After the Capsule update is complete, the system boots from the newly updated slot.

#### Check and Verify the Update Keys

After the Capsule update is complete, to check and verify the update keys:

1. Run the following commands to install `mokutil` on the target device:
	```
	$ sudo apt-get update
	$ sudo apt-get install mokutil
	```
2. Run the following command to check the UEFI Secure Boot status:
	```
	$ mokutil --sb-state
	```

> [!note] Note
> The command should output “SecureBoot enabled”.

3. Run the following command to check the updated KEK:
	```
	$ mokutil --kek
	```

> [!note] Note
> The `update_kek_0.crt` file is in the output key list.

4. Run the following command to check the updated db:
	```
	$ mokutil --db
	```

> [!note] Note
> The `update_db_0.crt` and `update_db_1.crt` files are in the output key list.

5. Run the following command to check the updated dbx:
	```
	$ mokutil --dbx
	```

> [!note] Note
> The `db_2.crt` file is in the output key list.

## UEFI Payload Encryption

UEFI Payload Encryption encrypts UEFI payloads. This security measure requires the use of a specific UEFI payload encryption key, which is user-defined and stored in the encrypted key blob, then flashed onto the encrypted key store (EKS) partition.

When the system boots into [OP-TEE](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Security/OpTee.html#sd-security-optee), the user key PTA extracts this key from EKB. When the system boots to UEFI, the L4tLauncher (OS Loader) calls the trusted application to decrypt and load the encrypted UEFI payloads.

> [!note] Note
> UEFI Payload Encryption can be enabled only when UEFI Secure Boot is enabled.

The UEFI payloads are:

- initrd
- kernel images in the rootfs and in the kernel and the recovery partitions.
- kernel-dtb images in the rootfs and in the kernel-dtb and the recovery-dtb partitions.

The UEFI Payload Encryption implementation includes the UEFI, user key, and the TA:

- UEFI: Call TA to decrypt and authenticate UEFI payloads and aborts the boot on error.
- User Key: A user-defined UEFI payload encryption key that is stored in EKB.
- Trusted Application (TA): Decrypt and authenticate the UEFI payloads using the UEFI payload encryption key.

To activate UEFI Payload Encryption, create a unique user key, generate customer EKB, and enable UEFI Payload Encryption during the flashing process.

The following flow chart illustrates how the encrypted payloads are decrypted and loaded:

![암호화된 페이로드가 로드되는 방식](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/_images/UEFI_PayloadDecrypt.png)

암호화된 페이로드가 로드되는 방식

### Prepare the User Encryption Key

1. Generate a random UEFI payload encryption key (256 bits log) using a random number generator.
2. Save the output to `user_encryption.key` in big-endian hex format.

### Generate the EKB

1. Generate the EKB (refer to [EKB Generation](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Security/OpTee.html#sd-security-optee-ekb-ekbgeneration) for more information).
2. Copy the EKB to the `<Linux For Tegra>/bootloader` folder.

> [!note] Note
> L4TLauncher cannot detect whether UEFI payload encryption is enabled. However, if the EKB contains the UEFI payload encryption key and UEFI secure boot is enabled, L4TLauncher will assume that UEFI payload encryption is enabled and will attempt to decrypt the UEFI payloads.

### Enable UEFI Payload Encryption During the Flashing Process

The `--uefi-enc <user_encryption.key>` option is used to provide the user encryption key and enable UEFI Payload Encryption.

To enable UEFI Payload Encryption, you must simultaneously enable UEFI secure boot. In this condition, the `---uefi-keys` and the `--uefi-enc` option are specified, and the flashing utility will generate the signed and encrypted UEFI payloads and flash them to board.

#### Using the --uefi-enc <user\_encryption.key> Option to Provide the User Encryption Key and Enable UEFI Payloads Encryption

> [!note] Note
> Although UEFI secure boot can be enabled separately from the low-level bootloader secure boot, we strongly recommend enabling bootloader secure boot to ensure the root-of-trust begins at the BootROM.

1. Issue the following command with the `--uefi-enc <user_encryption.key>` option:
	- For the Jetson AGX Orin series:
		```
		$ sudo ./flash.sh -u <pkc_keyfile> [-v <sbk_keyfile>] --uefi-keys uefi_keys/uefi_keys.conf --uefi-enc user_encryption.key <target> internal
		```
		Where `<target>` is one of the following options:
		> - For the Jetson AGX Orin: `jetson-agx-orin-devkit`
		- For the Jetson Orin NX series and the Orin Nano series:
		```
		$ sudo ./l4t_initrd_flash.sh -u <pkc_keyfile> [-v <sbk_keyfile>] --uefi-keys uefi_keys/uefi_keys.conf --uefi-enc <user_encryption.key> jetson-orin-nano-devkit external
		```

where *<user\_encryption.key>* is the pathname to a file that contains the user encryption key in the <Linux\_for\_Tegra>/ folder.

For more information about `pkc_keyfile` and `sbk_keyfile`, refer to.

After flashing is complete, your target will have UEFI Secure Boot and UEFI Payloads Encryption enabled.

## UEFI Variable Protection

UEFI Variable Protection secures UEFI variables against tampering. This security measure requires the use of a specific UEFI variable authentication key, which is user-defined and stored in the EKB then flashed onto EKS partition.

When the system boots into OP-TEE, the user key PTA extracts this key from EKB. When the system boots into UEFI, UEFI will call the TA to use the UEFI variable authentication key for calculating a measurement that verifies the integrity of UEFI variables. As a result of this process, any tampering with UEFI variables is detectable.

The UEFI Variable Protection implementation includes the UEFI, user key, and the TA:

- UEFI: Compares the measurements and aborts the boot if an attack is detected.
- User Key: A user-defined UEFI variable authentication key that is stored in EKB.
- Trusted Application (TA): Calculates measurements against the UEFI variables using the UEFI variable authentication key.

To activate UEFI Variable Protection, create a unique user key, generate a custom EKB, and enable UEFI Variable Protection during the flashing process.

### Prepare the UEFI Variable Authentication Key

1. Generate a random UEFI variable authentication key (128 bits long) with random number generator.
2. Save the output to `user_authentication.key` in big-endian hex format.

### Generate the EKB

1. Generate the EKB (refer to [EKB Generation](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/Security/OpTee.html#sd-security-optee-ekb-ekbgeneration) for more information).
2. Copy the EKB to the `<Linux_for_Tegra>/bootloader` folder.

### Enable UEFI Variable Protection During the Flashing Process

1. Issue the following command:
	- For the Jetson AGX Orin series:
		```
		$ sudo ./flash.sh -u <pkc_keyfile> [-v <sbk_keyfile>] --uefi-keys uefi_keys/uefi_keys.conf <target> mmcblk0p1
		```
		- For the Jetson Orin NX series and the Orin Nano series:
		```
		$ sudo ./l4t_initrd_flash.sh -u <pkc_keyfile> [-v <sbk_keyfile>] --uefi-keys uefi_keys/uefi_keys.conf jetson-orin-nano-devkit external
		```

For more information about `pkc_keyfile` and `sbk_keyfile`, refer to.

After flashing is complete, your target will have UEFI Secure Boot and UEFI Variable Protection enabled.

## UEFI Platform Vendor Key Feature

The UEFI platform vendor (PV) key feature allows PVs to deploy UEFI that is signed and encrypted by PV-owned keys without involving the solution providers.

NVIDIA® Jetson™ devices use a PKC key to verify the signature of boot component during device boot:

- The component at stage N verifies the components at stage N+1.
- The component at stage N+1 verifies the components at stage N+2, and so on.

With this feature, the UEFI owner, or the PV, can use a PV’s private key to sign UEFI, and can also deliver the public key to the solution provider, who was the owner of the boot components before UEFI. The public key is now built into the component (MB2) that loads in UEFI during boot.

During the secure boot process, the signature of the components before UEFI will be verified/authenticated by solution provider’s fused PKC. For UEFI, MB2 uses the built-in public key to verify PV authenticate key for key verification and signature authentication. As a result, the platform vendor, who does not own the fused PKC, can still independently sign and update the UEFI image.

![Secureboot에서 PV 키는 어떻게 사용되나요?](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/_images/Chain_of_trust_and_key.jpg)

Secureboot에서 PV 키는 어떻게 사용되나요?

In Jetson devices that use the T234 processor (NVIDIA® Jetson Orin™ NX series and NVIDIA® Jetson AGX Orin™ series), the PV key sign/authenticate UEFI is supported when secure boot is enabled.

### Platform Vendor Key Signed/Authenticate UEFI

This section describes how to use a tool to sign UEFI by the PV and to authenticate it by the solution provider.

The PV can sign UEFI by the key it owns. Here is a high-level overview of the process:

1. The PV provides the PV authentication key file to the solution provider.
2. The solution provider builds boot images with the PV authentication key that is built into MB2.
3. The solution provider sends the results from step 2 to the PV.
4. The PV creates the combined boot images and completes one of the following tasks:
	- Sends the images to the factory floor to build the device.
		- Sends the images to the OTA server for the update.

Before you enable UEFI PV key sign/authenticate feature, the RSA 3K authentication scheme fuse must be burned.

> [!note] Note
> - RSA-3072, ECDSA-P256, ECDSA-P521 are supported for UEFI PV key signing.
> - The cryptography algorithm of the UEFI PV key should be aligned with the PKC key used for signing low-level boot components.

#### Platform Vendor Procedure

To generate the PV authentication key and sign UEFI:

1. Generate the PV key pair and call it `pv_priv.pem`.:
	- To generate an ECDSA P-256 key:
		```
		$ openssl ecparam -name prime256v1 -genkey -noout -out pv_priv.pem
		```
		- To generate an ECDSA P-521 key:
		```
		$ openssl ecparam -name secp521r1 -genkey -noout -out pv_priv.pem
		```
		- To generate an RSA-3K key:
		```
		$ openssl genrsa -out pv_priv.pem 3072
		```
2. Create a certificate signing request and call it `pub_key.csr`.:
	```
	$ openssl req -key pv_priv.pem -new -out pub_key.csr
	```
3. Create a self-signed certificate and call it `pv_key.crt`.:
	```
	$ openssl x509 -signkey pv_priv.pem -in pub_key.csr -req -days 3650 -out pv_key.crt
	```
4. Provide the `pv_key.crt` file to the solution provider.
5. Get `bootloader.tar.gz` from the solution provider, place it in the Linux\_for\_Tegra/ folder, and untar it.:
	```
	$ tar -xzvf bootloader.tar.gz
	```
6. Gather the BOARDID, FAB, BOARDSKU, and BOARDREV environment information from the solution provider.
7. Generate the signed UEFI image with the `pv_priv.pem` PV private key.:
	```
	$ sudo BOARDID=<board_id> FAB=<fab> BOARDSKU=<sku_number> BOARDREV=<reversion> ./flash.sh --no-flash -u pv_priv.pem -k A_cpu-bootloader jetson-agx-orin-devkit internal
	```
8. Flash the device with the following commands:
	```
	$ boardctl -t topo recovery
	$ cd bootloader/
	$ sudo bash ./flashcmd.txt
	```

#### Solution Provider Procedure

After the solution provider receives the PV authentication key, to enable the PV sign/authenticate UEFI feature, complete the following steps:

1. Place the `pv_key.crt` file in the Linux\_for\_Tegra/ folder.
2. Generate the signed boot images with the board-fused PKC key file (`pkc.pem`) and the PV key certificate (`pv_key.crt`) file.:
	```
	$ sudo BOARDID=<board_id> FAB=<fab> BOARDSKU=<sku_number> BOARDREV=<reversion> ./flash.sh --no-flash -u pkc.pem --pv-crt pv_key.crt jetson-agx-orin-devkit internal
	```
3. Package Linux\_for\_Tegra/bootloader/ folder.:
	```
	$ tar -czvf bootloader.tar.gz ./bootloader
	```
4. Send the `bootloader.tar.gz` file to the PV.

### PV Key Encrypt/Decrypt UEFI

This section describes how to use the tool to encrypt UEFI by using a key owned by the PV and to decrypt it by the solution provider at MB2.

The mechanism is that the key encrypts UEFI is the PV-owned encryption key instead of the default SBK key. The PV will encrypt UEFI with its owned PV encryption key in one of the following ways:

> - The PV provides a fuse blob so that the PV encryption key can be burned into the fuse.
> - The PV provides the PV encryption key to the solution provider who can then inject the key into MB2 by using the tool.

Before enabling the UEFI PV key encrypt/decrypt feature, the SBK key and the encryption mode fuse must be burned. Refer to for more information.

### Fuse Solution

To provide a higher level of protection to the PV encryption key, the PV can burn the PV encryption key into the OEM\_K2 fuse. Now, MB2 can decrypt PV encrypted UEFI by using OEM\_K2 through Security Engine without knowing the actual key, and here are the decryption steps:

1. The PV generates a fuse blob, which includes the PV encryption key, and that will be burned into OEM\_K2 fuse.
2. The PV encrypts the UEFI image with the PV encryption key.
3. The solution provider generates its own fuse blob that includes the SBK key fuse and the encryption mode enable fuse.
4. The solution provider builds the boot images with the SBK key.
5. The solution provider sends the results from steps 3 and 4 to the PV.
6. The PV generates UEFI image, combines the boot images generated by the solution provider and the encrypted UEFI image generated by PV, and completes one of the following tasks:

> - Sends the images to the factory floor to build the device.
> - Sends the images to the OTA server for the OTA update.

> [!note] Note
> To increase security level, the PV encryption key in this procedure is used to derive a final key for UEFI image encryption and decryption.

#### Platform Vendor Procedure

This procedure generates the PV encryption key, generates the signing PV key, and encrypts and signs UEFI:

1. Generate the AES-256 key and call it `pv_enc.key`.:
	```
	$ openssl rand -hex 32 > pv_enc.key
	```
2. Manually reformat `pv_enc.key` into eight 32-bit words big-endian hexadecimal.
	For example, original content of `pv_enc.key` is `112233445566778899aabbccddeeff00ffeeddccbbaa99887766554433221100`, and after the reformat, it changes to `0x11223344 0x55667788 0x99aabbcc 0xddeeff00 0xffeeddcc 0xbbaa9988 0x77665544 0x33221100`.
3. Generate the PV private key and call it `pv_priv.pem`.:
	- To generate an ECDSA P-256 key:
		```
		$ openssl ecparam -name prime256v1 -genkey -noout -out pv_priv.pem
		```
		- To generate an ECDSA P-521 key:
		```
		$ openssl ecparam -name secp521r1 -genkey -noout -out pv_priv.pem
		```
		- To generate an RSA-3K key:
		```
		$ openssl genrsa -out pv_priv.pem 3072
		```
	> [!note] Note
	> If the PV signing key and its certificates have been previously generated, you can skip this step.
4. Create a certificate signing request and call it `pub_key.csr`.:
	```
	$ openssl req -key pv_priv.pem -new -out pub_key.csr
	```
5. Create a self-signed certificate and call it `pv_key.crt`.:
	```
	$ openssl x509 -signkey pv_priv.pem -in pub_key.csr -req -days 3650 -out pv_key.crt
	```
6. Provide the `pv_key.crt` file to the solution provider.
7. Get `bootloader.tar.gz` from the solution provider, place it in the Linux\_for\_Tegra/, and untar it.:
	```
	$ tar -xzvf bootloader.tar.gz
	```
8. Gather the BOARDID, FAB, BOARDSKU, and BOARDREV environment information from the solution provider.
9. Generate the signed and encrypted UEFI image with the PV private key (`pv_priv.pem`) and the PV encryption key (`pv_enc.key`).:
	```
	$ sudo BOARDID=<board_id> FAB=<fab> BOARDSKU=<sku_number> BOARDREV=<reversion> ./flash.sh --no-flash -u pv_priv.pem --pv-enc pv_enc.key -k A_cpu-bootloader jetson-agx-orin-devkit internal
	```
10. Get the fuse blob from the solution provider and provide the solution provider’s fuse blob and the PV’s fuse blob to the factory to burn fuses.
11. Burn the fuse.
12. Run the following commands to flash the device:
	```
	$ boardctl -t topo recovery
	$ cd bootloader/
	$ sudo bash ./flashcmd.txt
	```

#### Solution Provider Procedure

After the solution provider receives the PV authentication key, complete the following steps:

1. Place the `pv_key.crt` file in the same folder as the flash.sh script.
2. Generate the signed and encrypted boot images by using the PKC key (`pkc.pem`), the SBK key, and the PV authentication key certificate (`pv_key.crt`):
	```
	$ sudo BOARDID=<board_id> FAB=<fab> BOARDSKU=<sku_number> BOARDREV=<reversion> ./flash.sh --no-flash -u pkc.pem -v <sbk.key> --pv-crt pv_key.crt jetson-agx-orin-devkit internal
	```
3. Package the Linux\_for\_Tegra/bootloader/ folder.:
	```
	$ tar -czvf bootloader.tar.gz ./bootloader
	```
4. Send the `bootloader.tar.gz` file to the PV.

#### Fuse handling

Before you execute the above flashing command, the device fuses must be burned with the fuse blobs that are provided by the PV and the solution provider.

1. Here is an example of a fuse blob (1) provided by the PV:
	```
	<genericfuse MagicId="0x45535546" version="1.0.0">
	    <fuse name="PscOdmStatic" size="4" value="0x00000060"/>
	    <fuse name="OemK2" size="32" value="0x112233445566778899aabbccddeeff00ffeeddccbbaa99887766554433221100"/>
	</genericfuse>
	```
2. Here is an example of a fuse blob (2) provided by the solution provider:
	```
	<genericfuse MagicId="0x45535546" version="1.0.0">
	    <fuse name="PscOdmStatic" size="4" value="0x00000060"/>
	    <fuse name="OemK1" size="32" value="0xf3bedbff9cea44c05b08124e8242a71ec1871d55ef4841eb4e59a56b5f88fb2b"/>
	    <fuse name="PublicKeyHash" size="64" value="0xdc6632e495c7976659a94668a98d6ba7e22a2d9438a555ec64c0c1cc59e533067bfe64f454c1f30c63ad7627fb0cfa2f556aff45818254387016745ccf713081"/>
	    <fuse name="SecureBootKey" size="32" value="0x123456789abcdef0fedcba987654321023456789abcdef01edcba9876543210f"/>
	    <fuse name="BootSecurityInfo" size="4" value="0x209"/>
	    <fuse name="SecurityMode" size="4" value="0x1"/>
	</genericfuse>
	```

You must **ensure** that in the fuse burning sequence, the fuse blob 1 burned first.

> [!note] Note
> - The fuses specified in the PV’s fuse blob and the fuses specified in solution provider cannot be overlapped.
> - The fuse blob provided by the PV must be burned first because, after the “SecurityMode” that is specified in the solution provider’s fuse blob is burned, fuse burning except ODM fuses is blocked.
> - The above fuse values in both PV and solution provider’s fuse blobs are for demonstrations only.

## Kernel Module Signing

The kernel module signing facility signs modules during installation and then checks the signature upon loading the module. This allows increased kernel security by disallowing the loading of unsigned modules or modules that were signed with an invalid key.

Here are the kernel configure options for kernel module signing:

- To enable kernel module signature verification, in the **Enable Loadable Module Support** section, enable `CONFIG_MODULE_SIG`.
- To select the kernel module signature verification mode, set the `CONFIG_MODULE_SIG_FORCE` to one of the following options:
	> - `off`: permissive mode.
	> 	- If the module is signed, it must have a trusted signature.
	> 		- if the module is not signed, it can be loaded, and the kernel is marked as tainted.
	> - `on`: restrictive mode.
	> 	- Modules can only be loaded if they are signed with a trusted signature.
	> 		- The other modules will generate an error.
- To enable automatic kernel module signing at build time, set the `CONFIG_MODULE_SIG_ALL`.

> [!note] Note
> 기본적으로 커널 모듈 서명 검증이 활성화되어 있더라도 빌드 시점에 커널 모듈에 서명이 적용되지 않습니다.

- 서명 키를 지정하려면 `CONFIG_MODULE_SIG_KEY` PEM 형식의 개인 키를 설정하십시오.
	> - 기본적으로 CONFIG\_MODULE\_SIG\_KEY=”certs/signing\_key.pem” 설정이 변경되지 않으면 커널은 커널 모듈 서명을 위해 PEM 형식의 서명 키를 자동으로 생성합니다.
	> - `CONFIG_MODULE_SIG_KEY` 기본값 이외의 값으로 설정하면 `certs/signing_key.pem` 서명 키의 자동 생성이 비활성화되고 사용자가 선택한 키로 커널 모듈에 서명할 수 있습니다.

> [!note] 메모
> 커널 `CONFIG_SYSTEM_TRUSTED_KEYS` 옵션은 추가 인증서가 포함된 PEM 인코딩 파일의 파일 이름으로 설정할 수도 있습니다. 이 파일은 커널에 컴파일되어 커널 빌드 시 서명되지 않은 모듈의 커널 모듈 검증에 사용되는 X.509 인증서입니다. 자세한 내용은 [커널 모듈 서명 기능을](https://www.kernel.org/doc/Documentation/admin-guide/module-signing.rst) 참조하십시오.