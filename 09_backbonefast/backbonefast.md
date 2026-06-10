# 09. BackboneFast

> **모든 Switch에 공통 설정.** Root ~ Backup Root 구간(간접 장애)에서 Max-age 20초를 생략.

---

## 1. 특징

- BackboneFast는 **연결된 모든 Switch에서 공통**으로 설정해야 한다.
- Root ~ Backup Root 구간 장애 발생 시 **Blocking 유지(Max-age 20초)를 생략**한다.

```text
설정 전 : Blocking 유지(20초) → Listening(15초) → Learning(15초) → Forwarding (총 50초)
설정 후 :                       Listening(15초) → Learning(15초) → Forwarding (총 30초)
```

---

## 2. 동작 원리 (Inferior BPDU)

- **Inferior BPDU** : 기존 Root Bridge보다 **후순위(나쁜) Bridge-ID**를 가진 BPDU
- **Superior BPDU** : 기존 Root Bridge보다 **선순위(좋은) Bridge-ID**를 가진 BPDU

| 구분 | Inferior BPDU 수신 시 동작 |
| --- | --- |
| **BackboneFast 미설정** | BPDU를 **수신하지 않은 것으로 간주** → Max-age 20초 대기 후 전환 |
| **BackboneFast 설정** | Inferior BPDU를 **장애 발생으로 인식** → 바로 Listening → Learning → Forwarding |

---

## 3. 설정 전 장애 발생

```text
SW2(config)# interface fastethernet 0/22
SW2(config-if)# shutdown

! SW1 (Inferior BPDU를 약 20초간 무시 → Max-age 후 전환)
STP: VLAN0001 heard root 16385-000f.248a.3480 on Fa0/20
STP: VLAN0001 heard root 16385-000f.248a.3480 on Fa0/20
...   (약 20초간 반복)
STP: VLAN0001 Fa0/20 -> listening
STP: VLAN0001 Fa0/20 -> learning
STP: VLAN0001 Fa0/20 -> forwarding
```

---

## 4. 설정 후 장애 발생

```cisco
SW1(config)# spanning-tree backbonefast
SW2(config)# spanning-tree backbonefast
SW3(config)# spanning-tree backbonefast
```

```text
SW2(config)# interface fastethernet 0/22
SW2(config-if)# shutdown

! SW1 (Max-age 20초 대기 없이 바로 전환)
STP: VLAN0001 heard root 16385-000f.248a.3480 on Fa0/20
STP: VLAN0001 Fa0/20 -> listening
STP: VLAN0001 Fa0/20 -> learning
STP: VLAN0001 Fa0/20 -> forwarding
```

---

## 📌 STP 가속 기능 비교

| 기능 | 설정 위치 | 적용 구간 | 효과 |
| --- | --- | --- | --- |
| **PortFast** | Edge Port (PC/Server) | 단말 연결 포트 | 즉시 Forwarding (STP 미동작) |
| **UplinkFast** | Block Port 가진 Switch | **직접 연결(RP) 장애** | Block → 즉시 Forwarding |
| **BackboneFast** | 모든 Switch | **간접(Backbone) 장애** | Max-age 20초 생략 (50→30초) |
