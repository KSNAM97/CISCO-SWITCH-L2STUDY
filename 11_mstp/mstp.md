# 11. MSTP (IEEE 802.1S, Multiple Spanning-Tree) – 초대형 망

> CST와 PVST의 **장점을 결합**한 STP. 동일 경로를 갖는 여러 VLAN을 하나의 STP로 묶어 관리.

---

## 1. CST / PVST / MST 비교

| 방식 | 설명 | 장점 | 단점 |
| --- | --- | --- | --- |
| **CST** | VLAN 수와 무관하게 **STP 1개** 사용 | 장비 부하 적음 | 유연한 Topology 불가 |
| **PVST** | **VLAN당 STP 1개** 사용 | VLAN별 유연한 구성 | CPU 소모 증가 (최대 128개 STP) |
| **MST** | 동일 경로의 복수 VLAN을 **하나의 STP**로 관리 (STP Group) | 효율적 구성 | 설정 복잡 |

> - PVST는 최대 **128개 STP** 지원 → 128개 이상 VLAN 사용 시 MST로 전환 필요.
> - STP Mode를 MST로 변경하면 기본 Spanning-tree는 **RSTP로 동작**한다.

---

## 2. MST Cost (Long 기준)

| 속도 | Cost |
| --- | --- |
| 10 M | 2,000,000 |
| 100 M | 200,000 |
| 1 G | 20,000 |
| 10 G | 2,000 |
| 100 G | 200 |
| 1 T | 20 |
| 10 T | 2 |

---

## 3. EX1) VTP + VLAN 사전 구성

> 조건:
> - SW1: VLAN 2-10 / SW2: VLAN 11-20 / SW3: VLAN 21-30 생성 (전 Switch에서 확인)
> - VTP Domain = `GIT_NET`, Password = `ciscoGIT`

```cisco
# SW1
interface range fa0/20 , fa0/24
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
vtp domain GIT_NET
vtp password ciscoGIT
!
vlan 2-10

# SW2
interface range fa0/22 , fa0/24
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
vtp domain GIT_NET
vtp password ciscoGIT
!
vlan 11-20

# SW3
interface range fa0/20 , fa0/22
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
vtp domain GIT_NET
vtp password ciscoGIT
!
vlan 21-30
```

**확인**: `show interface trunk`, `show vtp status`, `show vtp password`, `show vlan brief`
→ SW1 ~ SW3 모두 VLAN 1 ~ 30 확인

---

## 4. EX2) MST Mode 변경

```cisco
# SW1 / SW2 / SW3 공통
spanning-tree mode mst
```

### 🔹 MST0 (기본 Instance)

- MST로 변경 후 최초에는 **VLAN 1 ~ 4094**의 Loop를 방지한다.
- 관리자가 별도 MST Instance를 만들고 VLAN을 매핑하면,
  **지정하지 않은 나머지 VLAN을 관리**하는 STP가 된다.

```text
SW1# show spanning-tree mst 0
##### MST0    vlans mapped: 1-4094
Bridge   address 0011.1111.1111  priority 32768 (32768 sysid 0)
Root     this switch for the CIST          ← MAC이 가장 낮은 SW1이 Root

Interface  Role Sts Cost   Prio.Nbr Type
Fa0/20     Desg FWD 200000 128.22   P2p
Fa0/24     Desg FWD 200000 128.26   P2p

SW3# show spanning-tree mst 0
Fa0/20    Root FWD 200000 128.22   P2p
Fa0/22    Altn BLK 200000 128.24   P2p   ← Block
```

---

## 5. EX3) MST Instance 구성 (STP 4개로 로드 분산)

> 조건:
> - VLAN 2-10  → SW1 Root, SW2 Backup
> - VLAN 11-20 → SW2 Root, SW3 Backup
> - VLAN 21-30 → SW3 Root, SW1 Backup
> - STP는 4개만 사용 (MST0 + Instance 1/2/3)

### ① MST Instance 매핑 (전 Switch 동일하게)

```cisco
# SW1 / SW2 / SW3 공통
spanning-tree mst configuration
 name MST
 instance 1 vlan 2-10
 instance 2 vlan 11-20
 instance 3 vlan 21-30
```

> ⚠️ `name`, `revision`, `instance ~ vlan` 매핑은 **모든 Switch가 동일**해야 같은 MST Region으로 동작한다.

**Pending 확인** (`show pending`)

```text
Name      [MST]
Revision  1     Instances configured 3

Instance   Vlans mapped
--------   --------------------------
0          1,31-4094
1          2-10
2          11-20
3          21-30
```

### ② Instance별 Priority 설정 (Root / Backup 지정)

```cisco
# SW1
spanning-tree mst 1 priority 0       ! VLAN 2-10  Root
spanning-tree mst 3 priority 4096    ! VLAN 21-30 Backup

# SW2
spanning-tree mst 1 priority 4096    ! VLAN 2-10  Backup
spanning-tree mst 2 priority 0       ! VLAN 11-20 Root

# SW3
spanning-tree mst 2 priority 4096    ! VLAN 11-20 Backup
spanning-tree mst 3 priority 0       ! VLAN 21-30 Root
```

**확인** (`show spanning-tree mst 1` 예시)

```text
SW1# show spanning-tree mst 1
##### MST1   vlans mapped: 2-10
Bridge  address 0011.1111.1111  priority 1 (0 sysid 1)
Root    this switch for the CIST           ← SW1이 Root

SW2# show spanning-tree mst 1
Bridge  address 0022.2222.2222  priority 4097 (4096 sysid 1)
Root    address 0011.1111.1111  priority 1 (0 sysid 1)
Fa0/24  Root FWD 200000 128.26   P2p

SW3# show spanning-tree mst 1
Bridge  address 0033.3333.3333  priority 32769 (32768 sysid 1)
Fa0/20  Root FWD 200000 128.22   P2p
Fa0/22  Altn BLK 200000 128.24   P2p       ← Block
```

> 💡 Instance Priority도 **4096의 배수**를 사용하며, 출력 Priority는 `설정값 + sysid(Instance 번호)`로 표시된다.
> (예: `4096 + 1 = 4097`)

🏁 모든 학습 완료. [메인으로](../README.md)
