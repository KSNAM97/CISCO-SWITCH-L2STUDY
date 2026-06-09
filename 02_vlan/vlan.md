# 02. VLAN (Virtual LAN)

> 물리적 분할이 아닌 **논리적 Broadcast Domain 분할** 기술을 학습합니다.

---

## 1. VLAN 개념

### 🔹 등장 배경

- Switch는 Collision Domain을 분할하지만 **Broadcast Domain은 공유**
- **문제**: Host가 증가할수록 Broadcast Traffic(ARP, DHCP 등)도 함께 증가
- **이유**: 해당 Switch의 모든 Port는 하나의 VLAN에 포함 (Default VLAN: VLAN 1)
- **해결**: VLAN을 사용하여 **논리적으로** 네트워크 분할
- **장점**:
  - Broadcast Traffic 분할 → 효율적인 LAN 구성
  - **위치적·물리적 제한 없이** 연결 가능

> Default VLAN은 전부 **VLAN 1**로 지정됨.

---

## 2. VLAN 범위

VLAN은 **12bit**로 구성 (0 ~ 4095)

- `0`, `4095`: 시스템 예약 → 사용 불가
- 사용 가능: **1 ~ 4094**

| 구분 | 범위 | 비고 |
| --- | --- | --- |
| **Standard VLAN** | 1 ~ 1005 | `1`, `1002~1005`는 예약 VLAN |
| **Extended VLAN** | 1006 ~ 4094 | 장비/IOS에 따라 지원 여부 다름 (VTP Transparent Mode 필요) |

> Cisco Switch는 Standard VLAN을 기본 지원,  
> Extended VLAN은 장비 시리즈·IOS에 따라 지원 유무가 다름.

### Default VLAN 확인

Copy
SW# show vlan brief

VLAN Name Status Ports

1 default active Et0/0, Et0/1, Et0/2, Et0/3 Et1/0, Et1/1, Et1/2, Et1/3 ... 1002 fddi-default act/unsup 1003 token-ring-default act/unsup 1004 fddinet-default act/unsup 1005 trnet-default act/unsup

Copy
---

## 3. VLAN 생성 / 삭제

### EX1) SW1에 VLAN 10 생성

en conf t ! vlan 10 ! end

Copy
확인:
SW# show vlan brief ... 10 VLAN0010 active

Copy
### EX2) VLAN 10, 20, 30 동시 생성

en conf t ! vlan 10,20,30 ! end

Copy
### EX3) VLAN 10, 20, 30 삭제

en conf t ! no vlan 10,20,30 ! end

Copy
### EX4) VLAN Name 지정

en conf t ! vlan 11-15 ! vlan 11 name CCNA ! vlan 12 name CCNP ! vlan 13 name LINUX-SERVER ! vlan 14 name MS-SERVER ! vlan 15 name AWS ! end

Copy
확인:
SW# show vlan brief VLAN Name Status Ports

11 CCNA active 12 CCNP active 13 LINUX-SERVER active 14 MS-SERVER active 15 AWS active

Copy
---

## 4. `Range` Command

> 동일한 설정을 다수의 Interface에 반복 설정 시 사용

### EX) VLAN 10/20에 포트 할당

조건:
- VLAN 10: Fa0/1 ~ Fa0/5
- VLAN 20: Fa0/6 ~ Fa0/10

SW(config)# vlan 10 SW(config)# vlan 20 ! SW(config)# interface range fa0/1 - 5 SW(config-if)# switchport mode access SW(config-if)# switchport access vlan 10 ! SW(config)# interface range fa0/6 - 10 SW(config-if)# switchport mode access SW(config-if)# switchport access vlan 20

Copy
확인:
SW# show vlan brief VLAN Name Status Ports

1 default active Fa0/11, Fa0/12, ... , Fa0/24 10 VLAN0010 active Fa0/1, Fa0/2, Fa0/3, Fa0/4, Fa0/5 20 VLAN0020 active Fa0/6, Fa0/7, Fa0/8, Fa0/9, Fa0/10

Copy
---

## 5. 핵심 명령어 정리

| 명령어 | 설명 |
| --- | --- |
| `vlan <ID>` | VLAN 생성 |
| `vlan <ID1>,<ID2>,<ID3>` | 여러 VLAN 동시 생성 |
| `vlan <START>-<END>` | 연속 VLAN 동시 생성 |
| `name <NAME>` | VLAN 이름 지정 |
| `no vlan <ID>` | VLAN 삭제 |
| `interface range <포트범위>` | 다수 인터페이스 동시 설정 |
| `switchport mode access` | Access 모드 지정 |
| `switchport access vlan <ID>` | Access VLAN 할당 |
| `show vlan brief` | VLAN 정보 확인 |

---

➡ 다음: [03. Trunk](../03_trunk/trunk.md)
