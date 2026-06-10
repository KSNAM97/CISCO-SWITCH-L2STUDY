# Cisco Switch L2 Study

![Cisco](https://img.shields.io/badge/Cisco-IOS-blue)
![Switch](https://img.shields.io/badge/Layer2-Switch-green)
![GNS3](https://img.shields.io/badge/Lab-GNS3-orange)
![License](https://img.shields.io/badge/License-MIT-yellow)

**Cisco Layer 2 Switch 학습 저장소 (VLAN / Trunk / VTP / STP / RSTP / MSTP)**

---

## 📌 학습 개요

Cisco IOS 기반 **L2 Switch**의 핵심 개념과 실습을 정리한 저장소입니다.  
Hub / Switch / Router의 차이, 스위치의 동작 원리부터 VLAN, Trunk, VTP, STP(PVST+),  
그리고 RSTP·MSTP 및 STP 가속 기능(PortFast / UplinkFast / BackboneFast)까지  
**CCNA / CCNP 수준**의 실무 스위칭 지식을 다룹니다.

| 항목 | 내용 |
| --- | --- |
| 대상 장비 | Cisco IOS (IOU / IOSvL2 / 2960·3560 계열) |
| 학습 영역 | Switch 기본 동작, VLAN, Trunk(802.1Q), VTP, STP(PVST+), RSTP(802.1w), MSTP(802.1s) |
| 실습 환경 | GNS3 / Cisco Packet Tracer |
| 핵심 키워드 | Flooding, Learning, Forwarding, Filtering, Aging, DTP, 802.1Q, Root Bridge, Convergence, Proposal/Agreement |

---

## 🗂 디렉터리 구조

```
cisco-switch-l2study/
├── 01_switch-basics/      # Hub vs Switch vs Router, 스위치 5대 동작
├── 02_vlan/               # VLAN 개념, 생성/삭제, Range 명령어
├── 03_trunk/              # Switchport Mode, 802.1Q Tagging, Router-on-a-Stick
├── 04_vtp/                # VLAN Trunking Protocol (Server/Client/Transparent)
├── 05_stp/                # Spanning-Tree (PVST+), Root Bridge, Port Role
├── 06_stp-port-states/    # STP 5가지 Port 상태, Convergence 타이머
├── 07_portfast/           # PortFast, BPDU Guard, Err-disable 복구
├── 08_uplinkfast/         # UplinkFast (직접 장애 시 Block→Forwarding 즉시 전환)
├── 09_backbonefast/       # BackboneFast (간접 장애 시 Max-age 생략)
├── 10_rstp/               # RSTP (802.1w, Rapid-PVST), Proposal/Agreement
├── 11_mstp/               # MSTP (802.1s), MST Region / Instance, 로드 분산
├── labs/                  # 종합 실습 (Preconfig 제공)
│   ├── lab01_vtp-4switch/
│   ├── lab02_stp-blockport/
│   ├── lab03_pvst-loadbalance/
│   ├── lab04_ds-access-design/
│   ├── lab05_portfast-bpduguard/
│   ├── lab06_rstp-convergence/
│   └── lab07_mstp-instance/
│
├── LICENSE
└── README.md
```

---

## 📚 이론 목차

| # | 주제 | 핵심 내용 |
| --- | --- | --- |
| 01 | [Switch 기본](./01_switch-basics/switch-basics.md) | Hub / Switch / Router 비교, Collision/Broadcast Domain, 스위치 5대 동작 |
| 02 | [VLAN](./02_vlan/vlan.md) | Virtual LAN 개념, Standard/Extended VLAN, VLAN 생성/삭제, `range` |
| 03 | [Trunk](./03_trunk/trunk.md) | Switchport Mode, IEEE 802.1Q Tagging, Router-on-a-Stick, `allowed vlan` |
| 04 | [VTP](./04_vtp/vtp.md) | VTP Domain/Password/Mode, Revision Number, VTP Message |
| 05 | [STP](./05_stp/stp.md) | PVST+, Root Bridge 선출, Port Role (DP/RP/AP), Priority·Cost 조정 |
| 06 | [STP Port 상태](./06_stp-port-states/stp-port-states.md) | Disable/Blocking/Listening/Learning/Forwarding, Convergence(30/50초), 타이머 |
| 07 | [PortFast](./07_portfast/portfast.md) | Edge Port 즉시 Forwarding, BPDU Guard, Err-disable & 복구 |
| 08 | [UplinkFast](./08_uplinkfast/uplinkfast.md) | Block Port 보유 SW, RP(직접) 장애 시 즉시 전환 |
| 09 | [BackboneFast](./09_backbonefast/backbonefast.md) | Inferior BPDU 인식, 간접 장애 시 Max-age 20초 생략 |
| 10 | [RSTP](./10_rstp/rstp.md) | IEEE 802.1w, Rapid-PVST, Proposal/Agreement, Discarding 상태 |
| 11 | [MSTP](./11_mstp/mstp.md) | IEEE 802.1s, CST/PVST/MST 비교, MST Region/Instance, 로드 분산 |

---

## 🧪 실습 (Labs)

| # | 실습 | 핵심 주제 |
| --- | --- | --- |
| Lab 01 | [VTP 4-Switch 동기화](./labs/lab01_vtp-4switch/) | Server / Client / Transparent Mode 동작 검증 |
| Lab 02 | [STP Block Port 제어 (2 SW)](./labs/lab02_stp-blockport) | VLAN별 경로 분리 (Port-Priority / Cost) |
| Lab 03 | [PVST+ Root 분산 (3 SW Triangle)](./labs/lab03_pvst-loadbalance/) | VLAN별 Root / Backup Root 구성 |
| Lab 04 | [DS-Access 2-Tier 설계](./labs/lab04_ds-access-design/) | Distribution Switch 이중화 + Root/Backup Root |
| Lab 05 | [PortFast + BPDU Guard](./labs/lab05_portfast-bpduguard/) | Edge Port 즉시 전환, Err-disable 발생 & 복구 |
| Lab 06 | [RSTP 수렴 검증](./labs/lab06_rstp-convergence/) | Rapid-PVST 전환, Proposal/Agreement, 장애 수렴 시간 비교 |
| Lab 07 | [MSTP Instance 구성](./labs/lab07_mstp-instance/) | MST Region 통일, Instance별 Root/Backup, 로드 분산 |

 각 실습 폴더에 `topology` 이미지 존재  
 각 실습 폴더의 `preconfig/*.txt`를 바로 복사해서 GNS3 콘솔에 붙여넣어 사용  

---

## 🎯 핵심 학습 포인트

- **Layer 별 대표장비**: Hub(L1) / Switch(L2) / Router(L3)
- **스위치 5대 동작**: Flooding → Learning → Forwarding → Filtering → Aging
- **VLAN**: 물리적 분할이 아닌 **논리적 Broadcast Domain 분할**
- **Trunk(802.1Q)**: 하나의 링크로 다수의 VLAN 전송 (4Byte Tag 추가)
- **Router-on-a-Stick**: Sub-interface로 VLAN 간 라우팅 (대역폭 60~65% 권장)
- **VTP**: VLAN Database 동기화 (Revision Number 가장 높은 장비 기준)
- **STP**: Layer 2 Loop 방지 → Bridge-ID(Priority + MAC) 기반 Root Bridge 선출
- **STP Port 상태**: Blocking → Listening(15초) → Learning(15초) → Forwarding (총 30~50초)
- **STP 가속 기능**: PortFast(Edge) / UplinkFast(직접 장애) / BackboneFast(간접 장애)
- **RSTP(802.1w)**: Proposal/Agreement 기반 빠른 수렴, Discarding 상태로 통합
- **MSTP(802.1s)**: 동일 경로 VLAN을 Instance로 묶어 STP 개수 최소화 + 로드 분산

---

## ⚡ STP 계열 한눈에 비교

| 구분 | 표준 | 수렴 속도 | VLAN-STP 관계 | 비고 |
| --- | --- | --- | --- | --- |
| **CST** | 802.1D | 느림 (30~50초) | 전체 1개 | 부하 적으나 비유연 |
| **PVST+** | Cisco | 느림 | VLAN당 1개 | 로드 분산 가능 (최대 128) |
| **RSTP** | 802.1w | 빠름 (수 초) | VLAN당 1개 (Rapid-PVST) | Proposal/Agreement |
| **MSTP** | 802.1s | 빠름 (RSTP 기반) | Instance당 N개 VLAN | 초대형 망, STP 최소화 |

---

## 🛠 사용 도구

| 도구 | 용도 |
| --- | --- |
| **GNS3** | 스위치 토폴로지 시뮬레이션 |
| **Cisco IOU / IOSvL2** | L2 Switch 이미지 |
| **Cisco Packet Tracer** | 보조 실습 환경 |
| **MobaXterm** | 콘솔 접속 |
| **Git / GitHub** | 버전 관리 |

---

## License

[MIT License](./LICENSE)

<div align="center">

**작성자**: [KSNAM97](https://github.com/KSNAM97)
**작성일**: 2026.06  

</div>

