# 📖 보충 설명: RIPv2 over Frame-Relay Hub & Spoke

RIPv2는 **Distance Vector** 프로토콜이며, **Split-Horizon이 기본 활성화**되어 있습니다.  
NBMA 환경에서는 이 두 가지 특성이 모두 발목을 잡습니다.

---

## ⚠️ 주요 이슈

```
[Split-Horizon 문제 시나리오]

  Spoke1 → Hub : "10.1.1.0/24 광고"
  Hub    →  ?  : Spoke2에게 이 광고를 못 보냄 (같은 I/F로 들어왔으니까)

  결과: Spoke1 ↔ Spoke2 라우팅 불가
```

---

## 🛠️ 해결 방법

```cisco
interface Serial0/0.1 multipoint
 no ip split-horizon          ! ← 핵심
 ip address 10.1.1.1 255.255.255.0
 frame-relay map ip 10.1.1.2 102 broadcast
 frame-relay map ip 10.1.1.3 103 broadcast
!
router rip
 version 2
 no auto-summary
 network 10.0.0.0
```

---

## 🎯 학습 포인트

| 항목 | 설명 |
|------|------|
| `broadcast` 키워드 | Multicast `224.0.0.9` 전달용 |
| `no ip split-horizon` | Spoke 간 라우팅 광고 가능하게 |
| `no auto-summary` | Classless 동작 강제 |
| `version 2` | Subnet mask 정보 전달 |

---

## 🔍 검증

```cisco
show ip route rip               ! Spoke가 다른 Spoke 망 학습했는지
show ip protocols               ! split-horizon 상태 확인
debug ip rip                    ! 광고 송수신 확인
```

---


# 8. Lab - Hub&Spoke + RIPv2

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
- 각 Router의 FastEthernet 0/1에 `192.168.x.x/24` 할당 (x = Router 번호)
- RIPv2 사용, 모든 네트워크 구간에서 통신
- `no auto-summary` 적용

---

## ⚙ 1단계 - 기본 RIP 설정

### R1
```cisco
interface fastethernet 0/1
 no shutdown
 ip address 192.168.1.1 255.255.255.0
!
router rip
 version 2
 no auto-summary
 network 192.168.1.0
 network 192.168.123.0
```

### R2 (Hub)
```cisco
interface fastethernet 0/1
 no shutdown
 ip address 192.168.2.2 255.255.255.0
!
router rip
 version 2
 no auto-summary
 network 192.168.2.0
 network 192.168.123.0
```

### R3
```cisco
interface fastethernet 0/1
 no shutdown
 ip address 192.168.3.3 255.255.255.0
!
router rip
 version 2
 no auto-summary
 network 192.168.3.0
 network 192.168.123.0
```

### ❌ 1단계 결과
```cisco
R1# show ip route
C   192.168.123.0/24 is directly connected, Serial1/0.123
C   192.168.1.0/24   is directly connected, FastEthernet0/1
! RIPv2 정보 없음
```
> NBMA 환경 → 멀티캐스트(224.0.0.9) 미전달 → RIP 업데이트 실패

---

## ⚙ 2단계 - `broadcast` 옵션 추가

### R2 (Hub)
```cisco
interface serial 1/0.123 multipoint
 ip address 192.168.123.2 255.255.255.0
 frame-relay map ip 192.168.123.1 201 broadcast
 frame-relay map ip 192.168.123.3 203 broadcast
```

### R1 (Spoke)
```cisco
interface serial 1/0.123 multipoint
 ip address 192.168.123.1 255.255.255.0
 frame-relay map ip 192.168.123.2 102 broadcast
 frame-relay map ip 192.168.123.3 102 broadcast
```

### R3 (Spoke)
```cisco
interface serial 1/0.123 multipoint
 ip address 192.168.123.3 255.255.255.0
 frame-relay map ip 192.168.123.1 302 broadcast
 frame-relay map ip 192.168.123.2 302 broadcast
```

### ⚠ 2단계 결과 - 부분적 라우팅
```cisco
R2# show ip route
C   192.168.123.0/24 is directly connected, Serial1/0.123
R   192.168.1.0/24 [120/1] via 192.168.123.1, Serial1/0.123
C   192.168.2.0/24 is directly connected, FastEthernet0/1
R   192.168.3.0/24 [120/1] via 192.168.123.3, Serial1/0.123

R1# show ip route
C   192.168.1.0/24 is directly connected, FastEthernet0/1
R   192.168.2.0/24 [120/1] via 192.168.123.2, Serial1/0.123
! R3 네트워크(192.168.3.0/24) 정보 없음 ❌

R3# show ip route
! R1 네트워크(192.168.1.0/24) 정보 없음 ❌
```
> **원인**: Sub-Interface Split-Horizon ON → Hub가 받은 정보를 같은 sub-if로 재송신 못함

---

## ⚙ 3단계 - Hub에서 Split-Horizon 해제

```cisco
! R2 (Hub)
interface serial 1/0.123
 no ip split-horizon
```

### ✅ 최종 결과
```cisco
R1# show ip route
C   192.168.123.0/24 is directly connected, Serial1/0.123
C   192.168.1.0/24   is directly connected, FastEthernet0/1
R   192.168.2.0/24 [120/1] via 192.168.123.2, Serial1/0.123
R   192.168.3.0/24 [120/2] via 192.168.123.3, Serial1/0.123
R1# ping 192.168.3.3 source 192.168.1.1   ! ✅


R2# show ip route
C   192.168.123.0/24 is directly connected, Serial1/0.123
R   192.168.1.0/24 [120/1] via 192.168.123.1, Serial1/0.123
C   192.168.2.0/24 is directly connected, FastEthernet0/1
R   192.168.3.0/24 [120/1] via 192.168.123.3, Serial1/0.123


R3# show ip route
C   192.168.123.0/24 is directly connected, Serial1/0.123
R   192.168.1.0/24 [120/2] via 192.168.123.1, Serial1/0.123
R   192.168.2.0/24 [120/1] via 192.168.123.2, Serial1/0.123
C   192.168.3.0/24 is directly connected, FastEthernet0/1
R3# ping 192.168.1.1 source 192.168.3.3   ! ✅
```

---

## 🧹 정리 (다음 실습 전 RIP 삭제)

```cisco
! R1, R2, R3
no router rip
```
