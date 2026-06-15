# 5. Lab - Frame-Relay Point-to-Point

> Point-to-point로 만들어진 Interface들은 서로 다른 네트워크로 동작을 실시하게 된다.

## 🗺 토폴로지

```
                         ------------------------------------------------------
                         |                  FR-Switch                          |
                         ------------------------------------------------------
                            |       |       |       |       |       |
                          S1/0   S1/1    S1/2    S1/3    S0/0    S0/1
                            |       |       |       |       |       |
                          S1/0   S1/0    S1/0    S1/0    S1/0    S1/0
                            |       |       |       |       |       |
                           R1      R2      R3      R4      R5      R6
```

```
R1 ──── FRSW ──── R2 ──── FRSW ──── R3 ──── FRSW ──── R4
   102        201    203        302    304        403
   192.168.12.0/24    192.168.23.0/24    192.168.34.0/24
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
 frame-relay route 302 interface serial 1/1 203
 frame-relay route 304 interface serial 1/3 403
!
interface serial 1/3
 no shutdown
 encapsulation frame-relay
 frame-relay intf-type dce
 clock rate 1612800
 frame-relay route 403 interface serial 1/2 304
```

---

## ⚙ 라우터 설정

### R1
```cisco
interface serial 1/0
 no shutdown
 encapsulation frame-relay
!
interface serial 1/0.12 point-to-point 
 frame-relay interface-dlci 102
 ip address 192.168.12.1 255.255.255.0
```

### R2
```cisco
interface serial 1/0
 no shutdown
 encapsulation frame-relay
!
interface serial 1/0.12 point-to-point
 frame-relay interface-dlci 201
 ip address 192.168.12.2 255.255.255.0
!
interface serial 1/0.23 point-to-point
 frame-relay interface-dlci 203
 ip address 192.168.23.2 255.255.255.0
```

### R3
```cisco
interface serial 1/0
 no shutdown
 encapsulation frame-relay
!
interface serial 1/0.23 point-to-point 
 frame-relay interface-dlci 302
 ip address 192.168.23.3 255.255.255.0
!
interface serial 1/0.34 point-to-point 
 frame-relay interface-dlci 304
 ip address 192.168.34.3 255.255.255.0
```

### R4
```cisco
interface serial 1/0
 no shutdown
 encapsulation frame-relay
!
interface serial 1/0.34 point-to-point 
 frame-relay interface-dlci 403
 ip address 192.168.34.4 255.255.255.0
```

---

## 🔍 정보 확인

```cisco
R1# show ip route
C    192.168.12.0/24 is directly connected, Serial1/0.12
R1# ping 192.168.12.2


R2# show ip route
C    192.168.12.0/24 is directly connected, Serial1/0.12
C    192.168.23.0/24 is directly connected, Serial1/0.23
R2# ping 192.168.12.1
R2# ping 192.168.23.3


R3# show ip route
C    192.168.23.0/24 is directly connected, Serial1/0.23
C    192.168.34.0/24 is directly connected, Serial1/0.34
R3# ping 192.168.23.2
R3# ping 192.168.34.4


R4# show ip route
C    192.168.34.0/24 is directly connected, Serial1/0.34
R4# ping 192.168.34.3
```

### Frame-Relay Map 확인
```cisco
R1# show frame-relay map
Serial1/0.12 (up): point-to-point dlci, dlci 102(0x66,0x1860), broadcast
          status defined, active
```
