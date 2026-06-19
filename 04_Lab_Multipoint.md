# 📖 보충 설명: Multipoint Sub-Interface Lab

이 실습은 **하나의 서브인터페이스에 여러 DLCI를 매핑**하여 Hub가 여러 Spoke와 통신하는 **Multipoint(NBMA)** 구조를 구성합니다.

```
                R1 (Hub)
          S0/0.1 multipoint
          10.1.1.1/24
            /    |    \
          DLCI  DLCI  DLCI
          102   103   104
           │     │     │
          R2    R3    R4
       (모두 10.1.1.0/24 동일 서브넷)
```

---

## 🎯 실습 목표

| 학습 포인트 | 설명 |
|-------------|------|
| **Multipoint 서브인터페이스** | 물리 I/F는 그대로, 서브인터페이스로 NBMA 구현 |
| **frame-relay map ... broadcast** | NBMA에서 라우팅 광고 전달 |
| **Split-Horizon 이슈** | 같은 인터페이스로 들어온 광고는 같은 인터페이스로 못 나감 |
| **NBMA의 한계** | Spoke 간 직접 통신 시 Hub 경유 필요 |

---

## ⚠️ 주의해야 할 동작

```text
1. Hub에서 Spoke A가 보낸 라우팅 광고를 Spoke B로 못 보내는 문제
   → no ip split-horizon (RIP/EIGRP)

2. OSPF는 기본 network type이 non-broadcast
   → neighbor 명령으로 수동 지정 필요
   → 또는 point-to-multipoint로 변경 권장

3. broadcast 키워드 빠뜨리면 라우팅 프로토콜 동작 안 함
   → show frame-relay map에서 "broadcast" 표기 확인
```

---

## 🔍 검증 흐름

```cisco
show frame-relay map            ! broadcast 플래그 확인
show ip route                   ! 라우팅 학습 확인
show ip ospf interface          ! Network type 확인
debug ip ospf adj               ! 인접 형성 디버깅
```

---


# 4. Lab - Frame-Relay Multipoint

> Sub-Interface Multipoint 사용  
> 하나의 네트워크(192.168.100.0/24)에 여러 대 연결 (NBMA)

## 🗺 토폴로지

```
                                 -----102----R1
                               /       |
                          /            103
                     /                 |
      201      /                       |
  R2-------                            |    192.168.100.0/24
      203      /                       |
                     /                 |
                          /            301
                               /       |
                                 -----302----R3
```

---

## ⚙ FRSW 설정

```cisco
no ip routing
!
frame-relay switching
!
interface serial 1/0
 no shutdown
 encapsulation frame-relay
 frame-relay intf-type dce
 clock rate 1612800
 frame-relay route 102 interface serial 1/1 201
 frame-relay route 103 interface serial 1/2 301
!
interface serial 1/1
 no shutdown
 encapsulation frame-relay
 frame-relay intf-type dce
 clock rate 1612800
 frame-relay route 201 interface serial 1/0 102
 frame-relay route 203 interface serial 1/2 302
!
interface serial 1/2
 no shutdown
 encapsulation frame-relay
 frame-relay intf-type dce
 clock rate 1612800
 frame-relay route 301 interface serial 1/0 103
 frame-relay route 302 interface serial 1/1 203
```

---

## ⚙ 라우터 설정

### R1
```cisco
interface serial 1/0
 no shutdown
 encapsulation frame-relay
!
interface serial 1/0.123 multipoint
 frame-relay map ip 192.168.100.2 102
 frame-relay map ip 192.168.100.3 103
 ip address 192.168.100.1 255.255.255.0
```

### R2
```cisco
interface serial 1/0
 no shutdown
 encapsulation frame-relay
!
interface serial 1/0.123 multipoint
 frame-relay map ip 192.168.100.1 201
 frame-relay map ip 192.168.100.3 203
 ip address 192.168.100.2 255.255.255.0
```

### R3
```cisco
interface serial 1/0
 no shutdown
 encapsulation frame-relay
!
interface serial 1/0.123 multipoint
 frame-relay map ip 192.168.100.1 301
 frame-relay map ip 192.168.100.2 302
 ip address 192.168.100.3 255.255.255.0
```

---

## 🔍 정보 확인

```cisco
R1# show ip route
C    192.168.123.0/24 is directly connected, Serial1/0.123
R1# ping 192.168.123.2
R1# ping 192.168.123.3


R2# show ip route
C    192.168.123.0/24 is directly connected, Serial1/0.123
R2# ping 192.168.123.1
R2# ping 192.168.123.3


R3# show ip route
C    192.168.123.0/24 is directly connected, Serial1/0.123
R3# ping 192.168.123.1
R3# ping 192.168.123.2
```
