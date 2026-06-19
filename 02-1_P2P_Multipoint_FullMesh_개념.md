# 📚 Point-to-Point / Multipoint / Full-Mesh 개념 정리

Frame-Relay를 공부할 때 가장 헷갈리는 세 가지 개념을 정리합니다.  
이 셋은 **서로 다른 계층의 개념**이므로 구분해서 이해하는 것이 중요합니다.

| 구분 | 의미하는 것 | 비유 |
|------|-------------|------|
| **Full-Mesh** | 노드 간 연결 **구조 (Topology)** | "누가 누구와 친구인가" |
| **Multipoint** | 하나의 인터페이스가 **여러 상대와 통신하는 방식** | "단톡방 (1:N)" |
| **Point-to-Point** | 하나의 인터페이스가 **한 상대와만 통신하는 방식** | "1:1 DM" |

> ⚠️ **Full-Mesh ≠ Multipoint ≠ Sub-Interface**  
> Full-Mesh는 "토폴로지", Multipoint/P2P는 "인터페이스 동작 방식",  
> Sub-Interface는 "물리 인터페이스를 논리적으로 나누는 방법"입니다.

---

## 1️⃣ Point-to-Point (P2P)

하나의 인터페이스(또는 서브인터페이스)가 **오직 하나의 상대 노드**와만 통신하는 방식입니다.

```
   R1 ────────── R2
 (S0/0.1 P2P)  (S0/0.1 P2P)
   DLCI 102      DLCI 201
```

### ✅ 특징
- **하나의 서브인터페이스 = 하나의 DLCI = 하나의 상대방**
- 각 P2P 서브인터페이스마다 **별도의 서브넷** 할당
- **Split-Horizon 영향 없음** → Distance Vector 라우팅 프로토콜에 유리
- OSPF Network Type 기본값: `point-to-point` (DR/BDR 선출 없음)

### ✅ 설정 예시
```cisco
interface Serial0/0
 encapsulation frame-relay
 no shutdown
!
interface Serial0/0.102 point-to-point
 ip address 10.1.12.1 255.255.255.0
 frame-relay interface-dlci 102
!
interface Serial0/0.103 point-to-point
 ip address 10.1.13.1 255.255.255.0
 frame-relay interface-dlci 103
```

---

## 2️⃣ Multipoint

하나의 인터페이스(또는 서브인터페이스)가 **여러 상대 노드와 동시에 통신**하는 방식입니다.  
Frame-Relay의 **NBMA(Non-Broadcast Multi-Access)** 특성을 그대로 가집니다.

```
            R1 (S0/0 multipoint)
           /     |     \
         DLCI  DLCI   DLCI
         102   103    104
          |     |      |
         R2    R3     R4
       (모두 같은 서브넷: 10.1.1.0/24)
```

### ✅ 특징
- **하나의 인터페이스에 여러 DLCI**가 매핑됨
- 모든 상대 노드가 **같은 서브넷**을 공유
- **NBMA 환경** → Broadcast/Multicast 기본적으로 전달 불가
  - `frame-relay map ip ... broadcast` 옵션 필요
- **Split-Horizon 이슈 발생** (Hub&Spoke에서 Spoke→Spoke 광고 차단됨)
  - 해결: `no ip split-horizon` (RIP/EIGRP)
- OSPF Network Type: `non-broadcast` 또는 `point-to-multipoint` 권장

### ✅ 설정 예시
```cisco
interface Serial0/0.1 multipoint
 ip address 10.1.1.1 255.255.255.0
 frame-relay map ip 10.1.1.2 102 broadcast
 frame-relay map ip 10.1.1.3 103 broadcast
 frame-relay map ip 10.1.1.4 104 broadcast
 no ip split-horizon
```

---

## 3️⃣ Full-Mesh

**토폴로지(Topology)** 개념입니다.  
N개의 노드가 있을 때, **모든 노드가 서로 직접 VC로 연결**되어 있는 구조를 말합니다.

```
        R1 ───── R2
        │ ╲    ╱ │
        │  ╲  ╱  │
        │   ╳    │
        │  ╱  ╲  │
        │ ╱    ╲ │
        R4 ───── R3
```

### ✅ 특징
- 연결 수: **N × (N-1) / 2**개의 VC 필요
- 장점: 모든 노드 간 직접 통신 가능, 경로 최적
- 단점: PVC 관리 부담 큼 → 노드 수 많아지면 비현실적
- 대안: **Partial-Mesh** 또는 **Hub & Spoke**

---

## 🔄 세 개념의 관계

```
┌──────────────────────────────────────────────────┐
│ Topology Layer : Full-Mesh / Partial / Hub&Spoke │ ← 무엇을 만들 것인가
├──────────────────────────────────────────────────┤
│ Interface Layer: Multipoint / Point-to-Point     │ ← 어떻게 동작하는가
├──────────────────────────────────────────────────┤
│ Physical Layer : Sub-Interface / Physical I/F    │ ← 어떻게 나눌 것인가
└──────────────────────────────────────────────────┘
```

📌 **한 줄 요약**  
> *Full-Mesh는 "그림(구조)", Multipoint/P2P는 "동작 방식", Sub-Interface는 "나누는 방법"이다.*
