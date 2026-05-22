---
title: "01 임베디드 플랫폼 : CPU"
source: "https://devsophia.tistory.com/entry/01-%EC%9E%84%EB%B2%A0%EB%94%94%EB%93%9C-%ED%94%8C%EB%9E%AB%ED%8F%BC-CPU"
author:
  - "[[devsophia]]"
published: 2019-05-03
created: 2026-05-21
description: "임베디드 플랫폼 임베디드 플랫폼 = 하드웨어(CPU) + 소프트웨어(Firmware or OS) + 개발환경(Tool) 임베디드 플랫폼은 원하는 제품을 완성하기 위해 비용중심으로 위의 세 가지를 적절히 갖춰야 한다. 선택하는 CPU에 따라 Firmware형태의 소프트웨어를 사용할지, 운영체제를 사용할지도 달라지고, 운영체제를 사용한다고 하더라도 실시간 운영체제(RTOS)를 사용할지 Non-RTOS를 사용할지도 달라진다. 그렇다면 임베디드 시스템에 들어가는 소프트웨어는 어떤 기준에 따라 선택해야 할까 ? 기준 소프트웨어 종류 Task 수가 많지 않은 경우 Task들 간에 자원을 공유하지 않는 경우 Task들 사이에 우선순위가 필요없는 경우 Firmware Task의 수는 적지만 우선순위가 보장되어야 하는 .."
tags:
  - "clippings"
---
## 임베디드 플랫폼

임베디드 플랫폼 = 하드웨어(CPU) + 소프트웨어(Firmware or OS) + 개발환경(Tool)

임베디드 플랫폼은 원하는 제품을 완성하기 위해 비용중심으로 위의 세 가지를 적절히 갖춰야 한다.

선택하는 CPU에 따라 Firmware형태의 소프트웨어를 사용할지, 운영체제를 사용할지도 달라지고, 운영체제를 사용한다고 하더라도 실시간 운영체제(RTOS)를 사용할지 Non-RTOS를 사용할지도 달라진다.

그렇다면 임베디드 시스템에 들어가는 소프트웨어는 어떤 기준에 따라 선택해야 할까?

| 기준 | 소프트웨어 종류 |
| --- | --- |
| - Task 수가 많지 않은 경우 - Task들 간에 자원을 공유하지 않는 경우 - Task들 사이에 우선순위가 필요없는 경우 | Firmware |
| - Task의 수는 적지만 우선순위가 보장되어야 하는 경우 | RTOS |
| - 같은 자원에 여러 개의 Task가 접근해야 하는 경우 | Non-RTOS |

## CPU

CPU(마이크로프로세서)는 CPU Core와 CPU Peripherals로 이루어진다.

CPU = CPU Core + CPU Peripherals

여기서 CPU Peripherals는 주변장치 Controller라고 부르기도 한다.

USB Controller, Ethernet MAC Controller 등의 컨트롤러는 USB와 이더넷과 같은 주변장치들과 CPU Core 사이를 연결하는 역할을 한다. 즉, CPU의 연산 결과로 주변 하드웨어를 제어하는 일을 하는것이 CPU Peripheral이다.

임베디드 시스템을 설계할 때 하드웨어적인 관점에서 생각해보자면, 여러 주변장치들을 연결할 때 회로를 따로 구성하지 않고도 칩 내에 회로 구성이 까다로운 주변장치에 대한 Controller가 내장되어 있다면 비용을 절감할 수 있을것이다.

소프트웨어적인 관점에서 생각해보자면, CPU가 몇 bit 프로세서인지, 즉 어느 정도의 연산 처리 능력을 가지고 있는지가 중요하다. 예를 들어 임베디드 리눅스 OS를 사용하려고 하면 CPU는 32bit 연산을 지원하는 Core를 가지고 있어야 한다.

이렇듯 임베디드 시스템에서 CPU를 선정할 때 하드웨어적인 관점에서는 CPU Peripheral, 소프트웨어적인 관점에서는 CPU Core를 고려하여 선정하여야 한다.

추가적으로 사람인과 같은 구직사이트를 보면 MPU와 MCU라는 단어가 명확한 구분없이 사용되는 경우가 있는데 이 둘의 차이점은 무엇일까?

MPU (Micro Processor Unit)은 CPU Core가 중심이 되어 CPU의 연산처리능력이 주를 이루는 CPU를 의미하고, MCU (Micro Controller Unit)은 CPU Peripheral이 중심이 되어 CPU Core와 주변장치들간의 회로가 CPU내부에 잘 설계되어있는 CPU를 의미한다.

그렇다면 CPU Core로 하여금 주변장치(하드웨어)를 제어하게 하려면 어떻게 해야할까?

두괄식으로 말하자면 C언어와 같은 프로그래밍 언어를 사용하여 레지스터의 값을 설정하는 것이다.

주변장치 내에 레지스터와 같은 기억소자가 존재할수도 있고, 없을 수도 있다.

있는 경우에는 주변장치 내의 레지스터를 직접 설정하면 되지만, 없는 경우에는 기억소자를 갖춘 하드웨어(CPU Peripheral의 I/O Controller와 같은)에 연결하여 해당 레지스터를 설정해 주면 된다.

즉, 임베디드 소프트웨어란 CPU Peripheral 내에 존재하는 특정 레지스터들의 값을 설정하는 실행 코드이다.

## 참조

유명환 / Fun + Fun 뻔뻔하게 배우는 임베디드 리눅스 / (주)지앤선 / 2010년

<iframe width="660" height="280" frameborder="0" allow="attribution-reporting; run-ad-auction" src="https://googleads.g.doubleclick.net/pagead/ads?client=ca-pub-9527582522912841&amp;output=html&amp;h=280&amp;slotname=4947159016&amp;adk=1112192385&amp;adf=1035773254&amp;pi=t.ma~as.4947159016&amp;w=660&amp;fwrn=4&amp;fwrnh=100&amp;lmt=1779326098&amp;rafmt=1&amp;ad_type=inventory&amp;format=660x280&amp;url=https%3A%2F%2Fdevsophia.tistory.com%2Fentry%2F01-%25EC%259E%2584%25EB%25B2%25A0%25EB%2594%2594%25EB%2593%259C-%25ED%2594%258C%25EB%259E%25AB%25ED%258F%25BC-CPU&amp;host=ca-host-pub-9691043933427338&amp;fwr=0&amp;fwrattr=true&amp;rpe=1&amp;resp_fmts=3&amp;asro=0&amp;aiactd=0&amp;aicctd=0&amp;ailctd=0&amp;aimartd=4&amp;aieuf=1&amp;aicrs=1&amp;uach=WyJMaW51eCIsIjYuOC4wIiwieDg2IiwiIiwiMTQyLjAuNzQ0NC41OSIsbnVsbCwwLG51bGwsIjY0IixbWyJDaHJvbWl1bSIsIjE0Mi4wLjc0NDQuNTkiXSxbIkdvb2dsZSBDaHJvbWUiLCIxNDIuMC43NDQ0LjU5Il0sWyJOb3RfQSBCcmFuZCIsIjk5LjAuMC4wIl1dLDBd&amp;abgtt=6&amp;dt=1779326098743&amp;bpp=2&amp;bdt=416&amp;idt=252&amp;shv=r20260519&amp;mjsv=m202605150101&amp;ptt=9&amp;saldr=aa&amp;abxe=1&amp;cookie=ID%3D958dbffd7557cd7e%3AT%3D1747355653%3ART%3D1778808765%3AS%3DALNI_MbpmEo8WncVj_4pmTzqBf7hLwerLA&amp;gpic=UID%3D000010c326f9f405%3AT%3D1747355653%3ART%3D1778808765%3AS%3DALNI_MYLqVoGi-JhCjzBaWoZFLSYX-7__Q&amp;eoidce=1&amp;prev_fmts=0x0&amp;nras=1&amp;correlator=8774449293249&amp;frm=20&amp;pv=1&amp;u_tz=540&amp;u_his=1&amp;u_h=1080&amp;u_w=1920&amp;u_ah=1048&amp;u_aw=1854&amp;u_cd=24&amp;u_sd=1&amp;dmc=8&amp;adx=590&amp;ady=2569&amp;biw=1839&amp;bih=927&amp;scr_x=0&amp;scr_y=0&amp;eid=31098638%2C42533293%2C95390680&amp;oid=2&amp;pvsid=7039209489163365&amp;tmod=1477919587&amp;uas=0&amp;nvt=1&amp;fc=1920&amp;brdim=66%2C32%2C66%2C32%2C1854%2C32%2C1854%2C1048%2C1854%2C927&amp;vis=1&amp;rsz=%7C%7CopeEbr%7C&amp;abl=CS&amp;pfx=0&amp;fu=128&amp;bc=31&amp;plas=465x728_l%7C576x728_r&amp;bz=1&amp;pgls=CAk.&amp;ifi=2&amp;uci=a!2&amp;btvi=1&amp;fsb=1&amp;dtd=261" title="Advertisement" aria-label="Advertisement"></iframe>