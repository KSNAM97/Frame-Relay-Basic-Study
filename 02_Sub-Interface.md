# 📖 보충 설명: Sub-Interface란?

**Sub-Interface(서브인터페이스)** 는 하나의 **물리 인터페이스**를 논리적으로 여러 개로 나눠 쓰는 기술입니다.

```
    [ 물리 인터페이스 Serial0/0 ]
                │
    ┌───────────┼───────────┐
    │           │           │
 S0/0.102   S0/0.103    S0/0.104
 (P2P)      (P2P)       (Multipoint)
 DLCI 102   DLCI 103    DLCI 104,105
```

---

## 🎯 왜 서브인터페이스를 쓰는가?

| 이유 | 설명 |
|------|------|
| **Split-Horizon 회피** | P2P 서브인터페이스로 분리하면 Spoke→Spoke 라우팅 광고 차단 문제 해결 |
| **서브넷 분리** | DLCI마다 별도 서브넷 부여 가능 → 주소 설계 유연 |
| **OSPF 동작 단순화** | P2P 타입으로 동작 → DR/BDR 선출 없음, 인접 자동 형성 |
| **장애 격리** | 하나의 VC 장애가 다른 VC에 영향 없음 |

---

## 🔀 두 가지 모드

| 모드 | 키워드 | 특징 |
|------|--------|------|
| **Point-to-Point** | `point-to-point` | 1개 DLCI, 별도 서브넷, NBMA 이슈 없음 |
| **Multipoint** | `multipoint` | N개 DLCI, 같은 서브넷, NBMA 이슈 있음 |

```cisco
! P2P 예시
interface Serial0/0.102 point-to-point
 ip address 10.1.12.1 255.255.255.0
 frame-relay interface-dlci 102

! Multipoint 예시
interface Serial0/0.1 multipoint
 ip address 10.1.1.1 255.255.255.0
 frame-relay map ip 10.1.1.2 102 broadcast
 frame-relay map ip 10.1.1.3 103 broadcast
```

> ⚠️ 모드 선택은 **인터페이스 생성 시점에만 가능**합니다.  
> 변경하려면 서브인터페이스를 삭제 후 재생성해야 합니다.

---


# 2. Sub-Interface

## 🔹 Sub-Interface 정의

- 하나의 물리적 Interface를 여러개의 논리적인 Interface로 분할하여 사용하는 기능  
  [Frame-Relay Sub-Interface는 **V/C 단위**로 생성이 가능하다]

- Frame-Relay Sub-Interface는 **Point-to-point 방식**과 **Multipoint 방식**으로 구성이 가능하다

---

## 🔹 Interface 종류

| 구분 | 설명 | 연결 방식 |
|------|------|-----------|
| **주 Interface** | Interface를 분할하지 않은 Interface | 1:N 연결 = Multi-Access 구조 |
| **Sub-Interface Multipoint** | 논리적인 Interface로 분할한 Interface | 1:N 연결 = Multi-Access 구조 |
| **Sub-Interface Point-to-point** | 논리적인 Interface로 분할한 Interface | 1:1 연결 = Point-to-Point 구조 |

> 주 Interface는 기본적으로 Sub-interface Multipoint와 설정 및 통신방식이 동일하다

---

## 🔹 호환 매트릭스

```
주 Interface                  ↔  주 Interface
주 Interface                  ↔  Sub-Interface Multipoint
Sub-Interface Multipoint      ↔  Sub-Interface Multipoint
Sub-Interface Point-to-point  ↔  Sub-Interface Point-to-point
```

> ⚠ **Sub-Interface Point-to-point는 Sub-Interface Point-to-point로만 연결해야 한다.**
