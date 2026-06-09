# VLAN (Virtual LAN)

## 개요

- Switch는 Collision Domain은 분할하지만 **Broadcast Domain은 공유**한다.
- 문제: Host 증가 → Broadcast Traffic 증가 (ARP, DHCP 등)
- 원인: 스위치의 모든 포트는 하나의 VLAN에 속함 (Default VLAN = **VLAN 1**)
- 해결: VLAN으로 **논리적 네트워크 분할**
- 장점: Broadcast Traffic 분할 / 위치·물리적 제약 없는 연결

## VLAN 구성

- VLAN은 **12bit** (0 ~ 4095)
  - `0`, `4095`: 시스템 예약
  - 사용 가능: **1 ~ 4094**

### VLAN 범위

| 구분 | 범위 | 비고 |
|------|------|------|
| Standard VLAN | 1 ~ 1005 | 1, 1002~1005 예약 |
| Extended VLAN | 1006 ~ 4094 | 장비/IOS에 따라 지원 |

Cisco Switch는 Standard VLAN을 기본 지원하며, Extended VLAN은 장비별 지원 차이가 있음.

---

## VLAN 생성/삭제

### 단일 VLAN 생성
```
en
conf t
!
vlan 10
!
end
```

### 다중 VLAN 생성
```
en
conf t
!
vlan 10,20,30
!
end
```

### VLAN 삭제
```
en
conf t
!
no vlan 10,20,30
!
end
```

### 정보 확인
```
show vlan brief
```

---

## Range 명령어

동일한 설정을 다수의 Interface에 일괄 적용

```
SW(config)# interface range fa0/1 - 5
SW(config-if)# switchport mode access
SW(config-if)# switchport access vlan 10
```

> 자세한 실습 스크립트는 [`preconfig/`](./preconfig/) 폴더 참조
