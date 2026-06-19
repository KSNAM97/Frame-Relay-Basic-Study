# 📖 보충 설명: EIGRP over Frame-Relay Hub & Spoke

EIGRP는 **Advanced Distance Vector** 프로토콜로, RIP보다 정교하지만 **NBMA 이슈는 동일**합니다.  
다만 EIGRP 특유의 추가 고려사항이 있습니다.

---

## ⚠️ EIGRP 특유의 NBMA 이슈

```
1. Split-Horizon (EIGRP용 별도 명령)
   → no ip split-horizon eigrp <AS>

2. Bandwidth 기반 메트릭 계산 왜곡
   → Multipoint 인터페이스의 bandwidth가 모든 PVC에 공유됨
   → bandwidth 명령으로 PVC당 적정값 설정 권장

3. Pacing (대역폭의 50% 제한)
   → 잘못된 bandwidth 설정 시 EIGRP 패킷 자체가 drop될 수 있음
```

---

## 🛠️ 핵심 설정

```cisco
interface Serial0/0.1 multipoint
 ip address 10.1.1.1 255.255.255.0
 no ip split-horizon eigrp 100      ! ← 핵심
 bandwidth 256                       ! PVC당 대역폭 명시
 frame-relay map ip 10.1.1.2 102 broadcast
 frame-relay map ip 10.1.1.3 103 broadcast
!
router eigrp 100
 no auto-summary
 network 10.0.0.0
```

---

## 🎯 학습 포인트

| 항목 | 설명 |
|------|------|
| `no ip split-horizon eigrp 100` | EIGRP는 IP용 명령과 별개 명령 필요 |
| `bandwidth` | 메트릭 + Pacing 계산에 사용 |
| `eigrp stub` | Spoke를 stub으로 지정해 SIA(Stuck-In-Active) 예방 |

---

## 🔍 검증

```cisco
show ip eigrp neighbors         ! 인접 형성 확인
show ip eigrp topology          ! Successor / FS 확인
show ip route eigrp             ! 학습 라우팅 확인
```

---


# 9. Lab - Hub&Spoke + EIGRP

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
- AS = `100`, `no auto-summary`
- 각 Router의 FastEthernet 0/1에 `192.168.x.x/24` 할당
- 모든 네트워크 구간 통신 가능

> ⚠ 사전 조건: `frame-relay map`에 **`broadcast` 옵션이 모두 적용**되어 있어야 함

---

## ⚙ 1단계 - 기본 EIGRP 설정

### R2 (Hub)
```cisco
router eigrp 100
 no auto-summary
 network 192.168.2.2 0.0.0.0
 network 192.168.123.2 0.0.0.0
```

### R1 (Spoke)
```cisco
router eigrp 100
 no auto-summary
 network 192.168.1.1 0.0.0.0
 network 192.168.123.1 0.0.0.0
```

### R3 (Spoke)
```cisco
router eigrp 100
 no auto-summary
 network 192.168.3.3 0.0.0.0
 network 192.168.123.3 0.0.0.0
```

### ⚠ 1단계 결과
```cisco
R1# show ip eigrp neighbor    ! 인접성 1개 (R2만)
R2# show ip eigrp neighbor    ! 인접성 2개 (R1, R3)
R3# show ip eigrp neighbor    ! 인접성 1개 (R2만)

R2# show ip route
C   192.168.123.0/24 is directly connected, Serial1/0.123
D   192.168.1.0/24 [90/2195456] via 192.168.123.1, Serial1/0.123
C   192.168.2.0/24 is directly connected, FastEthernet0/1
D   192.168.3.0/24 [90/2195456] via 192.168.123.3, Serial1/0.123

R1# show ip route
D   192.168.2.0/24 [90/2195456] via 192.168.123.2, Serial1/0.123
! 192.168.3.0/24 없음 ❌

R3# show ip route
D   192.168.2.0/24 [90/2195456] via 192.168.123.2, Serial1/0.123
! 192.168.1.0/24 없음 ❌
```

> **원인**: EIGRP도 **Distance Vector 계열** → Split-Horizon 작용

---

## ⚙ 2단계 - Hub에서 EIGRP Split-Horizon 해제

```cisco
! R2 (Hub)
interface serial 1/0.123
 no ip split-horizon eigrp 100
```

### ✅ 최종 결과
```cisco
R1# show ip route
C   192.168.123.0/24 is directly connected, Serial1/0.123
C   192.168.1.0/24   is directly connected, FastEthernet0/1
D   192.168.2.0/24 [90/2195456] via 192.168.123.2, Serial1/0.123
D   192.168.3.0/24 [90/2707456] via 192.168.123.2, Serial1/0.123
R1# ping 192.168.3.3 source 192.168.1.1   ! ✅


R2# show ip route
C   192.168.123.0/24 is directly connected, Serial1/0.123
D   192.168.1.0/24 [90/2195456] via 192.168.123.1, Serial1/0.123
C   192.168.2.0/24 is directly connected, FastEthernet0/1
D   192.168.3.0/24 [90/2195456] via 192.168.123.3, Serial1/0.123


R3# show ip route
C   192.168.123.0/24 is directly connected, Serial1/0.123
D   192.168.1.0/24 [90/2707456] via 192.168.123.2, Serial1/0.123
D   192.168.2.0/24 [90/2195456] via 192.168.123.2, Serial1/0.123
C   192.168.3.0/24 is directly connected, FastEthernet0/1
R3# ping 192.168.1.1 source 192.168.3.3   ! ✅
```

---

## 🧹 정리

```cisco
! R1, R2, R3
no router eigrp 100
```
