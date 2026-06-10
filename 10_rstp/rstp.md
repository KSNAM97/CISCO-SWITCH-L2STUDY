# 10. RSTP (IEEE 802.1W, Rapid-PVST)

> PVST의 **Convergence 시간이 길다는 단점**을 보완하여 개발된 Protocol.
> Listening / Learning 같은 긴 대기 시간 없이 빠르게 Forwarding으로 전환.

---

## 1. 특징

- BPDU 교환 후 대기 시간 없이 **Proposal / Agreement** 메시지를 사용하여 빠르게 Forwarding 전환.
- BPDU 교환 후 **Bridge-ID가 작은 Switch**가 해당 Port를 DP로 전환하는 **Proposal**을 송신하고,
  **Bridge-ID가 큰 Switch**는 그 응답인 **Agreement**를 송신한다.

| 메시지 | 의미 |
| --- | --- |
| **Proposal** | 자신이 DP로 동작함을 알리는 Message |
| **Agreement** | Proposal에 대한 응답 |

> 💡 Switch–Hub 이중화 시
> - PVST : **Loop 발생**
> - RSTP : **Backup Port 생성** (Block Port와 동일 개념)
> - PVST만 지원하는 장비 연결 시 자동으로 PVST로 전환

```text
   32768                                          32768
   1111.1111.1111                                 2222.2222.2222
   SW1 ════════════════════════════════════════════ SW2

   ──────────── 1. Proposal ────────────►
   ◄──────────── 2. Agreement ───────────
```

---

## 2. Port 상태 비교 (PVST vs RSTP)

| PVST | RSTP |
| --- | --- |
| Disable | Discarding |
| Blocking | Discarding |
| Listening | Discarding |
| Learning | Learning |
| Forwarding | Forwarding |

> PVST의 Disable / Blocking / Listening 3가지가 RSTP의 **Discarding** 하나로 통합됨.

---

## 3. 설정

```cisco
Switch(config)# spanning-tree mode rapid-pvst
```

> Mode 변경을 통해 PVST를 RSTP로 동작시킬 수 있으며, 그 외 나머지 기능은 PVST와 동일하다.

### EX1) RSTP 구성

> 조건:
> - SW1, SW2, SW3은 RSTP로 동작
> - VLAN 1에 대해 SW2가 Root Bridge, SW3이 Backup Root Bridge

```cisco
# SW1
spanning-tree mode rapid-pvst

# SW2
spanning-tree mode rapid-pvst
spanning-tree vlan 1 priority 4096

# SW3
spanning-tree mode rapid-pvst
spanning-tree vlan 1 priority 16384
```

**확인**: `show spanning-tree`

---

## 4. 장애 발생 / 복구 동작 (debug spanning-tree events)

**Root ~ Non-Root 구간 장애 발생** (SW2 RP shutdown)

```text
SW2(config-if)# shutdown

! SW1
RSTP(1): updt roles, root port Fa0/24 going down
RSTP(1): Fa0/20 is now root port           ← 즉시 새 RP 선출
```

**장애 복구** (Proposal / Agreement 교환으로 즉시 전환)

```text
! SW2
RSTP(1): initializing port Fa0/24
RSTP(1): Fa0/24 is now designated
RSTP(1): transmitting a proposal on Fa0/24
RSTP(1): received an agreement on Fa0/24

! SW1
RSTP(1): updt roles, received superior bpdu on Fa0/24
RSTP(1): Fa0/24 is now root port
RSTP(1): Fa0/20 blocked by re-root
RSTP(1): synced Fa0/24
RSTP(1): Fa0/20 is now alternate
RSTP(1): transmitting an agreement on Fa0/24 as a response to a proposal
```
⬅ 이전: [09. BackboneFast](../09_backbonefast/backbonefast.md) | ➡ 다음: [11. MSTP](../11_mstp/mstp.md)
