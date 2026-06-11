# 13. Storm-control

> Switch Port로 유입되는 **Unicast / Multicast / Broadcast 트래픽**의 임계치를 설정하여
> 초과 시 트래픽을 폐기하거나 Port를 err-disable로 전환하는 보안 기능.

---

## 1. 개요

- Switch의 특정 Port로 입력되는 Unicast / Multicast / Broadcast 트래픽에 대해 **임계치(threshold)** 를 설정.
- 임계치 이내의 트래픽은 정상 처리하지만, **임계치를 초과**하는 트래픽에 대해서는
  해당 트래픽을 **처리하지 않거나(drop)** Port를 **err-disable** 상태로 전환한다.
- 일반적으로 **Broadcast 트래픽**에만 적용한다.
- Storm-control을 설정하지 않으면 비정상 트래픽이 발생할 때 **CPU 소모율이 100%까지 치솟을 수 있다.**

---

## 2. EX1) Broadcast 임계치 30% (단일 임계치)

> SW1의 **Fa0/1**로 입력되는 Broadcast 트래픽에 의해 CPU 소모율이 **30% 이상** 소모되면 처리하지 않아야 한다.

```cisco
# SW1
interface fastethernet 0/1
 storm-control broadcast level 30
```

**확인**

```text
SW1# show storm-control broadcast
Interface   Filter State    Upper       Lower       Current
----------- --------------- ----------- ----------- -------
Fa0/1       Forwarding      30.00%      30.00%      0.00%   ← 임계치 30%
Fa0/2       inactive        100.00%     100.00%     N/A
...
```

---

## 3. EX2) Upper / Lower 임계치 (Hysteresis)

> CPU 소모율이 **30% 이상**이면 차단하되, **20% 이하**로 떨어지면 다시 처리한다.

```cisco
# SW1
interface fastethernet 0/1
 storm-control broadcast level 30 20
```

**확인**

```text
Interface   Filter State    Upper       Lower       Current
----------- --------------- ----------- ----------- -------
Fa0/1       Forwarding      30.00%      20.00%      0.00%
```

---

## 4. EX3) PPS(초당 패킷 수) 기준 임계치

> Broadcast가 초당 **300 pps 이상**이면 차단, **100 pps 이하**가 되면 다시 처리한다.

```cisco
# SW1
interface fastethernet 0/1
 storm-control broadcast level pps 300 100
```

**확인**

```text
Interface   Filter State    Upper       Lower       Current
----------- --------------- ----------- ----------- -------
Fa0/1       Forwarding      300pps      100pps      0pps
```

---

## 5. EX4) 임계치 초과 시 Err-disable 처리

> CPU 소모율이 30% 이상이면 해당 Port를 **err-disable** 상태로 전환한다.

```cisco
# SW1
interface fastethernet 0/1
 storm-control broadcast level 30
 storm-control action shutdown
```

> 💡 `storm-control action`의 기본 동작은 **drop**, `shutdown` 옵션 추가 시 임계치 초과 즉시 err-disable 전환.

---

## 6. EX5) Pps 기준 + Err-disable

> Broadcast가 초당 300개 이상 입력되면 해당 Switchport를 err-disable 상태로 전환한다.

```cisco
# SW1
interface fastethernet 0/1
 storm-control broadcast level pps 300
 storm-control action shutdown
```

---

## 7. [실습] Storm-control + STP 수렴

#### 🔸 토폴로지

```text
            Fa0/23                          Fa0/23
   ┌─────┐ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ ┌─────┐
   │ SW1 │ ─────────────────────────────── │ SW2 │
   └─────┘ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ └─────┘
            Fa0/24                          Fa0/24 (Blocking)
```

### EX1-1)
