# 06. STP 5가지 Port 상태 (IEEE 802.1D)

> 장비 연결 시 무조건 STP가 동작하며, Port는 아래 5가지 상태를 거친다.

---

## 1. Disable

- Switchport가 동작하지 않는 상태 (`shutdown` 상태이거나 No Cable인 상태)

| 항목 | 동작 |
| --- | --- |
| BPDU 송/수신 | ❌ |
| MAC Address 학습 | ❌ |
| 데이터 전송 | ❌ |

---

## 2. Blocking

- Switch가 **Loop 방지**를 위해 Port를 논리적으로 차단한 상태

| 항목 | 동작 |
| --- | --- |
| BPDU 수신 | ✅ |
| MAC Address 학습 | ❌ |
| 데이터 전송 | ❌ |

---

## 3. Listening (Loop 인지 확인 단계)

- Blocking 상태인 Port가 **DP / RP로 전환** 시 Listening 상태로 전환
- 통신 장비 연결 또는 기존 경로 장애로 인해 활성화로 전환되는 **첫 번째 단계**
- **15초간 진행**

| 항목 | 동작 |
| --- | --- |
| BPDU 수신 | ✅ |
| MAC Address 학습 | ❌ |
| 데이터 전송 | ❌ |

---

## 4. Learning

- Listening에서 15초 유지 후 **Loop가 감지되지 않으면** Learning으로 전환
- MAC Address Table에 MAC 등록 가능, 역시 **15초간 진행**

| 항목 | 동작 |
| --- | --- |
| BPDU 수신 | ✅ |
| MAC Address 학습 | ✅ |
| 데이터 전송 | ❌ |

---

## 5. Forwarding

- Listening → Learning을 거쳐 전환되며 **데이터 통신이 가능한 상태**

| 항목 | 동작 |
| --- | --- |
| BPDU 수신 | ✅ |
| MAC Address 학습 | ✅ |
| 데이터 전송 | ✅ |

---

## 6. 전체 수렴 시간 (Convergence)

```text
Block (20초 : Max-age)  ──►  Listening (15초)  ──►  Learning (15초)  ──►  Forwarding
└──────────────────────── 총 50초 소요 ─────────────────────────────┘
```

> - **RP / Non-Root 구간 장애**: Listening → Learning → Forwarding (총 30초)
> - **Root ~ Backup Root 구간 장애**: Block(Max-age 20초) → Listening → Learning → Forwarding (총 50초)

### 타이머 조정 명령어

```cisco
Switch(config)# spanning-tree vlan <x> hello-time   <초>   ! Default 2초
Switch(config)# spanning-tree vlan <x> forward-time <초>   ! Default 15초
Switch(config)# spanning-tree vlan <x> max-age      <초>   ! Default 20초
```

---

## 7. 장애 발생 / 복구 디버그 (PVST)

> `debug spanning-tree events` 로 상태 전환 과정 확인

**Root ~ Non-Root 구간 장애 발생 (SW3에서 RP shutdown)**

```text
SW3(config)# interface gi0/1
SW3(config-if)# shutdown

! SW3
STP: VLAN0001 we are the spanning tree root
STP[1]: Generating TC trap for port GigabitEthernet0/1
%LINK-5-CHANGED: Interface Gi0/1, changed state to administratively down
STP: VLAN0001 Topology Change rcvd on Gi0/0

! SW1 (새 경로 Gi0/2가 listening → learning → forwarding)
STP: VLAN0001 Gi0/2 -> listening
STP: VLAN0001 heard root 32769-0c5d.99af.0000 on Gi0/2
STP: VLAN0001 Gi0/2 -> learning
STP: VLAN0001 Gi0/2 -> forwarding
```

**장애 복구 (no shutdown)**

```text
SW3(config-if)# no shutdown

! SW3
set portid: VLAN0001 Gi0/1: new port id 8002
STP: VLAN0001 Gi0/1 -> listening
STP: VLAN0001 heard root 32769-0c5d.99af.0000 on Gi0/1
STP: VLAN0001 Gi0/1 -> learning
STP: VLAN0001 Gi0/1 -> forwarding

! SW1 (기존 임시 경로 Gi0/2 → blocking 복귀)
STP: VLAN0001 sent Topology Change Notice on Gi0/0
STP: VLAN0001 Gi0/2 -> blocking
```
