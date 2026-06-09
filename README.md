# Cisco Switch L2 Study

![Cisco](https://img.shields.io/badge/Cisco-IOU-blue)
![Switch](https://img.shields.io/badge/Layer2-Switch-green)
![GNS3](https://img.shields.io/badge/Lab-GNS3-orange)
![License](https://img.shields.io/badge/License-MIT-yellow)

**Cisco Layer 2 Switch 학습 저장소 (VLAN / Trunk / VTP / STP)**

---

## 📌 학습 개요

Cisco IOS 기반 **L2 Switch**의 핵심 개념과 실습을 정리한 저장소입니다.  
Hub / Switch / Router의 차이, 스위치의 동작 원리부터 VLAN, Trunk, VTP, STP(PVST+)까지  
**CCNA / CCNP 수준**의 실무 스위칭 지식을 다룹니다.

| 항목 | 내용 |
| --- | --- |
| 대상 장비 | Cisco IOS (IOU / IOSvL2 / 2960·3560 계열) |
| 학습 영역 | Switch 기본 동작, VLAN, Trunk(802.1Q), VTP, STP(PVST+) |
| 실습 환경 | GNS3 / Cisco Packet Tracer |
| 핵심 키워드 | Flooding, Learning, Forwarding, Filtering, Aging, DTP, 802.1Q, Root Bridge |

---

## 🗂 디렉터리 구조

```
cisco-switch-l2study/
├── 01_switch-basics/      # Hub vs Switch vs Router, 스위치 5대 동작
├── 02_vlan/               # VLAN 개념, 생성/삭제, Range 명령어
├── 03_trunk/              # Switchport Mode, 802.1Q Tagging, Router-on-a-Stick
├── 04_vtp/                # VLAN Trunking Protocol (Server/Client/Transparent)
├── 05_stp/                # Spanning-Tree (PVST+), Root Bridge, Port Role
├── labs/                  # 종합 실습 (Preconfig 제공)
│   ├── lab01_vtp-4switch/
│   ├── lab02_stp-blockport/
│   ├── lab03_pvst-loadbalance/
│   └── lab04_ds-access-design/
|   
├── LICENSE
└── README.md
```


## 📚 이론 목차

| # | 주제 | 핵심 내용 |
| --- | --- | --- |
| 01 | [Switch 기본](./01_switch-basics/switch-basics.md) | Hub / Switch / Router 비교, Collision/Broadcast Domain, 스위치 5대 동작 |
| 02 | [VLAN](./02_vlan/vlan.md) | Virtual LAN 개념, Standard/Extended VLAN, VLAN 생성/삭제, `range` |
| 03 | [Trunk](./03_trunk/trunk.md) | Switchport Mode, IEEE 802.1Q Tagging, Router-on-a-Stick, `allowed vlan` |
| 04 | [VTP](./04_vtp/vtp.md) | VTP Domain/Password/Mode, Revision Number, VTP Message |
| 05 | [STP](./05_stp/stp.md) | PVST+, Root Bridge 선출, Port Role (DP/RP/AP), Priority·Cost 조정 |

---

## 🧪 실습 (Labs)


| # | 실습 | 핵심 주제 |
| --- | --- | --- |
| Lab 01 | [VTP 4-Switch 동기화](./labs/lab01_vtp-4switch/) | Server / Client / Transparent Mode 동작 검증 |
| Lab 02 | [STP Block Port 제어 (2 SW)](./labs/lab02_stp-blockport) | VLAN별 경로 분리 (Port-Priority / Cost) |
| Lab 03 | [PVST+ Root 분산 (3 SW Triangle)](./labs/lab03_pvst-loadbalance/) | VLAN별 Root / Backup Root 구성 |
| Lab 04 | [DS-Access 2-Tier 설계](./labs/lab04_ds-access-design/) | Distribution Switch 이중화 + Root/Backup Root |

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

---

## 🛠 사용 도구

| 도구 | 용도 |
| --- | --- |
| **GNS3** | 스위치 토폴로지 시뮬레이션 |
| **Cisco IOU vL2** | L2 Switch 이미지 |
| **Cisco Packet Tracer** | 보조 실습 환경 |
| **MobaXterm** | 콘솔 접속 |
| **Git / GitHub** | 버전 관리 |

---

## License

[MIT License](./LICENSE)

작성자: [KSNAM97](https://github.com/KSNAM97)
