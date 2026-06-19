# 📖 보충 설명: Full-Mesh Lab의 목적

이 실습은 **물리 인터페이스에 여러 DLCI를 직접 매핑**하여 모든 라우터가 서로 직접 통신하는 **Full-Mesh** 구조를 만듭니다.

```
        R1 ───── R2
        │ ╲    ╱ │
        │  ╲  ╱  │
        │   ╳    │      ← 모든 노드가 서로 PVC로 직결
        │  ╱  ╲  │
        │ ╱    ╲ │
        R4 ───── R3
```

---

## 🎯 실습 목표

| 학습 포인트 | 설명 |
|-------------|------|
| **frame-relay map** | DLCI ↔ 상대 IP 수동 매핑 방식 이해 |
| **Inverse-ARP** | 자동 DLCI ↔ IP 매핑 동작 확인 |
| **NBMA 특성** | 같은 서브넷에 여러 노드, Broadcast 미지원 |
| **VC 수 계산** | N×(N-1)/2 → 노드 증가 시 PVC 폭증 체감 |

---

## ⚙️ 핵심 명령어 미리보기

```cisco
interface Serial0/0
 encapsulation frame-relay
 ip address 10.1.1.1 255.255.255.0
 frame-relay map ip 10.1.1.2 102 broadcast
 frame-relay map ip 10.1.1.3 103 broadcast
 frame-relay map ip 10.1.1.4 104 broadcast
 no frame-relay inverse-arp     ! 수동 매핑 시 권장
```

---

## 🔍 확인 명령어

```cisco
show frame-relay pvc            ! PVC 상태 (ACTIVE/INACTIVE/DELETED)
show frame-relay map            ! DLCI ↔ IP 매핑 테이블
show frame-relay lmi            ! LMI 상태 카운터
ping 10.1.1.2                   ! 연결성 검증
```

> 💡 `broadcast` 키워드가 없으면 라우팅 프로토콜의 Hello/Update가 전달되지 않습니다.

---


# 3. Lab - Frame-Relay Full-Mesh

> **하나의 Interface를 사용하여 동일 네트워크상으로 여러대의 장비를 연결하는 기능 (NBMA)**  
> Frame-Relay Multipoint는 **Full-mesh 방식**과 **Hub&Spoke 방식**으로 구성이 가능하다.

## 🔹 Frame-Relay Full-Mesh
- Frame-Relay로 연결된 모든 Frame-Relay Router 간 **V/C를 연결**하는 구조

## 🗺 토폴로지

```
	-----------------------------------------------------------------
	|                  Frame-relay Switch                            |
	-----------------------------------------------------------------
		 |       |       |       |       |       |
	      S1/0    S1/1    S1/2    S1/3    S0/0    S0/1
		 |       |       |       |       |       |
	      S1/0    S1/0    S1/0    S1/0    S1/0    S1/0
		 |       |       |       |       |       |
		R1      R2      R3      R4      R5      R6
```

```
                                R4
                            401 |  402
                              S1/0
                                |
        104                     |                       204
        102                     |                       201
        S1/0          S1/0      |   S1/1          S1/0
   R1 ───────────── FRSW ───────────────────────────────── R2
        103                     |                       203
                              S1/2
                                |
                              S1/0
                            301 |  302
                                R3
```

---

## ⚙ FRSW 설정

```cisco
ip routing
!
frame-relay switching
!
interface serial 1/0
 no shutdown
 encapsulation frame-relay        ! Layer 2 Protocol 지정
 frame-relay intf-type dce        ! 해당 Interface를 DCE로 지정
 clock rate 1612800               ! 대역폭에 따른 전송 속도 지정
 frame-relay route 102 interface serial 1/1 201
 frame-relay route 103 interface serial 1/2 301
 frame-relay route 104 interface serial 1/3 401
!
interface serial 1/1
 no shutdown
 encapsulation frame-relay
 frame-relay intf-type dce
 clock rate 1612800
 frame-relay route 201 interface serial 1/0 102
 frame-relay route 203 interface serial 1/2 302
 frame-relay route 204 interface serial 1/3 402
!
interface serial 1/2
 no shutdown
 encapsulation frame-relay
 frame-relay intf-type dce
 clock rate 1612800
 frame-relay route 301 interface serial 1/0 103
 frame-relay route 302 interface serial 1/1 203
 frame-relay route 304 interface serial 1/3 403
!
interface serial 1/3
 no shutdown
 encapsulation frame-relay
 frame-relay intf-type dce
 clock rate 1612800
 frame-relay route 401 interface serial 1/0 104
 frame-relay route 402 interface serial 1/1 204
 frame-relay route 403 interface serial 1/2 304
```

---

## ⚙ 라우터 설정

### R1
```cisco
interface serial 1/0
 no shutdown
 encapsulation frame-relay
 ip address 192.168.10.1 255.255.255.0
 frame-relay map ip 192.168.10.2 102
 frame-relay map ip 192.168.10.3 103
 frame-relay map ip 192.168.10.4 104
```

### R2
```cisco
interface serial 1/0
 no shutdown
 encapsulation frame-relay
 ip address 192.168.10.2 255.255.255.0
 frame-relay map ip 192.168.10.1 201
 frame-relay map ip 192.168.10.3 203
 frame-relay map ip 192.168.10.4 204
```

### R3
```cisco
interface serial 1/0
 no shutdown
 encapsulation frame-relay
 ip address 192.168.10.3 255.255.255.0
 frame-relay map ip 192.168.10.1 301
 frame-relay map ip 192.168.10.2 302
 frame-relay map ip 192.168.10.4 304
```

### R4
```cisco
interface serial 1/0
 no shutdown
 encapsulation frame-relay
 ip address 192.168.10.4 255.255.255.0
 frame-relay map ip 192.168.10.1 401
 frame-relay map ip 192.168.10.2 402
 frame-relay map ip 192.168.10.3 403
```

---

## 🔍 정보 확인

```cisco
R1# show ip route
C    192.168.10.0/24 is directly connected, Serial1/0
R1# ping 192.168.10.2
R1# ping 192.168.10.3
R1# ping 192.168.10.4


R2# show ip route
C    192.168.10.0/24 is directly connected, Serial1/0
R2# ping 192.168.10.1
R2# ping 192.168.10.3
R2# ping 192.168.10.4


R3# show ip route
C    192.168.10.0/24 is directly connected, Serial1/0
R3# ping 192.168.10.1
R3# ping 192.168.10.2
R3# ping 192.168.10.4


R4# show ip route
C    192.168.10.0/24 is directly connected, Serial1/0
R4# ping 192.168.10.1
R4# ping 192.168.10.2
R4# ping 192.168.10.3
```

### Frame-Relay Map 확인
```cisco
R1# show frame-relay map 
Serial1/0 (up): ip 192.168.10.2 dlci 102(0x66,0x1860), static,
              CISCO, status defined, active
Serial1/0 (up): ip 192.168.10.3 dlci 103(0x67,0x1870), static,
              CISCO, status defined, active
Serial1/0 (up): ip 192.168.10.4 dlci 104(0x68,0x1880), static,
              CISCO, status defined, active
```

### Frame-Relay Route 확인
```cisco
FRSW# show frame-relay route
Input Intf   Input Dlci   Output Intf   Output Dlci   Status
Serial1/0    102          Serial1/1     201           active
Serial1/0    103          Serial1/2     301           active
Serial1/0    104          Serial1/3     401           active
Serial1/1    201          Serial1/0     102           active
Serial1/1    203          Serial1/2     302           active
Serial1/1    204          Serial1/3     402           active
Serial1/2    301          Serial1/0     103           active
Serial1/2    302          Serial1/1     203           active
Serial1/2    304          Serial1/3     403           active
Serial1/3    401          Serial1/0     104           active
Serial1/3    402          Serial1/1     204           active
Serial1/3    403          Serial1/2     304           active
```
