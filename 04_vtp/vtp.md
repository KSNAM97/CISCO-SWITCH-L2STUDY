# 04. VTP (VLAN Trunking Protocol)

> Cisco에서 **VLAN Database 동기화**를 위해 개발한 Protocol을 학습합니다.

---

## 1. VTP 개념

- Cisco에서 **VLAN Database 동기화**를 목적으로 개발한 Protocol
- 다수의 Switch나 VLAN을 사용할 때 각각의 Switch에서 VLAN을 생성·수정·삭제하는 번거로움 해결
- **하나의 Switch에서 VLAN을 생성/수정/삭제**하면 → 연결된 모든 Switch에서 **VLAN Database Table 자동 동기화**

### 🔹 VTP 사용 조건

1. Switch 간 **Trunk로 연결**되어야 함
2. **VTP Domain 일치** (Trunk로 연결 시 자동 전파)
3. **VTP Password 일치** (각 Switch에서 개별 설정)

> Domain 이름과 Password가 설정되어야 VTP Mode 사용 가능.

---

## 2. VTP Mode

| Mode | 생성 | 삭제 | 수정 | 전파 | 일치 | 중계 |
| --- | :-: | :-: | :-: | :-: | :-: | :-: |
| **Server** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Client** | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| **Transparent** | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ |

VTP로 VLAN을 동기화하려면 **전파 또는 일치 권한 중 하나 이상**이 필요합니다.

### 권한 의미

| 권한 | 설명 |
| --- | --- |
| 생성 | VLAN 생성 가능 여부 |
| 삭제 | VLAN 삭제 가능 여부 |
| 수정 | VLAN 정보 변경 가능 여부 |
| **전파** | 자신이 변경한 VLAN 정보를 Trunk로 연결된 장비로 전달 |
| **일치** | Trunk Port로 수신된 VLAN 정보를 자신과 동기화 |
| 중계 | Trunk Port로 수신된 VLAN 정보를 다른 Trunk Port로 전달 |

> **가장 높은 Revision Number 기준**으로 동기화가 이루어집니다.  
> (Server Mode 장비라도 Revision이 더 높은 Client/Server를 만나면 DB가 덮어쓰여짐 → 운영상 주의)

---

## 3. VLAN 범위와 Mode 관계

- Standard VLAN: 1 ~ 1005 (모든 Mode 지원)
- **Extended VLAN: 1006 ~ 4094 → Transparent Mode에서만 생성 가능**

### 시나리오 예시

 
R: 1 R: 1 R: 0 R: 1 vtp server vtp client vtp transparent vtp server (외부와 동기화 X) SW1 ─────── SW2 ─────── SW3 ─────────────── SW4 vlan 10 vlan 10 vlan 10 vlan 10 vlan 2000 (Extended, 자체만)

 
- SW1·SW2·SW4: VTP로 동기화
- SW3: Transparent이므로 자신은 동기화 X, 중계만 함 → SW4도 VLAN 10 학습

---

## 4. VTP 설정 명령어

### 🔹 VTP Domain 설정

SW(config)# vtp domain

 > 하나의 Switch에서 설정하면 모든 Switch로 자동 전파.

### 🔹 VTP Password 설정

SW(config)# vtp password

 > 각 Switch에서 **개별 설정** 필요.

### 🔹 VTP Mode 변경

SW(config)# vtp mode server Device mode already VTP SERVER.

SW(config)# vtp mode client Setting device to VTP CLIENT mode.

SW(config)# vtp mode transparent Setting device to VTP TRANSPARENT mode.

 
---

## 5. VTP Message

| Message | 설명 |
| --- | --- |
| **Summary-Advertise** | VTP Server가 VLAN DB 변경 시 Revision Number 전파. 변경이 없어도 **5분 주기**로 전파. |
| **Advertise-Request** | Summary-Advertise를 수신한 Switch가 자신보다 **높은 Revision Number**를 발견하면 상세 정보 요청 |
| **Subset-Advertise** | Advertise-Request를 받은 Switch가 실제 **VLAN Database 정보** 전송 |

### 동작 흐름

[Server] [Client] │ Summary-Advertise (Rev=5) │ │ ─────────────────────────────────────► │ │ │ (자기 Rev=3 < 5 → 요청) │ Advertise-Request │ │ ◄───────────────────────────────────── │ │ │ │ Subset-Advertise (VLAN DB) │ │ ─────────────────────────────────────► │ (DB 동기화 완료)

 
---

## 6. 핵심 명령어 정리

| 명령어 | 설명 |
| --- | --- |
| `vtp domain <NAME>` | VTP Domain 설정 |
| `vtp password <WORD>` | VTP Password 설정 |
| `vtp mode {server\|client\|transparent}` | VTP Mode 변경 |
| `vtp version {1\|2\|3}` | VTP 버전 지정 |
| `show vtp status` | VTP 상태 확인 |
| `show vtp password` | VTP Password 확인 |

---

## 7. 운영 시 주의사항

- **새로운 Switch를 기존 VTP Domain에 추가할 때 반드시 Revision Number를 초기화** (Domain 이름을 다른 것으로 바꿨다 되돌리면 0으로 리셋).  
- Revision Number가 더 높은 장비가 도메인에 연결되면 **운영 중인 VLAN DB가 덮어써질 위험**이 있음.
- 운영 안정성이 중요한 환경에서는 **Transparent Mode**로 운영하거나, **VTP를 사용하지 않는 것**도 일반적인 선택.

---

➡ 다음: [05. STP](../05_stp/stp.md)
