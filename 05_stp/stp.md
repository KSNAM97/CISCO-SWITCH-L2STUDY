# 05. STP (Spanning-Tree Protocol, IEEE 802.1D)

> Switch 이중화 환경에서 **Layer 2 Loop를 방지**하는 알고리즘.

---

## 1. STP 등장 배경

- 망의 안정화를 위해서는 장비 간 **이중화 구성**이 효율적
- 이중화 시 한 경로 장애 발생해도 후속 경로로 통신 가능
- **하지만** Switch 간 이중화 이상으로 연결 시 **Loop 발생**

### Loop 발생 시 문제

L2는 TTL이 없어 Loop가 자동으로 멈추지 않음 → 다음 3가지 현상 발생:

1. **Broadcast Storm** (Broadcast + 2개 이상 경로 = 무한 순환)
2. **중복된 Frame 수신**
3. **불안정한 MAC Address Table 생성** (포트가 계속 바뀜)

→ 이를 방지하는 알고리즘이 **STP**.  
→ Root Bridge로 가는 경로 중 **여분 경로를 차단(Block)** 하여 Loop 제거.

---

## 2. PVST+ (Per-VLAN Spanning-Tree)

- Cisco 독자 방식 STP 확장
- **VLAN당 하나의 STP Instance**를 지원
- VLAN마다 다른 Root Bridge, 다른 경로 사용 가능 → 트래픽 로드 분산

---

## 3. BPDU (Bridge Protocol Data Unit)

Switch 간 연결 시 상호 송수신하는 STP 메시지.

| 필드 | 내용 |
| --- | --- |
| **Bridge-ID** | `VLAN Priority` + `MAC Address` |
| **Cost** | 10M = 100, 100M = 19, **1G = 4**, 10G = 2 |
| **Port-Priority** | `128.x` (x = 포트 번호). Fa0/1 = 128.1, Fa0/2 = 128.2 |
| **Hello-Time** | 2초 |
| **Max-Age** | 20초 (이 시간 초과 시 장애로 판단) |
| **Forward-Delay** | 15초 (장애 발생 후 15초 이내 포트 오픈) |

> 모든 SW는 **연결되기 전**에는 자신을 Root라 생각하고 BPDU를 송출.  
> 연결 후 협상 → **Root Bridge만 BPDU 생성 가능**, 나머지는 중계.

---

## 4. STP 동작 순서

### ① Root Bridge 선출 (Bridge-ID 기준)

**선출 우선순위**:
1. **Priority** (`32768 + VLAN 번호`)가 **가장 낮은** Switch
2. Priority 동률이면 **MAC Address가 가장 낮은** Switch

### ② Port Role 선출

| Role | 설명 |
| --- | --- |
| **DP (Designated Port)** | BPDU를 **송신**하는 Port. Root Bridge의 모든 Port는 DP. **세그먼트당 1개**의 DP. |
| **RP (Root Port)** | BPDU를 **수신**하는 Port. Root Bridge로 가는 **최단경로**. **Non-Root Switch당 1개**. |
| **AP (Alternated Port) / Block** | 차단 포트. Loop 방지를 위해 BPDU만 수신하고 데이터는 Block. |

> Root Bridge: 모든 Port = **DP**  
> Non-Root: **RP 1개** + 나머지 DP 또는 AP

### ③ AP(Block Port) 선출 우선순위

1. **Cost 값이 큰 Port**가 Block
2. Cost 동률 → **Sender Bridge-ID가 큰 Port**가 Block
3. 모두 동률 → **Port-Priority가 큰 Port**가 Block (Root Bridge 기준)

---

## 5. STP 토폴로지 예시

### 🔹 Topology 1 (단일 링크)

 
     Fa0/20 [DP]              Fa0/20 [RP]
SW1 ════════════════════════════ SW2 Fa0/21 [DP] Fa0/21 [AP] 32768 32768 1111.1111.1111 2222.2222.2222

 
- SW1: MAC이 더 낮음 → **Root Bridge**
- SW1 모든 포트: DP
- SW2: 한 포트는 RP(최단), 다른 포트는 AP

#### Root Bridge (SW1) 확인

SW1# show spanning-tree

VLAN0001 Spanning tree enabled protocol ieee Root ID Priority 32769 Address 0c4c.9ffd.0000 This bridge is the root ← Root ...

Interface Role Sts Cost Prio.Nbr Type Gi0/0 Desg FWD 4 128.1 P2p ← DP Gi0/1 Desg FWD 4 128.2 P2p ← DP ...

 
#### Non-Root (SW2) 확인

SW2# show spanning-tree

VLAN0001 Root ID Priority 32769 Address 0c4c.9ffd.0000 Cost 4 Port 1 (GigabitEthernet0/0) ← Root Port ...

Interface Role Sts Cost Prio.Nbr Type Gi0/0 Root FWD 4 128.1 P2p ← RP Gi0/1 Altn BLK 4 128.2 P2p ← AP

 
### 🔹 Topology 2 (삼각형 - 3 SW)

     SW1 (Root)
   /          \
 DP            DP
/                \
SW2 ────────────── SW3 AP ← Block 발생

 
- SW2와 SW3은 각각 Root로 향하는 RP를 가지며,
- SW2-SW3 사이 링크에서는 Bridge-ID가 큰 쪽이 **AP(Block)**.

### 🔹 Topology 3 (속도 다른 링크 - Cost 차이)

SW1(Root) ── Gi(1G, Cost 4) ── SW3 │ └── Et(10M, Cost 100) ── SW2 ── SW3 (Et)

 
- SW2는 SW1까지 **Ethernet(Cost 100)** → 비싼 경로
- SW3은 SW1까지 **Gi(Cost 4)** → 빠른 경로
- 누적 Cost가 큰 쪽 Port가 Block

---

## 6. Priority를 이용한 Root Bridge 변경

> **`spanning-tree vlan <ID> priority <값>`**  
> `값`은 **4096의 배수** (0, 4096, 8192, 12288, ...).

### EX1) SW2가 VLAN 10/20/30/40의 Root Bridge가 되도록

SW2(config)# vlan 10,20,30,40 ! SW2(config)# interface range fa 0/23 - 24 SW2(config-if)# switchport trunk encapsulation dot1q SW2(config-if)# switchport mode trunk ! SW2(config)# spanning-tree vlan 10,20,30,40 priority 4096

 
확인: `show spanning-tree vlan 10`에서 `This bridge is the root` 출력.

---

## 7. Cost / Port-Priority를 이용한 Block Port 변경

> **Cost는 BPDU를 수신하는 Non-Root 측 포트만 RPC 계산에 사용**됩니다.  
> Root에서 Cost를 바꿔도 BPDU에 실리지 않아 Non-Root의 Blocking 결정에 영향 X.  
> → **Block Port를 가진 Switch (Root에서 멀리 떨어진 Switch)에서 설정** 또는 **Root에서 Port-Priority 조정**.

### 토폴로지

          <VLAN 10,20,30,40>
gi3/2 gi3/2 SW1 ═══════════════════════════════════ SW2 gi3/3 gi3/3 32768 32768 1111.1111.1111 2222.2222.2222

 
### EX1) Root에서 Port-Priority로 경로 분리

> SW1이 Root.  
> VLAN 10/20 → gi3/2 사용, 장애 시 gi3/3 사용  
> VLAN 30/40 → gi3/3 사용, 장애 시 gi3/2 사용

**Port-Priority가 낮은 쪽이 선호** → 다른 쪽이 자동 Block.

방법 ①: 사용하고자 하는 포트의 Priority를 **낮춘다**

SW1(config)# interface gi3/2 SW1(config-if)# spanning-tree vlan 10,20 port-priority 64 ! SW1(config)# interface gi3/3 SW1(config-if)# spanning-tree vlan 30,40 port-priority 64

 
방법 ②: 사용하지 않을 포트의 Priority를 **높인다**

SW1(config)# interface gi3/2 SW1(config-if)# spanning-tree vlan 30,40 port-priority 192 ! SW1(config)# interface gi3/3 SW1(config-if)# spanning-tree vlan 10,20 port-priority 192

 
> `port-priority`는 **16의 배수** (0, 16, 32, ..., 128(기본), ..., 240).

---

## 8. 핵심 명령어 정리

| 명령어 | 설명 |
| --- | --- |
| `show spanning-tree` | 전체 STP 상태 |
| `show spanning-tree vlan <ID>` | 특정 VLAN STP 상태 |
| `spanning-tree vlan <ID> priority <값>` | Bridge Priority 변경 (4096 배수) |
| `spanning-tree vlan <ID> root primary` | 해당 VLAN의 Primary Root로 자동 설정 |
| `spanning-tree vlan <ID> root secondary` | Secondary Root |
| `spanning-tree vlan <ID> cost <값>` | 포트의 Cost 변경 |
| `spanning-tree vlan <ID> port-priority <값>` | 포트 Port-Priority 변경 (16 배수) |

---

## 9. 핵심 요약

- **STP의 목적**: L2 Loop 방지 (Broadcast Storm / 중복 Frame / 불안정 MAC Table)
- **Root Bridge 선출**: Priority(낮음) → MAC(낮음)
- **Port Role**: Root는 모두 DP, Non-Root는 RP 1개 + 나머지 DP/AP
- **AP 선출**: Cost > Sender Bridge-ID > Port-Priority
- **PVST+**: VLAN당 STP Instance → 로드 분산 가능
- **튜닝**: Root 변경은 `priority`, Block 변경은 `port-priority` 또는 `cost`

---

🏁 모든 학습 완료. [메인으로](../README.md)
