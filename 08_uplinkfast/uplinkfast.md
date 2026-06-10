# 08. UplinkFast

> Block Port를 가진 Switch에서, RP 경로 장애 시 Block Port를 **즉시 Forwarding**으로 전환.

---

## 1. 특징

- **Block Port가 생성된 Switch**(Access Layer 등)에서 설정한다.
- 어떠한 VLAN에 대해서도 **Root Bridge 기능을 수행하지 않아야 한다.**
- Root ~ Non-Root 구간 장애 발생 시 **Block Port를 바로 활성화** (RP 장애 시 즉시 전환)

```text
UplinkFast 설정 전 : RP 경로 장애 → Listening(15초) → Learning(15초) → Forwarding (총 30초)
UplinkFast 설정 후 : RP 경로 장애 → Forwarding (즉각 전환)
```

---

## 2. 설정 전 장애 발생 동작

```text
SW2(config)# interface fastethernet 0/24
SW2(config-if)# shutdown

! SW1
STP: VLAN0001 new root port Fa0/20, cost 38
STP: VLAN0001 Fa0/20 -> listening
STP: VLAN0001 Fa0/20 -> learning
STP: VLAN0001 Fa0/20 -> forwarding
```

---

## 3. 설정 후 장애 발생 동작

```cisco
SW1(config)# spanning-tree uplinkfast   ! ← UplinkFast 설정
```

```text
SW2(config)# interface fastethernet 0/24
SW2(config-if)# shutdown

! SW1 (즉시 Forwarding)
STP: VLAN0001 new root port Fa0/20, cost 3038
%SPANTREE_FAST-7-PORT_FWD_UPLINK: VLAN0001 FastEthernet0/20 moved to Forwarding (UplinkFast).
```

> 💡 UplinkFast 설정 시 Cost에 큰 값(+3000)이 더해진다 (`cost 38` → `cost 3038`).
> 이는 해당 Switch가 다른 Switch의 전송 경로로 선택되지 않도록(Root가 되지 않도록) 하기 위함이다.
