# 14. Port-security

> Switch Port에 접속할 수 있는 장비를 **MAC Address 기준으로 제한**하는 L2 보안 기능.

---

## 1. 개요

- 관리자는 자신이 관리하는 모든 네트워크의 **물리적 구성도와 경로**를 파악하고 있어야 한다.
- Port-Security는 Switch의 특정 Port에 접속 가능한 장비를 **MAC Address 기준으로 제한**한다.
- Switch Port로 Frame이 입력되면 Switch는 해당 Frame의 **Source MAC Address**를 확인하여
  - 허용된 MAC이면 통신을 처리하고,
  - 허용되지 않은 MAC이면 **Violation 정책**에 따라 트래픽을 폐기하거나 Port를 err-disable로 전환한다.

---

## 2. 명령어 구조

```cisco
interface fastethernet 0/x
 switchport mode access
 switchport port-security maximum X                       ! 허용할 MAC 개수
 switchport port-security mac-address HHHH.HHHH.HHHH      ! 허용할 MAC
 switchport port-security violation {protect|restrict|shutdown}
 switchport port-security                                 ! 기능 활성화 (마지막에)
```

### 🔹 Violation 모드 비교

| 모드 | 동작 | Log 출력 | Counter 증가 |
| --- | --- | --- | --- |
| **protect** | 미허용 MAC 트래픽만 폐기 | ✗ | ✗ |
| **restrict** | 미허용 MAC 트래픽 폐기 + **Log 출력** | ✓ | ✓ |
| **shutdown** (기본값) | Port를 **err-disable** 전환 | ✓ | ✓ |

> 마지막 줄의 `switchport port-security` 명령이 있어야 **기능이 활성화**된다.

---

## 3. EX1) Port별 허용 MAC 1개 등록 + Shutdown 모드

```cisco
# SW1
interface range fa0/1 - 20
 spanning-tree portfast
!
interface fastethernet 0/1
 switchport mode access
 switchport port-security maximum 1
 switchport port-security mac-address 00D0.BC6D.42E3
 switchport port-security violation shutdown
 switchport port-security
!
interface fastethernet 0/2
 switchport mode access
 switchport port-security maximum 1
 switchport port-security mac-address 0002.4A7A.D77B
 switchport port-security violation shutdown
 switchport port-security
!
interface fastethernet 0/3
 switchport mode access
 switchport port-security maximum 1
 switchport port-security mac-address 00D0.FF5B.9439
 switchport port-security violation shutdown
 switchport port-security
!
interface fastethernet 0/20
 switchport mode access
 switchport port-security maximum 1
 switchport port-security mac-address 0050.0F8E.BB57
 switchport port-security violation shutdown
 switchport port-security
```

**개별 Port 확인**

```text
SW1# show port-security interface fastEthernet 0/1
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Aging Time                 : 0 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 1
Total MAC Addresses        : 1
Configured MAC Addresses   : 1
Sticky MAC Addresses       : 0
Last Source Address:Vlan   : 00D0.BC6D.42E3:1
Security Violation Count   : 0
```

**전체 요약**

```text
SW1# show port-security
Secure Port  MaxSecureAddr  CurrentAddr  SecurityViolation  Security Action
                 (Count)       (Count)        (Count)
--------------------------------------------------------------------
   Fa0/1          1             1              0            Shutdown
   Fa0/2          1             1              0            Shutdown
   Fa0/3          1             1              0            Shutdown
   Fa0/20         1             1              0            Shutdown
```

| 필드 | 의미 |
| --- | --- |
| **MaxSecureAddr** | 최대 허용 MAC Address 개수 |
| **CurrentAddr** | 현재 Secure MAC으로 등록된 개수 |
| **SecurityViolation** | 위반 발생 횟수 |
| **Security Action** | 위반 시 동작 |

---

## 4. EX2) Violation 모드 변경 (Shutdown → Restrict)

> err-disable로 끊지 않고 **로그만 남기고 폐기**하도록 변경.

```cisco
# SW1
interface fastethernet 0/1
 switchport port-security violation restrict
 switchport port-security
!
interface fastethernet 0/2
 switchport port-security violation restrict
 switchport port-security
!
interface fastethernet 0/3
 switchport port-security violation restrict
 switchport port-security
!
interface fastethernet 0/20
 switchport port-security violation restrict
 switchport port-security
```

---

## 5. Port-security Sticky

> 처음 입력된 Source MAC을 **자동으로 학습 후 Secure MAC으로 등록**하는 기능.

### 🔹 등장 배경

- 일반 Port-Security는 관리자가 허용할 MAC을 **직접 입력**해야 한다.
- Port 수가 많고 PC가 다수인 환경에서는 모든 MAC을 일일이 확인/등록하는 것이 번거롭다.
- 이때 사용하는 기능이 **Sticky**이다.

### 🔹 동작 방식

1. Sticky가 설정된 Port로 Frame이 입력되면 Switch가 Source MAC을 확인한다.
2. 해당 Port에 아직 Secure MAC이 등록되어 있지 않다면, **처음 입력된 MAC을 자동으로 Secure MAC으로 등록**한다.
3. 이후 등록된 MAC 외 다른 MAC이 들어오면 Violation 정책에 따라 처리한다.

### 🔹 설정 예시

```cisco
# SW1
interface fastethernet 0/1
 switchport mode access
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 switchport port-security
```

**확인** — 학습된 MAC이 `running-config`에 `sticky`로 등록된다.

```text
SW1# show running-config interface fa0/1
interface FastEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 00D0.BC6D.42E3   ← 자동 학습
```

> 💡 Sticky로 학습된 MAC은 `write memory`로 저장해야 재부팅 후에도 유지된다.

---

🏁 모든 학습 완료. [메인으로](../README.md)
