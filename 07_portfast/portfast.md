# 07. PortFast

> 가장 많이 사용하는 STP 옵션. PC / Server 연결 포트를 즉시 Forwarding으로 전환.

---

## 1. 등장 배경

- Switch는 연결되는 장비가 Router / Switch / PC / Server인지 **알 수 없기 때문에**
  모든 Port에 STP를 사용하여 Loop를 체크한다.
- PC / Server는 **Loop를 발생시키지 않으므로** 바로 통신이 가능해야 한다.
- Loop를 발생시키지 않는 장비가 연결되는 Port를 **즉시 Forwarding** 하려면 PortFast 사용.

```text
PortFast 설정 전 : 장비연결 → Listening(15초) → Learning(15초) → Forwarding (총 30초)
PortFast 설정 후 : 장비연결 → Forwarding (즉각 전환)
```

> PortFast를 Enable한 Port는 STP를 동작시키지 않는다.

---

## 2. PortFast 사용 시 Loop 방지 대책 (BPDU Guard)

- PortFast는 특정 Port의 STP를 해제하는 기능이므로,
  해당 Port로 **Switch가 연결되면 Loop가 발생**할 수 있다.
- PortFast 설정 전 **BPDU Guard**를 먼저 설정하면 Switch가 연결되어도 Loop를 방지할 수 있다.

### 🔹 BPDU Guard
- BPDU Guard를 설정한 Switchport로 **BPDU가 입력되면** 해당 Port를 **err-disable** 상태로 전환.

### 🔹 Err-disable
- **Disable** : 관리자가 `shutdown` 명령어로 직접 차단한 상태
- **Err-disable** : 관리자가 설정한 기능(BPDU Guard, Storm-control, Port-security 등)에 의해
  Switch가 **스스로** 자신의 Port를 논리적 shutdown 상태로 전환한 것
- 복구: 물리적으로 `shutdown` → `no shutdown` 해야 err-disable이 해제됨

---

## 3. 설정

```cisco
# SW3
interface range gi0/0 - 2
 switchport mode access
 spanning-tree bpduguard enable
 spanning-tree portfast
```

**STP 상태 확인** (`show spanning-tree`)

```text
Interface   Role Sts Cost   Prio.Nbr Type
----------- ---- --- ------ -------- -----------------
Gi0/0       Desg FWD 4      128.1    P2p Edge   ← PortFast = Edge 표기
Gi0/1       Desg FWD 4      128.2    P2p Edge
Gi0/2       Desg FWD 4      128.3    P2p Edge
Gi0/3       Desg FWD 4      128.4    P2p
```

**err-disabled 확인**

```cisco
SW3# show interfaces status err-disabled

Port      Name      Status        Reason       Err-disabled Vlans
Gi0/0               err-disabled  bpduguard
```

---

## 4. Err-disable 복구 방법

**① 장비를 잘못 연결한 경우 (재연결 후 수동 복구)**

```cisco
interface gi0/0
 shutdown
 no shutdown
```

**② Switch를 잘못 연결한 경우 (BPDU Guard / PortFast 제거)**

```cisco
interface gi0/0
 no spanning-tree bpduguard enable
 no spanning-tree portfast
 shutdown
 no shutdown
```

**③ 자동 복구 설정 (errdisable recovery)**

errdisable recovery interval <seconds> — 복구를 시도하기까지의 대기 시간을 설정

설정 가능한 범위는 보통 30 ~ 86400초이며, 명시하지 않으면 300초가 기본값

```cisco
errdisable recovery cause bpduguard
errdisable recovery interval 300
show errdisable recovery
```

```text
%PM-4-ERR_RECOVER: Attempting to recover from bpduguard err-disable state on Gi0/1
%LINK-3-UPDOWN: Interface Gi0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Gi0/1, changed state to up
```
⬅ 이전: [06. STP Port 상태](../06_stp-port-states/stp-port-states.md) | ➡ 다음: [08. UplinkFast](../08_uplinkfast/uplinkfast.md)
