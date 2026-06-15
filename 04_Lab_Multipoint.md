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
