# 02. VLAN (Virtual LAN)

> 물리적 분할이 아닌 **논리적 Broadcast Domain 분할** 기술을 학습합니다.

---

## 1. VLAN 개념

### 🔹 등장 배경

- Switch는 Collision Domain을 분할하지만 **Broadcast Domain은 공유**
- **문제**: Host가 증가할수록 Broadcast Traffic (ARP, DHCP 등)도 함께 증가
- **이유**: 해당 Switch의 모든 Port는 하나의 VLAN에 포함 (Default VLAN: VLAN 1)
- **해결**: VLAN을 사용하여 **논리적으로** 네트워크 분할

### 🔹 장점

- Broadcast Traffic 분할 → 효율적인 LAN 구성
- **위치적·물리적 제한 없이** 연결 가능

> 💡 Default VLAN은 전부 **VLAN 1**로 지정됨.

---

## 2. VLAN 범위

VLAN은 **12 bit**로 구성 (0 ~ 4095)

- `0`, `4095`: 시스템 예약 → 사용 불가
- 사용 가능: **1 ~ 4094**

| 구분              | 범위        | 비고                                                            |
| ----------------- | ----------- | --------------------------------------------------------------- |
| **Standard VLAN** | 1 ~ 1005    | `1`, `1002 ~ 1005`는 예약 VLAN                                  |
| **Extended VLAN** | 1006 ~ 4094 | 장비 / IOS에 따라 지원 여부 다름 (VTP Transparent Mode 필요)    |

> Cisco Switch는 Standard VLAN을 기본 지원,
> Extended VLAN은 장비 시리즈·IOS에 따라 지원 유무가 다름.

### Default VLAN 확인

```cisco
SW# show vlan brief

VLAN  Name                 Status     Ports
----  -------------------  ---------  ------------------------------
1     default              active     Et0/0, Et0/1, Et0/2, Et0/3
                                      Et1/0, Et1/1, Et1/2, Et1/3
                                      ...
1002  fddi-default         act/unsup
1003  token-ring-default   act/unsup
1004  fddinet-default      act/unsup
1005  trnet-default        act/unsup
```

---

## 3. Port Mode (Access / Trunk)

> Switch Port는 용도에 따라 **Access** 또는 **Trunk** 모드로 동작합니다.

### 🔹 Access Port

- **단 하나의 VLAN**에만 소속되는 포트
- 주로 **PC, 프린터, 서버 등 단말(End Device)** 연결에 사용
- 단말로 나가는 Frame은 **태그 없이(Untagged)** 전달
  → PC는 자신이 어떤 VLAN인지 알지 못함 (Switch가 처리)
- 태그 없는 Frame이 들어오면 **해당 포트의 Access VLAN**으로 처리

### 🔹 Trunk Port

- **여러 VLAN**의 Traffic을 하나의 링크로 전달
- 주로 **Switch ↔ Switch, Switch ↔ Router** 연결에 사용
- **802.1Q 태그(Tagged)**로 VLAN을 구분
  (→ 자세한 내용은 [03. Trunk](../03_trunk/trunk.md) 참고)

### 🔹 Access vs Trunk 비교

| 구분        | Access Port                   | Trunk Port                        |
| ----------- | ----------------------------- | --------------------------------- |
| 소속 VLAN   | **1개**                       | **여러 개**                       |
| 태그        | Untagged (태그 없음)          | Tagged (802.1Q)                   |
| 주 용도     | PC, 프린터 등 단말 연결       | Switch ↔ Switch, Switch ↔ Router  |
| 관련 명령   | `switchport access vlan <ID>` | `switchport trunk allowed vlan`   |

> 💡 **암기 팁**: `access` = VLAN 하나 / `trunk` = 여러 VLAN을 `allowed`로 지정

---

## 4. VLAN 생성 / 삭제

### EX1) SW1에 VLAN 10 생성

```cisco
en
conf t
!
vlan 10
!
end
```

**확인**

```cisco
SW# show vlan brief
...
10    VLAN0010    active
```

### EX2) VLAN 10, 20, 30 동시 생성

```cisco
en
conf t
!
vlan 10,20,30
!
end
```

### EX3) VLAN 10, 20, 30 삭제

```cisco
en
conf t
!
no vlan 10,20,30
!
end
```

### EX4) VLAN Name 지정

```cisco
en
conf t
!
vlan 11-15
!
vlan 11
 name CCNA
!
vlan 12
 name CCNP
!
vlan 13
 name LINUX-SERVER
!
vlan 14
 name MS-SERVER
!
vlan 15
 name AWS
!
end
```

**확인**

```cisco
SW# show vlan brief

VLAN  Name           Status   Ports
----  -------------  -------  -----
11    CCNA           active
12    CCNP           active
13    LINUX-SERVER   active
14    MS-SERVER      active
15    AWS            active
```

---

## 5. Access Port 설정

> 생성한 VLAN에 Port를 할당하여 단말을 소속시킵니다.

### 🔹 명령어 구조

```cisco
SW(config)# interface <포트>
SW(config-if)# switchport mode access          ! 포트를 Access 모드로 고정
SW(config-if)# switchport access vlan <ID>     ! 해당 포트를 특정 VLAN에 할당
```

- `switchport mode access`
  → 포트를 **Access 모드로 고정** (DTP 협상 비활성화)
  → 미설정 시 기본값(`dynamic auto`)으로 동작하여 의도치 않게 Trunk가 될 수 있음

- `switchport access vlan <ID>`
  → 포트를 지정한 VLAN에 소속
  → VLAN 미생성 시 일부 IOS에서는 자동 생성

### EX) Fa0/1 포트를 VLAN 10에 할당

```cisco
SW(config)# vlan 10
SW(config-vlan)# exit
!
SW(config)# interface fa0/1
SW(config-if)# switchport mode access
SW(config-if)# switchport access vlan 10
SW(config-if)# end
```

**확인**

```cisco
SW# show interfaces fa0/1 switchport

Name: Fa0/1
Switchport: Enabled
Administrative Mode: static access      ! Access 모드로 고정됨
Operational Mode: static access
Access Mode VLAN: 10 (VLAN0010)         ! 소속 VLAN 확인
```

> 💡 `show interfaces <포트> switchport` 명령으로
> 포트가 Access인지 Trunk인지, 어떤 VLAN에 속하는지 확인할 수 있음.

---

## 6. `Range` Command

> 동일한 설정을 다수의 Interface에 반복 설정 시 사용

### EX) VLAN 10 / 20에 포트 할당

**조건**

- VLAN 10: `Fa0/1 ~ Fa0/5`
- VLAN 20: `Fa0/6 ~ Fa0/10`

```cisco
SW(config)# vlan 10
SW(config)# vlan 20
!
SW(config)# interface range fa0/1 - 5
SW(config-if)# switchport mode access
SW(config-if)# switchport access vlan 10
!
SW(config)# interface range fa0/6 - 10
SW(config-if)# switchport mode access
SW(config-if)# switchport access vlan 20
```

**확인**

```cisco
SW# show vlan brief

VLAN  Name        Status   Ports
----  ----------  -------  --------------------------------------------
1     default     active   Fa0/11, Fa0/12, ... , Fa0/24
10    VLAN0010    active   Fa0/1,  Fa0/2,  Fa0/3,  Fa0/4,  Fa0/5
20    VLAN0020    active   Fa0/6,  Fa0/7,  Fa0/8,  Fa0/9,  Fa0/10
```

---

## 7. 핵심 명령어 정리

| 명령어                                    | 설명                          |
| ----------------------------------------- | ----------------------------- |
| `vlan <ID>`                               | VLAN 생성                     |
| `vlan <ID1>,<ID2>,<ID3>`                  | 여러 VLAN 동시 생성           |
| `vlan <START>-<END>`                      | 연속 VLAN 동시 생성           |
| `name <NAME>`                             | VLAN 이름 지정                |
| `no vlan <ID>`                            | VLAN 삭제                     |
| `interface range <포트범위>`              | 다수 인터페이스 동시 설정     |
| `switchport mode access`                  | Access 모드 지정              |
| `switchport access vlan <ID>`             | Access VLAN 할당              |
| `show vlan brief`                         | VLAN 정보 확인                |
| `show interfaces <포트> switchport`       | 포트 모드 / 소속 VLAN 확인    |

---

➡ 다음: [03. Trunk](../03_trunk/trunk.md)
