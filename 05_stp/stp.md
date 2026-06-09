# IEEE 802.1D Spanning-Tree Protocol (STP)

## 개요

- 망 안정화를 위해 장비 간 **이중화**가 효율적
- 한 경로 장애 시 후속 경로 사용 가능
- 하지만 Switch 간 이중화 연결 시 **Loop 발생**:
  - **Broadcast Storm**
  - **중복 Frame 수신**
  - **불안정한 MAC Address Table**
- 이를 방지하는 알고리즘이 **STP**
- Root-Bridge로 가는 경로를 차단 (Block Port)

## PVST+ (Per-VLAN Spanning-Tree)

- 하나의 VLAN당 하나의 STP를 지원하는 Cisco 기능

---

## STP 동작 순서

### 1. Root-Bridge 선출 (기준: Bridge-ID)

- 모든 Switch는 연결 직후 BPDU 송출 → Root-Bridge 선출
- 선출 후에는 **Root-Bridge만 BPDU 생성 가능**

선출 기준 (Bridge-ID):
1. **Priority** (32768 + VLAN 번호) 낮은 Switch
2. **MAC Address** 낮은 Switch

### 2. Port 선출

| Port 종류 | 설명 |
|----------|------|
| **DP** (Designated Port) | BPDU 송신 포트. Root-Bridge의 모든 포트는 DP. 각 세그먼트당 1개 |
| **RP** (Root Port) | BPDU 수신 포트. Root-Bridge로 가는 **최단 경로**. Non-Root당 1개 |
| **AP** (Alternated Port) | Block Port |

### 3. AP (Alternated/Block Port) 선출 기준

1. **Cost값이 큰** Port
2. **Sender Bridge-ID가 큰** Port
3. **Port-Priority가 큰** Port (Root-Bridge의 Port-Priority 기준)

---

## BPDU 주요 필드

| 필드 | 값/설명 |
|------|---------|
| Bridge-ID | VLAN Priority + MAC Address |
| Cost | 10M=100, 100M=19, 1G=4, 10G=2 |
| Port-Priority | `128.x` (x = port 번호) |
| Hello-Time | 2초 |
| Max-Age | 20초 (초과 시 장애 판단) |
| Forward-Delay | 15초 (장애 발생 시 포트 오픈 시간) |

---

## Root-Bridge 변경 (Priority 사용)

```
SWX(config)# spanning-tree vlan <X> priority <4096의 배수>
SWX(config)# spanning-tree vlan <X> root primary
SWX(config)# spanning-tree vlan <X> root secondary
```

> `priority`는 **4096의 배수**만 입력 가능 (실제 값 = priority + VLAN ID)

## Block Port 변경 (Cost / Port-Priority)

- **Cost 변경**: Block Port를 가진 Switch (Root에서 더 먼 Switch)에서 설정
  ```
  interface <포트>
   spanning-tree vlan <ID> cost <값>
  ```
- **Port-Priority 변경**: Root-Bridge에서 설정
  ```
  interface <포트>
   spanning-tree vlan <ID> port-priority <값>
  ```

---

## 정보 확인

```
show spanning-tree
show spanning-tree vlan <X>
```

> 실습 스크립트는 [`preconfig/`](./preconfig/) 폴더 참조.
