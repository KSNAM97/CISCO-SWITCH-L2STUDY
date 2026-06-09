# Switch 기본 동작원리

## 1. 계층별 대표 장비 비교

### Hub (Layer 1)
- Layer 1의 대표 장비
- 동일 네트워크 내 장비 간 통신에 사용
- 전기적인 신호를 사용하여 데이터를 Forwarding (신호 증폭)
- **Collision Domain 공유**

### Switch (Layer 2)
- Layer 2의 대표 장비
- 동일 네트워크 내 장비 간 통신에 사용
- Ethernet Header의 **MAC Address** 사용 (16진수, 48bit: `HH-HH-HH-HH-HH-HH`)
- 목적지 MAC을 기준으로 MAC Address Table을 참조하여 Forwarding
- **Collision Domain 분할 / Broadcast Domain 공유**
- Cisco IOS 의존 **하드웨어 기반** 장비

### Router (Layer 3)
- Layer 3의 대표 장비
- **다른 네트워크**로 통신 시 사용
- IP Header의 **IP Address** 사용 (10진수, 32bit)
- 목적지 IP를 기준으로 Routing Table을 참조하여 Forwarding
- **Broadcast Domain 분할**
- Cisco IOS 의존 **소프트웨어 기반** 장비

---

## 2. Collision / Broadcast Domain 관점

| 장비 | Collision Domain | Broadcast Domain |
|------|-----------------|-----------------|
| HUB | 공유 (1개) | 공유 (1개) |
| Switch | 포트마다 분할 | 공유 (VLAN 시 분할 가능) |
| Router | 인터페이스마다 분할 | 인터페이스마다 분할 |

## 3. 장애 발생 시 영향 범위

- **HUB 장애** → HUB에 연결된 모든 장비 통신 불가
- **Switch 장애** → 해당 스위치 연결 장비 영향, 타 세그먼트는 영향 없음
- **Router 장애** → 내부 통신은 가능, **외부 네트워크 통신 불가**

---

## 4. Switch의 동작 원리

### Flooding
- 목적지 MAC Address를 **모를 때** 수행
- 들어온 포트를 제외한 **모든 포트로 프레임 전송**
- 대상: **Unknown Unicast / Unknown Multicast(Multicast) / Broadcast Frame**

### Learning
- 프레임의 **Source MAC**을 확인하여 어느 포트에서 들어왔는지 MAC Table에 기록

### Forwarding
- 목적지 MAC을 알고 있을 때 **해당 포트로만 전송**

### Filtering
- 출발지와 목적지가 **같은 포트 방향**일 때 → 프레임 **Drop**
- 예: 같은 HUB에 PC1, PC2가 연결되어 스위치 Fa0/1로 들어온 경우
  - SA도 Fa0/1, DA도 Fa0/1 → Filtering

### Aging
- MAC Address Table에 학습된 정보가 **일정 시간 사용되지 않으면 자동 삭제**
- Cisco 기본 Aging Time: **300초 (5분)**
- 삭제 후 재통신 시 Learning 단계로 복귀

---

## 5. ARP & ICMP 동작 예시

### ARP Request (Broadcast)
```
Ethernet                    ARP Request
SA: 0011.1111.1111          Sender IP : 192.168.1.1
DA: FFFF.FFFF.FFFF          Sender MAC: 0011.1111.1111
                            Target IP : 192.168.1.4
                            Target MAC: 0000.0000.0000.0000
```

### ARP Reply (Unicast)
```
Ethernet                    ARP Reply
SA: 0044.4444.4444          Sender IP : 192.168.1.4
DA: 0011.1111.1111          Sender MAC: 0044.4444.4444
                            Target IP : 192.168.1.1
                            Target MAC: 0011.1111.1111
```

### ICMP Echo Request / Reply
```
Ethernet                IP                      ICMP
SA: 0011.1111.1111      SA: 192.168.1.1         Request (Type 8)
DA: 0044.4444.4444      DA: 192.168.1.4

Ethernet                IP                      ICMP
SA: 0044.4444.4444      SA: 192.168.1.4         Reply (Type 0)
DA: 0011.1111.1111      DA: 192.168.1.1
```

---

## 6. 주요 테이블 비교

| 장비 | 보유 테이블 |
|------|-----------|
| IP 처리 장비 (Router) | ARP Table |
| Frame 스위칭 장비 (Switch) | MAC Table |
| L3 Switch | ARP + MAC Table |
