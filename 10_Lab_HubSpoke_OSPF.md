# 📖 보충 설명: OSPF over Frame-Relay Hub & Spoke

OSPF는 **Link-State** 프로토콜로, **Network Type** 설정이 NBMA 환경 동작의 핵심입니다.  
잘못된 타입을 선택하면 인접조차 형성되지 않습니다.

---

## 🔑 OSPF Network Type 선택 가이드

| Type | DR/BDR | Hello | Neighbor 수동 | /32 호스트 라우트 | 적합한 환경 |
|------|--------|-------|---------------|-------------------|-------------|
| `broadcast` | ⭕ | 10s | ❌ | ❌ | NBMA 부적합 (실은 가능, 권장X) |
| `non-broadcast` | ⭕ | 30s | ⭕ 필요 | ❌ | Full-Mesh NBMA |
| `point-to-multipoint` | ❌ | 30s | ❌ | ⭕ | **Hub&Spoke 권장** |
| `point-to-multipoint non-broadcast` | ❌ | 30s | ⭕ 필요 | ⭕ | broadcast 미지원 시 |
| `point-to-point` | ❌ | 10s | ❌ | ❌ | P2P 서브인터페이스 |

---

## ⚠️ 자주 발생하는 함정

```
1. Hub와 Spoke의 network type 불일치
   → Hello interval이 달라서 인접 형성 실패
   → 양쪽 모두 동일하게 맞춰야 함

2. non-broadcast에서 neighbor 명령 누락
   → Unicast Hello를 보내야 하는데 대상 없음
   → router ospf 모드에서 neighbor 10.1.1.2 추가

3. Spoke가 DR로 선출됨
   → Spoke는 다른 Spoke와 직접 통신 불가 → LSA 흐름 깨짐
   → Spoke에 ip ospf priority 0 설정 (DR 후보 제외)
```

---

## 🛠️ 설정 예시 (point-to-multipoint 권장)

```cisco
! Hub
interface Serial0/0.1 multipoint
 ip address 10.1.1.1 255.255.255.0
 ip ospf network point-to-multipoint
 frame-relay map ip 10.1.1.2 102 broadcast
 frame-relay map ip 10.1.1.3 103 broadcast
!
router ospf 1
 network 10.1.1.0 0.0.0.255 area 0

! Spoke
interface Serial0/0.1 multipoint
 ip address 10.1.1.2 255.255.255.0
 ip ospf network point-to-multipoint
 frame-relay map ip 10.1.1.1 201 broadcast
 frame-relay map ip 10.1.1.3 201 broadcast   ! Spoke→Spoke는 Hub 경유
```

---

## 🔍 검증

```cisco
show ip ospf interface          ! Network type, Hello/Dead 확인
show ip ospf neighbor           ! 인접 상태 (FULL이어야 정상)
show ip route ospf              ! /32 호스트 라우트 확인 (P2MP 특징)
debug ip ospf adj               ! 인접 형성 과정 추적
```

---

## 💡 결론

```text
Hub&Spoke + OSPF 조합에서는 일반적으로
"point-to-multipoint"가 가장 단순하고 안정적인 선택지.

DR/BDR 신경 안 써도 되고,
neighbor 수동 지정도 필요 없고,
Spoke 추가 시 설정 변경 최소화.
```

---


# 10. Lab - Hub&Spoke + OSPF

## 🗺 토폴로지



![HubSpoke-Basic-topology](./images/HubSpoke-Basic-topology.png)

```
       R2 (Hub)
   192.168.2.0/24
        |
        | s1/0.123 multipoint
        | 192.168.123.2/24
        |
      [FR-SW]
       /     \
   R1         R3  (Spoke)
192.168.1.0   192.168.3.0
192.168.123.1 192.168.123.3
```
## 📌 조건
- Process = `100`, Area = `0`
- Router-ID = `x.x.x.x` (x = Router 번호)
- 모든 네트워크 구간에서 통신

> ⚠ 사전 조건: `frame-relay map`에 **`broadcast` 옵션 적용 완료**

---

## ⚙ 1단계 - 기본 OSPF 설정

### R2 (Hub)
```cisco
router ospf 100
 router-id 2.2.2.2
 network 192.168.2.0   0.0.0.255 area 0
 network 192.168.123.0 0.0.0.255 area 0
```

### R1 (Spoke)
```cisco
router ospf 100
 router-id 1.1.1.1
 network 192.168.1.0   0.0.0.255 area 0
 network 192.168.123.0 0.0.0.255 area 0
```

### R3 (Spoke)
```cisco
router ospf 100
 router-id 3.3.3.3
 network 192.168.3.0   0.0.0.255 area 0
 network 192.168.123.0 0.0.0.255 area 0
```

### ❌ 1단계 결과
```cisco
R1/R2/R3# show ip ospf neighbor   ! 인접성 없음
R1/R2/R3# show ip route           ! Connected만 확인
```
> NBMA 환경 → OSPF는 자동으로 NBMA Network Type으로 동작 → **DR/BDR 선출 필요**  
> Spoke ↔ Spoke 간 VC가 없어 **DR/BDR 선출이 정상적으로 이루어지지 않음**

---

## 📊 OSPF Network Type 비교

| Network Type | Hello/Dead | DR/BDR | 통신 환경 |
|--------------|-----------|--------|-----------|
| **Broadcast (BMA)** | 10 / 40 | 선출함 | Ethernet, FastEthernet |
| **Non-Broadcast (NBMA)** | 30 / 120 | 선출함 | F/R Multipoint, NBMA 구간 |
| **Point-to-Point** | 10 / 40 | 선출 안 함 | PPP, HDLC, F/R P2P |
| **Point-to-Multipoint** | 30 / 120 | 선출 안 함 | OSPF 전용 환경 |

---

## 🔑 NBMA 구간 OSPF 인접성 해결 방법

1. **NBMA Network Type 유지**: DR/BDR 선출 시 `neighbor` 명령으로 지정된 Router와 인접성 맺음
2. **2대 라우터**: Network Type을 **point-to-point**로 변경 → DR/BDR 선출 안 함
3. **3대 이상**: Network Type을 **point-to-multipoint**로 변경 → DR/BDR 선출 안 함

---

## ⚙ 2단계 - NBMA 방식 (DR/BDR 선출 + neighbor 명령)

### Spoke가 DR/BDR로 선출되지 않도록 설정

```cisco
! R2 (Hub) — DR로 선출되도록 우선순위 최대
interface serial 1/0.123
 ip ospf priority 255
```

```cisco
! R1, R3 (Spoke) — DR/BDR 선출 제외
interface serial 1/0.123
 ip ospf priority 0
```

### Hub에서 Spoke들을 `neighbor`로 명시

```cisco
! R2 (Hub)
router ospf 100
 neighbor 192.168.123.1
 neighbor 192.168.123.3
```

> NBMA 환경에서는 DR/BDR 선출 시 **`neighbor` 명령으로 지정된 라우터와 인접성**이 연결됨

---

## ✅ 검증

```cisco
R2# show ip ospf neighbor
Neighbor ID  Pri  State        Dead Time  Address         Interface
1.1.1.1      0    FULL/DROTHER 00:01:xx   192.168.123.1   Serial1/0.123
3.3.3.3      0    FULL/DROTHER 00:01:xx   192.168.123.3   Serial1/0.123

R1# show ip route
O   192.168.2.0/24 [110/xx] via 192.168.123.2, Serial1/0.123
O   192.168.3.0/24 [110/xx] via 192.168.123.2, Serial1/0.123

R3# show ip route
O   192.168.1.0/24 [110/xx] via 192.168.123.2, Serial1/0.123
O   192.168.2.0/24 [110/xx] via 192.168.123.2, Serial1/0.123
```

---

## 💡 핵심 정리

- OSPF는 **Link-State** 프로토콜이므로 Split-Horizon 영향 없음
- 그러나 NBMA 환경에서는 **Hello 패킷(멀티캐스트) 전달 문제** + **DR/BDR 선출 이슈** 발생
- 해결 방법:
  - **NBMA 방식**: `neighbor` 명령 + Hub를 DR로 강제 선출
  - **Point-to-Multipoint 방식**: `ip ospf network point-to-multipoint` (DR/BDR 없음, 더 간단)
