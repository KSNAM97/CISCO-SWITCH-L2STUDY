# Cisco Switch 학습 자료

Cisco 스위치(L2) 기본 동작 원리부터 VLAN, Trunk, VTP, STP까지 실습 중심으로 정리한 자료입니다.

## 📚 목차

| # | 주제 | 설명 |
|---|------|------|
| 01 | [Switch 기본](./01_switch-basics/switch-basics.md) | Hub/Switch/Router 비교, 스위치 동작원리 (Flooding, Learning, Forwarding, Filtering, Aging) |
| 02 | [VLAN](./02_vlan/vlan.md) | Virtual LAN 개념, 생성/삭제, Range 명령어 |
| 03 | [Trunk](./03_trunk/trunk.md) | Switchport Mode, IEEE 802.1Q Tagging, Router-on-a-Stick |
| 04 | [VTP](./04_vtp/vtp.md) | VLAN Trunking Protocol, Server/Client/Transparent Mode |
| 05 | [STP](./05_stp/stp.md) | Spanning-Tree Protocol, Root Bridge 선출, Port 역할 |

## 🛠️ 실습 환경

- **Cisco IOS** (IOU / IOSvL2 / 실장비)
- **에뮬레이터**: EVE-NG, GNS3, Packet Tracer

## 📂 디렉토리 구성

각 주제 폴더에는 다음과 같이 구성되어 있습니다.

- `*.md` : 이론 및 개념 정리
- `preconfig/*.txt` : 실습용 사전 구성(preconfig) 스크립트

## 📌 참고

- L2 Switch는 Cisco IOS에 의존하는 **하드웨어 기반 장비**입니다.
- 본 자료는 CCNA 수준의 학습 내용을 기반으로 작성되었습니다.

---

✏️ 작성: 학습 노트
