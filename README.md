# 📚 Cisco Layer 2 Switch Study

[![Cisco](https://img.shields.io/badge/Cisco-1BA0D7?style=flat&logo=cisco&logoColor=white)]()
[![Switch](https://img.shields.io/badge/Switch-L2-blue?style=flat)]()
[![GNS3](https://img.shields.io/badge/GNS3-Lab-green?style=flat)]()
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)]()



**Cisco Layer 2 Switch (VLAN / Trunk / VTP / STP / RSTP / MSTP / SPAN / Storm-control / Port-security)**

> - **Part 1 · Layer 2** ← 현재 문서
> - [Part 2 · Layer 3](https://github.com/KSNAM97/Cisco-Switch-L3-Study)

---

Cisco IOS 기반 **L2 Switch** 학습 정리 레포지토리.
Hub / Switch / Router 차이부터 VLAN, Trunk, VTP, STP(PVST+),
RSTP·MSTP와 STP 옵션(PortFast / UplinkFast / BackboneFast), 그리고
**SPAN / Storm-control / Port-security** 까지 포함하는 L2 보안·운영 기능을 다룬다.
**CCNA / CCNP** 수준의 실습 노트.

| 항목 | 내용 |
| --- | --- |
| 대상 | Cisco IOS (IOU / IOSvL2 / 2960·3560 등) |
| 범위 | Switch 기초, VLAN, Trunk(802.1Q), VTP, STP(PVST+), RSTP(802.1w), MSTP(802.1s), SPAN, Storm-control, Port-security |
| 실습 | GNS3 / Cisco Packet Tracer |
| 키워드 | Flooding, Learning, Forwarding, Filtering, Aging, DTP, 802.1Q, Root Bridge, Convergence, Proposal/Agreement, Mirroring, Err-disable, Sticky MAC |

---

```
cisco-switch-l2study/
 01_switch-basics/      # Hub vs Switch vs Router, 5가지 기본 동작
 02_vlan/               # VLAN 개념, 생성/삭제, Range 지정
 03_trunk/              # Switchport Mode, 802.1Q Tagging, Router-on-a-Stick
 04_vtp/                # VLAN Trunking Protocol (Server/Client/Transparent)
 05_stp/                # Spanning-Tree (PVST+), Root Bridge, Port Role
 06_stp-port-states/    # STP 5가지 Port 상태, Convergence 시간
 07_portfast/           # PortFast, BPDU Guard, Err-disable 복구
 08_uplinkfast/         # UplinkFast (액세스 스위치 Block→Forwarding 즉시 전환)
 09_backbonefast/       # BackboneFast (간접 링크 장애 시 Max-age 단축)
 10_rstp/               # RSTP (802.1w, Rapid-PVST), Proposal/Agreement
 11_mstp/               # MSTP (802.1s), MST Region / Instance, 부하 분산
 12_span/               # SPAN / RSPAN (트래픽 미러링)
 13_storm-control/      # Broadcast/Multicast/Unicast Storm 제어 + Err-disable
 14_port-security/      # MAC 기반 Port 접근 제어, Sticky MAC
 labs/                  # 통합 실습 (Preconfig 포함)
    lab01_vtp-4switch/
    lab02_stp-blockport/
    lab03_pvst-loadbalance/
    lab04_ds-access-design/
    lab05_portfast-bpduguard/
    lab06_rstp-convergence/
    lab07_mstp-instance/

 LICENSE
 README.md
```

---

## 📖 챕터별 목차

| # | 챕터 | 핵심 내용 |
| --- | --- | --- |
| 01 | [Switch 기본 동작](./01_switch-basics/switch-basics.md) | Hub / Switch / Router 차이, Collision/Broadcast Domain, 5가지 기본 동작 |
| 02 | [VLAN](./02_vlan/vlan.md) | Virtual LAN 개념, Standard/Extended VLAN, VLAN 생성/삭제, `range` |
| 03 | [Trunk](./03_trunk/trunk.md) | Switchport Mode, IEEE 802.1Q Tagging, Router-on-a-Stick, `allowed vlan` |
| 04 | [VTP](./04_vtp/vtp.md) | VTP Domain/Password/Mode, Revision Number, VTP Message |
| 05 | [STP](./05_stp/stp.md) | PVST+, Root Bridge 선출, Port Role (DP/RP/AP), Priority·Cost |
| 06 | [STP Port 상태](./06_stp-port-states/stp-port-states.md) | Disable/Blocking/Listening/Learning/Forwarding, Convergence(30/50초) |
| 07 | [PortFast](./07_portfast/portfast.md) | Edge Port 즉시 Forwarding, BPDU Guard, Err-disable & 복구 |
| 08 | [UplinkFast](./08_uplinkfast/uplinkfast.md) | Block Port 즉시 전환 (액세스 SW), RP 변경(즉시) |
| 09 | [BackboneFast](./09_backbonefast/backbonefast.md) | Inferior BPDU 수신, Max-age 20초 단축 |
| 10 | [RSTP](./10_rstp/rstp.md) | IEEE 802.1w, Rapid-PVST, Proposal/Agreement, Discarding |
| 11 | [MSTP](./11_mstp/mstp.md) | IEEE 802.1s, CST/PVST/MST 비교, MST Region/Instance, 부하 분산 |
| 12 | [SPAN](./12_span/span.md) | 트래픽 Mirroring, Local SPAN / Remote SPAN, Reflector-port |
| 13 | [Storm-control](./13_storm-control/storm-control.md) | Broadcast/Multicast/Unicast 폭주 제어, %/pps 임계치, Err-disable + 자동 복구 |
| 14 | [Port-security](./14_port-security/port-security.md) | MAC 기반 접근 제어, Violation 모드(protect/restrict/shutdown), Sticky MAC |

---

## 🧪 통합 실습 (Labs)

| # | 실습 | 내용 |
| --- | --- | --- |
| Lab 01 | [VTP 4-Switch](./labs/lab01_vtp-4switch) | Server / Client / Transparent Mode |
| Lab 02 | [STP Block Port (2 SW)](./labs/lab02_stp-blockport) | VLAN별 변경 (Port-Priority / Cost) |
| Lab 03 | [PVST+ Root (3 SW Triangle)](./labs/lab03_pvst-loadbalance) | VLAN별 Root / Backup Root |
| Lab 04 | [DS-Access 2-Tier](./labs/lab04_ds-access-design) | Distribution Switch + Root/Backup Root |
| Lab 05 | [PortFast + BPDU Guard](./labs/lab05_portfast-bpduguard) | Edge Port 보호, Err-disable & 복구 |
| Lab 06 | [RSTP 수렴](./labs/lab06_rstp-convergence) | Rapid-PVST 수렴, Proposal/Agreement |
| Lab 07 | [MSTP Instance](./labs/lab07_mstp-instance) | MST Region 구성, Instance Root/Backup |

각 실습 디렉터리에는 `topology` 와 `preconfig/*.txt` 가 포함되어 GNS3 에서 바로 로드할 수 있다.

---

## 🧠 핵심 요약

- **Layer 구분** : Hub(L1) / Switch(L2) / Router(L3)
- **Switch 5가지 동작** : Flooding · Learning · Forwarding · Filtering · Aging
- **VLAN**: Broadcast Domain 분리
- **Trunk(802.1Q)**: VLAN 다중 전송 (4Byte Tag 삽입)
- **Router-on-a-Stick**: Sub-interface 기반 VLAN 라우팅 (사용률 60~65% 한계)
- **VTP**: VLAN Database 동기화 (Revision Number 주의)
- **STP**: Layer 2 Loop 방지, Bridge-ID(Priority + MAC)로 Root 선출
- **STP Port 상태** : Blocking → Listening(15s) → Learning(15s) → Forwarding (총 30~50초)
- **STP 옵션** : PortFast(Edge) / UplinkFast(직접 장애) / BackboneFast(간접 장애)
- **RSTP(802.1w)**: Proposal/Agreement로 즉시 수렴, Discarding 통합
- **MSTP(802.1s)**: VLAN을 Instance에 매핑하여 STP 수 감소 + 부하 분산
- **SPAN**: 특정 Port 트래픽을 모니터링 장비로 Mirroring (Local / Remote)
- **Storm-control**: Broadcast/Multicast/Unicast 폭주 시 임계치 기반 drop/shutdown
- **Port-security**: MAC 기반 접근 제어, Sticky로 자동 학습/등록

---

## 📊 STP 종류 비교 요약

| 항목 | 표준 | 수렴 속도 | VLAN당 STP | 비고 |
| --- | --- | --- | --- | --- |
| **CST** | 802.1D | 느림(30~50초) | 1개 | 단일 STP |
| **PVST+** | Cisco | 느림 | VLAN당 1개 | 최대 128개 |
| **RSTP** | 802.1w | 빠름(수 초 이내) | VLAN당 1개 (Rapid-PVST) | Proposal/Agreement |
| **MSTP** | 802.1s | 빠름(RSTP 기반) | Instance당 N개 VLAN | 대규모 망, STP 수 절감 |

---

## 🛠️ 사용 도구

| 도구 | 용도 |
| --- | --- |
| **GNS3** | 메인 시뮬레이터 |
| **Cisco IOU / IOSvL2** | L2 Switch 이미지 |
| **Cisco Packet Tracer** | 보조 시뮬레이터 |
| **MobaXterm** | 터미널 접속 |
| **Git / GitHub** | 버전 관리 / 노트 정리 |

---

## License

[MIT License](./LICENSE)
