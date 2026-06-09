# 03. Trunk & Switchport Mode

> Switchport Mode (Access / Trunk / Dynamic), IEEE 802.1Q Tagging, Router-on-a-Stick을 학습합니다.

---

## 1. Switchport Mode

### 🔹 Access Mode

- **하나의 Port로 하나의 VLAN**을 사용하여 통신
- PC, Server 같은 **단말 장비** 연결용
- VLAN을 지원하지 않는 장비가 연결되는 Port

### 🔹 Trunk Mode

- **하나의 Port로 다수의 VLAN**을 사용하여 통신
- Switch, IP Phone 같이 VLAN을 지원하는 장비 연결용
- 예: `SW ─ IP Phone ─ PC` (음성 VLAN 111, 일반 VLAN 10 → Trunk Mode)

**Trunk가 필요한 이유**

Switch–Switch 간 Access로 연결하면 VLAN 수만큼 Link를 연결해야 함 → 비효율적.
Trunk를 사용하면 **하나의 링크로 다수의 VLAN** 전송 가능.

### 🔹 Dynamic Mode (DTP: Dynamic Trunking Protocol)

DTP Message를 사용한 Switch 간 자동 Trunk 협상.

| Mode                | DTP 송신 | DTP 수신 |
| ------------------- | :------: | :------: |
| `dynamic desirable` |    ✅    |    ✅    |
| `dynamic auto`      |    ❌    |    ✅    |

**기본값**: Cisco 2960 이후 모델은 **`dynamic auto`**가 기본.
보안상 이유(VLAN Hopping 공격 방지)로 `desirable`에서 `auto`로 변경됨.

> ⚠️ **중요**: 양쪽 다 기본값(`dynamic auto`)이면 Trunk가 **생성되지 않음** → Access로 동작.

| SW1 측              | SW2 측              | 결과              |
| ------------------- | ------------------- | ----------------- |
| `dynamic auto`      | `dynamic auto`      | ❌ Access         |
| `dynamic auto`      | `dynamic desirable` | ✅ Trunk          |
| `dynamic auto`      | `trunk`             | ✅ Trunk          |
| `trunk`             | `trunk`             | ✅ Trunk (권장)   |

---

## 2. Trunk Protocol

| 구분            | 설명                                       |
| --------------- | ------------------------------------------ |
| **IEEE 802.1Q** | IEEE 표준 Trunk Protocol (LAN 표준)        |
| **ISL**         | Cisco Switch 전용 Trunk Protocol (구형)    |

> Cisco 2950 이하는 `dot1q`만 지원.

---

## 3. IEEE 802.1Q Tagging

Ethernet Frame이 802.1Q Trunk Port로 전송될 때 **4 Byte의 Header가 추가**됩니다.

### Access Mode Ethernet Header

```
┌─────────────────┬────────────┬──────┬──────┐
│ Destination MAC │ Source MAC │ Type │ Data │
└─────────────────┴────────────┴──────┴──────┘
```

### 802.1Q Trunk Mode Ethernet Header

```
┌─────────────────┬────────────┬─────────────────┬──────┬──────┐
│ Destination MAC │ Source MAC │ Tagging (4Byte) │ Type │ Data │
└─────────────────┴────────────┴─────────────────┴──────┴──────┘
```

### 4 Byte Tag 구성 (32 bit)

| 필드          | bit | 설명                                                                  |
| ------------- | :-: | --------------------------------------------------------------------- |
| **EtherType** | 16  | `0x8100` (802.1Q임을 표기)                                            |
| **Priority**  |  3  | 우선순위 (8단계 차등 서비스용)                                        |
| **CFI**       |  1  | `0` = Ethernet, `1` = Token-ring                                      |
| **VLAN-ID**   | 12  | VLAN 정보 (0~4095). 표기되지 않은 환경은 **Native VLAN**으로 처리     |

> 💡 중요한 메시지는 VLAN 1로 전달되는 경우가 많아 **VLAN 1은 가능한 허용**을 권장.

---

## 4. Trunk 설정

### 4.1 사전 VLAN 구성 (SW1·SW2·SW3 예시)

**SW1**

```cisco
en
conf t
!
vlan 10
 name CISCO-CCNA
exit
!
interface range fa 0/1-4
 switchport mode access
 switchport access vlan 10
exit
!
vlan 20
 name CISCO-CCNP
exit
!
interface range fa 0/5-8
 switchport mode access
 switchport access vlan 20
!
vlan 30
 name CISCO-CCIE
!
end
```

> SW2 / SW3 도 동일한 방식으로 VLAN 10 / 20 / 30 구성.

### 4.2 Trunk 설정

```cisco
en
conf t
!
interface fa 0/20
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
end
```

> 일부 IOS는 `switchport trunk encapsulation`이 `dot1q`만 지원되므로 생략 가능.

### 4.3 확인

```cisco
SW# show int trunk

Port      Mode    Encapsulation   Status      Native vlan
Gi0/0     on      802.1q          trunking    1

Port      Vlans allowed on trunk
Gi0/0     1-4094

Port      Vlans allowed and active in management domain
Gi0/0     1,10,20
```

---

## 5. `Allowed` Command

- Switch 간 Trunk 구성 시 해당 Switch가 지원하는 **모든 VLAN이 Forwarding**됨
- **특정 VLAN만 허용** 또는 **특정 VLAN 제외**하려면 `allowed` 명령 사용

```cisco
interface gi 0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan all      ! ← Default
```

### EX1) VLAN 1, 10, 20만 허용

```cisco
interface gi 0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 1,10,20
```

### EX2) VLAN 40 추가

```cisco
interface gi 0/0
 switchport trunk allowed vlan add 40
```

### EX3) VLAN 10 제거

```cisco
interface gi 0/0
 switchport trunk allowed vlan remove 10
```

### EX4) VLAN 100 제외한 모든 VLAN 허용

```cisco
interface gi 0/0
 switchport trunk allowed vlan 1-99,101-4094
```

또는

```cisco
interface gi 0/0
 switchport trunk allowed vlan except 100
```

---

## 6. Router-on-a-Stick (Sub-Interface 방식)

> 하나의 물리 인터페이스를 **논리적인 Sub-interface**로 나누어
> 각 VLAN의 Gateway 역할을 수행하게 하는 방식.

### 🔹 대역폭 제약 (60~65% 권장)

Sub-interface는 단일 물리 링크를 공유하므로 **최대 60~65% 사용률**을 권장.

**근거 (Queuing Theory)** — M/M/1 큐 평균 지연:

```
W = 1 / (μ(1 - ρ))
```

| 사용률 (ρ) | 상대적 지연 |
| :--------: | :---------: |
|    50%     |     2배     |
|  **60%**   |  **2.5배**  |
|    70%     |    3.3배    |
|    80%     |     5배     |
|    90%     |    10배     |
|    95%     |    20배     |

→ **70%를 넘는 순간** 지연이 **비선형적으로 폭증**.  
버스트 트래픽 대비 + 프로토콜 오버헤드 + 확장성 / 이중화 여유까지 고려하면
**65%가 실무적 상한**.

### 🔹 Router 설정

```cisco
en
conf t
!
interface fa 0/0
 no ip address
 no shutdown
!
!--- VLAN 10 Sub-interface ---
interface fa 0/0.10
 encapsulation dot1q 10
 ip address 192.168.10.254 255.255.255.0
!
!--- VLAN 20 Sub-interface ---
interface fa 0/0.20
 encapsulation dot1q 20
 ip address 192.168.20.254 255.255.255.0
!
!--- VLAN 30 Sub-interface ---
interface fa 0/0.30
 encapsulation dot1q 30
 ip address 192.168.30.254 255.255.255.0
!
end
```

### 🔹 SW1 (Trunk 연결)

```cisco
en
conf t
!
interface fa 0/24
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
end
```

### 🔹 검증

```cisco
Router# show ip route
C   192.168.10.0/24 is directly connected, FastEthernet0/0.10
C   192.168.20.0/24 is directly connected, FastEthernet0/0.20
C   192.168.30.0/24 is directly connected, FastEthernet0/0.30

Router# show arp
Internet  192.168.10.2   2   0001.4209.9C52   ARPA   FastEthernet0/0.10
Internet  192.168.20.2   2   0002.179A.E75D   ARPA   FastEthernet0/0.20
Internet  192.168.30.1   2   0001.96B9.4B6D   ARPA   FastEthernet0/0.30
```

```cisco
SW1# show mac address-table
Vlan    Mac Address       Type      Ports
----    --------------    -------   ------
10      0001.4209.9c52    DYNAMIC   Fa0/2
10      0001.c9d3.cb01    DYNAMIC   Fa0/24   ← Router Trunk
20      0001.c9d3.cb01    DYNAMIC   Fa0/24
20      0002.179a.e75d    DYNAMIC   Fa0/6
30      0001.c9d3.cb01    DYNAMIC   Fa0/24
```

> - **IP를 다루는 장비** (Router) → **ARP Table**
> - **프레임을 스위칭하는 장비** (Switch) → **MAC Table**
> - **둘 다 하는 장비** (L3 Switch) → 두 테이블 모두 보유

---

## 7. Physical-Interface 방식 (Sub-Interface 미사용)

> 각 VLAN당 라우터의 물리 인터페이스를 1개씩 사용하는 방식 (비효율적이지만 단순)

### Router

```cisco
en
conf t
!
interface fa 0/0
 ip address 192.168.10.254 255.255.255.0
 no shutdown
!
interface fa 0/1
 ip address 192.168.20.254 255.255.255.0
 no shutdown
!
interface fa 1/0
 ip address 192.168.30.254 255.255.255.0
 no shutdown
!
end
```

### SW1 (Access로 라우터 연결)

```cisco
en
conf t
!
interface fa 0/22
 switchport mode access
 switchport access vlan 10
!
interface fa 0/23
 switchport mode access
 switchport access vlan 20
!
interface fa 0/24
 switchport mode access
 switchport access vlan 30
!
end
```

---

## 8. 핵심 명령어 정리

| 명령어                                              | 설명                          |
| --------------------------------------------------- | ----------------------------- |
| `switchport mode access`                            | Access 모드 지정              |
| `switchport access vlan <ID>`                       | Access VLAN 할당              |
| `switchport trunk encapsulation [dot1q\|isl]`       | Trunk Encapsulation 지정      |
| `switchport mode trunk`                             | Trunk 모드 지정               |
| `switchport mode dynamic [auto\|desirable]`         | DTP 모드 지정                 |
| `switchport trunk allowed vlan <list>`              | 허용 VLAN 지정                |
| `switchport trunk allowed vlan add <ID>`            | 허용 VLAN 추가                |
| `switchport trunk allowed vlan remove <ID>`         | 허용 VLAN 제거                |
| `switchport trunk allowed vlan except <ID>`         | 특정 VLAN 제외 모두 허용      |
| `encapsulation dot1q <VLAN>`                        | Sub-interface VLAN 태깅       |
| `show interfaces trunk`                             | Trunk 상태 확인               |
| `show interfaces <int> switchport`                  | 포트 상세 확인                |

---

➡ 다음: [04. VTP](../04_vtp/vtp.md)
