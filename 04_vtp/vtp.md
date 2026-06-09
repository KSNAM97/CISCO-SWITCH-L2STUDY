# VTP (VLAN Trunking Protocol)

## 개요

- Cisco에서 **VLAN Database 동기화** 목적으로 개발
- 한 Switch에서 VLAN을 생성/수정/삭제하면 연결된 모든 Switch에서 동기화
- 다수 Switch/VLAN 환경의 관리 부담 해소

## 사용 조건

1. Switch 간 **Trunk 연결**
2. **VTP Domain 일치** (Trunk 연결 시 자동 전파)
3. **VTP Password 일치** (각 Switch 개별 설정)

## VTP Mode

| Mode | 생성 | 삭제 | 수정 | 전파 | 일치 | 중계 |
|------|------|------|------|------|------|------|
| Server | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Client | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Transparent | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ |

> 📌 동기화 기준은 **가장 높은 Revision Number**
> 📌 Extended VLAN(1006~4094)은 **Transparent Mode에서만** 생성 가능

## 권한 용어 정리

| 용어 | 설명 |
|------|------|
| 생성 | VLAN을 만들 수 있는 권한 |
| 삭제 | VLAN을 삭제할 수 있는 권한 |
| 수정 | VLAN 정보 변경 권한 |
| 전파 | 변경된 VLAN 정보를 Trunk로 송신 |
| 일치 | 수신한 VLAN 정보로 자신의 DB 동기화 |
| 중계 | 수신한 VLAN 정보를 다른 Trunk로 중계 |

---

## VTP가 사용하는 Message

### Summary-Advertise
- VTP Server가 VLAN DB 변경 시 **Revision Number** 전파
- 변경 없을 시 **5분 주기**로 전파

### Advertise-Request
- 자신보다 높은 Revision Number 수신 시 상세 VLAN 정보 요청

### Subset-Advertise
- Advertise-Request에 대한 **응답** (VLAN DB 상세 정보 전파)

---

## VTP 설정 명령어

```
Switch(config)# vtp domain <WORD>      ! 도메인 (Trunk 통해 자동 전파)
Switch(config)# vtp password <WORD>    ! 패스워드 (개별 설정)
Switch(config)# vtp mode server | client | transparent
```

## 정보 확인

```
show vtp status
show vlan brief
```

> 실습 스크립트는 [`preconfig/vtp-preconfig.txt`](./preconfig/vtp-preconfig.txt) 참조
