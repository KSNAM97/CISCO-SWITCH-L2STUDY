# 01. Switch 기본 동작

> Layer 1 / 2 / 3 대표 장비 비교와 스위치의 5대 동작 원리
> (Flooding · Learning · Forwarding · Filtering · Aging)를 정리합니다.

---

## 1. Layer 별 대표 장비

### 🔹 Hub (Layer 1)

- Layer 1의 대표 장비
- 자신이 포함된 **동일 네트워크**에 존재하는 장비와 통신 시 사용
- **전기적 신호**를 사용하여 데이터를 Forwarding (신호 증폭 장비)
- **Collision Domain을 공유**하는 장비

### 🔹 Switch (Layer 2)

- Layer 2의 대표 장비
- 자신이 포함된 **동일 네트워크**에 존재하는 장비와 통신 시 사용
- **Ethernet Header의 MAC Address**를 사용 (16진수, 48 bit : `HH-HH-HH-HH-HH-HH`)
- 데이터 수신 시 Layer 2 Header(Ethernet Header)의 **목적지 MAC**을 근거로
  **MAC Address Table**을 참조하여 Forwarding
- **Collision Domain은 분할**하며 **Broadcast Domain은 공유**
  (VLAN 사용 시 Broadcast Domain도 분할 가능)
- Cisco IOS에 의존하는 **하드웨어 기반** 장비

### 🔹 Router (Layer 3)

- Layer 3의 대표 장비
- 자신이 포함되지 **않은 다른 네트워크**로 통신 시 사용
- **IP Header의 IP Address**를 사용 (10진수, 32 bit)
- 데이터 수신 시 Layer 3 Header(IP Header)의 **목적지 IP**를 근거로
  **Routing Table**을 참조하여 Forwarding
- **Broadcast Domain을 분할**하는 장비
- Cisco IOS에 의존하는 **소프트웨어 기반** 장비

---

## 2. 비교 정리

| 구분                  | Hub                       | Switch                    | Router                    |
| --------------------- | ------------------------- | ------------------------- | ------------------------- |
| **Layer**             | 1 (물리)                  | 2 (데이터링크)            | 3 (네트워크)              |
| **식별자**            | 없음 (전기 신호)          | MAC Address               | IP Address                |
| **참조 테이블**       | 없음                      | MAC Address Table         | Routing Table             |
| **Collision Domain**  | **공유**                  | **분할**                  | **분할**                  |
| **Broadcast Domain**  | **공유**                  | 공유 (VLAN 시 분할)       | **분할**                  |
| **처리 방식**         | 하드웨어 (단순)           | 하드웨어 기반 (ASIC)      | 소프트웨어 기반           |
| **장애 영향**         | 연결된 모든 장비 통신 불가| 연결된 장비만 영향        | 외부 네트워크 통신 불가   |

---

## 3. 스위치의 5대 동작 원리

### ① Flooding

- 스위치가 **목적지 MAC Address를 모를 때** 수행하는 동작
- 프레임을 받으면 Destination MAC을 MAC Address Table에서 검색
- 테이블에 없으면 **프레임이 들어온 포트를 제외한 모든 포트**로 전송

**대상 프레임**

- Unknown Unicast Frame
- Unknown Multicast Frame
- Broadcast Frame

### ② Learning

- 프레임 수신 시 **출발지(Source) MAC Address를 학습**하는 과정
- 프레임의 Source MAC과 들어온 포트를 MAC Address Table에 기록

### ③ Forwarding

- 목적지 MAC이 MAC Table에 **있고**, 들어온 포트와 **다른 포트**에 있을 때
- 해당 포트로만 프레임 전달 (다른 포트로는 안 보냄)

**예시**: PC1(Fa0/1) → PC4(Fa0/4)

```text
MAC Table: 0044.4444.4444 = Fa0/4
들어온 포트(Fa0/1) ≠ 목적지 포트(Fa0/4)
 → Fa0/4로만 Forwarding
```

### ④ Filtering

- 목적지 MAC이 MAC Table에 **있고**, **들어온 포트와 같은 포트**에 있을 때
- 프레임을 **버림 (drop)** → Loop 방지
- `SW ─ SW`: Loop 알고리즘(STP) 존재로 안전
- `SW ─ Hub`: Loop 가능성 존재

**예시**: PC1, PC2가 같은 허브에 연결 → 허브가 SW Fa0/1에 연결

```text
PC1 → PC2 프레임 전송 (둘 다 Fa0/1로 들어옴)
SA도 Fa0/1, DA도 Fa0/1
 → 이미 그 세그먼트 안에서 PC2가 받았을 것이므로 Filtering (폐기)
```

### ⑤ Aging

- MAC Address Table에 학습된 정보가 **일정 시간 사용되지 않으면 자동 삭제**되는 기능
- **Cisco 기본 Aging Time: 300초 (5분)**
- 5분간 해당 MAC에서 프레임이 들어오지 않으면 삭제
- 삭제 후 재통신 시 → **Learning 단계로 회귀**

---

## 4. Collision Domain / Broadcast Domain 관점

| 장비       | Collision Domain     | Broadcast Domain               |
| ---------- | -------------------- | ------------------------------ |
| **Hub**    | 모든 포트 공유 (1개) | 모든 포트 공유 (1개)           |
| **Switch** | **포트마다 분할**    | 공유 (1개) — VLAN 사용 시 분할 |
| **Router** | 인터페이스마다 분할  | **인터페이스마다 분할**        |

---

## 5. 장애 발생 시 영향 범위

- **Hub 장애**: Hub에 연결된 모든 장비가 통신 불가
- **Switch 장애**: 해당 스위치에 연결된 장비만 영향, 다른 세그먼트는 영향 없음
- **Router 장애**: 내부 네트워크 통신은 가능, **외부 네트워크 통신 불가**

---

## 6. ARP + ICMP 동작 흐름 (예시)

> `192.168.1.1` (PC1) → `192.168.1.4` (PC4) 로 ping을 보내는 경우

### ① ARP Request (Broadcast)

```text
┌─ Ethernet Header ──────────────┐  ┌─ ARP Request ─────────────────────┐
│ SA: 0011.1111.1111             │  │ Sender IP  : 192.168.1.1          │
│ DA: FFFF.FFFF.FFFF (Broadcast) │  │ Sender MAC : 0011.1111.1111       │
│                                │  │ Target IP  : 192.168.1.4          │
│                                │  │ Target MAC : 0000.0000.0000       │
└────────────────────────────────┘  └───────────────────────────────────┘
```

### ② ARP Reply (Unicast)

```text
┌─ Ethernet Header ──────┐  ┌─ ARP Reply ──────────────────┐
│ SA: 0044.4444.4444     │  │ Sender IP  : 192.168.1.4     │
│ DA: 0011.1111.1111     │  │ Sender MAC : 0044.4444.4444  │
│                        │  │ Target IP  : 192.168.1.1     │
│                        │  │ Target MAC : 0011.1111.1111  │
└────────────────────────┘  └──────────────────────────────┘
```

### ③ ICMP Echo Request

```text
┌─ Ethernet ─────────────┐  ┌─ IP ────────────────┐  ┌─ ICMP ─────────────┐
│ SA: 0011.1111.1111     │  │ SA: 192.168.1.1     │  │ Request (type-8)   │
│ DA: 0044.4444.4444     │  │ DA: 192.168.1.4     │  │                    │
└────────────────────────┘  └─────────────────────┘  └────────────────────┘
```

### ④ ICMP Echo Reply

```text
┌─ Ethernet ─────────────┐  ┌─ IP ────────────────┐  ┌─ ICMP ─────────────┐
│ SA: 0044.4444.4444     │  │ SA: 192.168.1.4     │  │ Reply (type-0)     │
│ DA: 0011.1111.1111     │  │ DA: 192.168.1.1     │  │                    │
└────────────────────────┘  └─────────────────────┘  └────────────────────┘
```

---

## 7. 핵심 요약

- 스위치는 **MAC Address 기반**으로 동작하는 L2 장비
- **5대 동작**
  - Flooding (모름) → Learning (학습) → Forwarding (앎 / 다른 포트)
    → Filtering (앎 / 같은 포트) → Aging (5분 후 삭제)
- **Flooding 대상**: Unknown Unicast / Unknown Multicast / Broadcast
- 스위치는 **Collision Domain 분할**, **Broadcast Domain 공유**
  (VLAN 사용 시 분할 가능 → 다음 챕터)

---

➡ 다음: [02. VLAN](../02_vlan/vlan.md)
