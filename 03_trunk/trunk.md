# Trunk

## Switchport Mode

### Access Mode
- 하나의 Port로 **하나의 VLAN**을 사용
- PC, Server 등 단말 장비 연결
- VLAN을 지원하지 않는 장비용 포트

### Trunk Mode
- 하나의 Port로 **다수의 VLAN** 통신
- Switch, IP Phone 등 VLAN 지원 장비 연결용
- 예: `SW — IP Phone — PC` (음성 VLAN 111, 데이터 VLAN 10)

### Dynamic Mode (DTP 협상)
- **desirable**: DTP 송/수신 모두 수행
- **auto**: DTP 수신만 (보안상 2960 이후 **기본값**)

> ⚠️ 양쪽 모두 `dynamic auto` 기본값이면 트렁크가 생성되지 않음 (Access로 동작)
> 최소 한쪽은 명시적으로 `trunk` 또는 `dynamic desirable` 설정 필요

## Trunk Encapsulation

| 종류 | 설명 |
|------|------|
| IEEE.802.1Q | LAN 구간 **표준** Trunk Protocol |
| ISL | Cisco Switch **전용** |

> Catalyst 2950 이하는 dot1q만 지원

## 802.1Q Tagging

Access Ethernet Header
```
| Destination MAC | Source MAC | Type | Data |
```

Trunk (802.1Q) Ethernet Header — **4Byte Tag 확장**
```
| Destination MAC | Source MAC | Tagging(4B) | Type | Data |
```

### Tag 4Byte 구조 (32bit)

| 필드 | 비트 | 설명 |
|------|------|------|
| EtherType | 16bit | `0x8100` (802.1Q 식별) |
| Priority (PCP) | 3bit | 우선순위 (8단계 QoS) |
| CFI | 1bit | 0=Ethernet, 1=Token-ring |
| VLAN-ID | 12bit | 0~4095, 미표기 시 Native VLAN |

---

## Router-on-a-Stick (Sub-interface)

하나의 물리 인터페이스를 논리적으로 분할.

> 💡 권장 사용률: **최대 대역폭의 60~65%**
> - Queuing Theory에 따라 사용률이 1에 가까워질수록 지연이 폭증
> - 버스트 트래픽 흡수 + 장애/성장 여유 확보 목적

자세한 명령어는 [`preconfig/router-on-a-stick.txt`](./preconfig/router-on-a-stick.txt) 참조.

---

## Allowed Command

특정 VLAN만 Forwarding하거나 제외하려는 경우 사용.

```
switchport trunk allowed vlan 1,10,20
switchport trunk allowed vlan add 40
switchport trunk allowed vlan remove 10
switchport trunk allowed vlan except 100
switchport trunk allowed vlan 1-99,101-4094
```

> 💡 중요 메시지는 VLAN 1로 전달되는 경우가 많아 **VLAN 1 허용 권장**

## 정보 확인 명령

```
show interfaces gi0/1 switchport
show interface trunk
show vlan brief
```
