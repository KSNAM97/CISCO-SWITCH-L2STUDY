# 12. SPAN (Switched Port Analyzer)

> Switch의 특정 Port로 송/수신되는 트래픽을 **Mirroring**하여 모니터링 장비로 복사 전송하는 기능.

---

## 1. 등장 배경

- 초창기 Mirroring은 **Hub에서만 가능**한 기능이었기 때문에, 네트워크 장비에 Hub를 연결하여
  특정 Port로 송/수신되는 트래픽을 모니터링 장비로 전송해 분석/수집했다.
- Switch는 **MAC Address 기반**으로 데이터를 전송하기 때문에,
  일반적인 방법으로는 특정 Port의 트래픽을 가로채서 수집할 수 없다.
- 이를 해결하기 위해 Cisco가 개발한 Protocol이 **SPAN**이며,
  관리자가 지정한 Port의 송/수신 트래픽 전체를 **모니터링 장비가 연결된 Port로 Copy** 전송한다.

### 🔹 SPAN 종류

| 종류 | 설명 |
| --- | --- |
| **Local SPAN** | 모니터링 장비와 감시 대상 장비가 **동일 Switch**에 연결된 환경에서 사용 |
| **Remote SPAN (RSPAN)** | 모니터링 장비와 감시 대상 장비가 **서로 다른 Switch**에 연결된 환경에서 사용 |

---

## 2. Local SPAN

### EX1) SW1의 Fa0/1 ↔ Fa0/20 양방향 Mirroring

> 조건:
> - SW1은 **Fa0/1**로 송/수신하는 양방향 데이터를 모니터링 장비가 연결된 **Fa0/20**으로 Copy 전송해야 한다.

#### 🔸 토폴로지

```text
                      ┌───────────────────────┐
                      │        Switch         │
                      └───────────────────────┘
                          │               │
                        Fa0/1           Fa0/20
                          │               │
                         PC          Monitoring (Wireshark)
                  192.168.1.100
```

#### 🔸 설정

```cisco
# SW1
monitor session 1 source interface fastethernet 0/1 both
monitor session 1 destination interface fastethernet 0/20
```

**확인**

```text
SW1# show monitor session 1
Session 1
----------
Type              : Local Session
Source Ports      :
    Both          : Fa0/1
Destination Ports : Fa0/20
    Encapsulation : Native
          Ingress : Disabled
```

**테스트**

```text
PC> ping 192.168.1.101
PC> telnet 192.168.1.101
```

> Monitoring 장비에서 **Wireshark**를 실행하면 Fa0/1로 송/수신되는 데이터가 캡처되어야 한다.

---

## 3. Remote SPAN (RSPAN)

### EX2) SW1(Fa0/1) → SW2(Fa0/20) 원격 Mirroring

> 조건:
> - 감시 대상 PC는 **SW1 Fa0/1**, 모니터링 장비는 **SW2 Fa0/20**에 연결되어 있다.
> - SW1 ↔ SW2는 **Fa0/24**로 Trunk 연결되어 있다고 가정한다.

#### 🔸 토폴로지

```text
   ┌────────────┐  Fa0/24            Fa0/24  ┌────────────┐
   │    SW1     │───────────────────────────│    SW2     │
   └────────────┘                            └────────────┘
        │                                          │
      Fa0/1                                      Fa0/20
        │                                          │
        PC                                     Monitoring
```

#### 🔸 SW1 설정 (Source Switch)

```cisco
# SW1
vlan 999
 remote-span
!
monitor session 1 source interface fastethernet 0/1 both
monitor session 1 destination remote vlan 999 reflector-port fastethernet 0/22
```

#### 🔹 Reflector-port

- Cisco Switch는 각 Port에 **CPU를 별도로 할당**하며, 해당 Port의 송/수신 데이터는 해당 Port의 CPU가 처리한다.
- Reflector-port는 Remote-SPAN 구성 시 Source Interface로 송/수신되는 데이터를 **Copy하여 전송하는 과정**을
  해당 Port의 CPU가 아닌 **Reflector-port로 지정한 Interface의 CPU**가 처리하도록 하는 기능이다.
- ⚠️ Reflector-port는 **해당 Switch에서 사용하지 않는 Port**로 지정해야 한다.

#### 🔸 SW2 설정 (Destination Switch)

```cisco
# SW2
vlan 999
 remote-span
!
monitor session 1 source remote vlan 999
monitor session 1 destination interface fastethernet 0/20
```

#### 🔸 확인

```text
SW1# show monitor session 1
Session 1
----------
Type              : Remote Source Session
Source Ports      :
    Both          : Fa0/1
Dest RSPAN VLAN   : 999

SW2# show monitor session 1
Session 1
----------
Type                  : Remote Destination Session
Source RSPAN VLAN     : 999
Destination Ports     : Fa0/20
    Encapsulation     : Native
          Ingress     : Disabled
```

**테스트**

```text
PC> ping 192.168.1.101
PC> telnet 192.168.1.101
PC> ping 192.168.1.102
PC> telnet 192.168.1.102
```

> SW2에 연결된 Monitoring 장비의 Wireshark에서 SW1의 Fa0/1 송/수신 트래픽이 확인되어야 한다.

---

⬅ 이전: [11. MSTP](../11_mstp/mstp.md) | ➡ 다음: [13. Storm-control](../13_storm-control/storm-control.md)
