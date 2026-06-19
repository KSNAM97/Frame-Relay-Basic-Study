# 📖 보충 설명: NBMA 환경의 라우팅 프로토콜 이슈

**NBMA(Non-Broadcast Multi-Access)** 는 이름 그대로 "여러 노드가 연결되지만 Broadcast가 안 되는" 환경입니다.  
이 특성 때문에 **모든 라우팅 프로토콜이 기본 가정과 어긋나는** 문제가 발생합니다.

---

## ⚠️ 핵심 이슈 3가지

```
1. Broadcast/Multicast 전달 불가
   → 라우팅 프로토콜의 Hello/Update가 안 감
   → 해결: frame-relay map ... broadcast

2. Split-Horizon 차단
   → Hub가 받은 광고를 같은 인터페이스의 다른 Spoke로 못 보냄
   → 해결: no ip split-horizon (RIP/EIGRP)

3. DR/BDR 선출 문제 (OSPF)
   → Spoke가 DR이 되면 모든 트래픽이 잘못된 경로로 흐름
   → 해결: network type 변경 또는 priority 0
```

---

## 🛠️ 프로토콜별 대처법 요약

| 프로토콜 | 이슈 | 해결 방법 |
|----------|------|-----------|
| **RIPv2** | Split-Horizon | `no ip split-horizon` |
| **EIGRP** | Split-Horizon, Bandwidth | `no ip split-horizon eigrp <AS>` |
| **OSPF** | Network Type, DR/BDR | `ip ospf network` 변경, `neighbor` 명령 |

---

## 🔑 OSPF Network Type 비교

| Type | Hello/Dead | DR/BDR | Neighbor 수동지정 | 추천 환경 |
|------|------------|--------|-------------------|-----------|
| `broadcast` | 10/40 | ⭕ | ❌ | 이더넷 (NBMA 부적합) |
| `non-broadcast` | 30/120 | ⭕ | ⭕ 필요 | Full-Mesh NBMA |
| `point-to-multipoint` | 30/120 | ❌ | ❌ | **Hub&Spoke 권장** |
| `point-to-point` | 10/40 | ❌ | ❌ | P2P 서브인터페이스 |

---

## 💡 실무 팁

```text
Hub&Spoke 환경에서는 point-to-multipoint가 대체로 가장 단순하고 안정적
→ DR/BDR 없음, neighbor 명령 없음, /32 host route 자동 생성
→ Spoke가 늘어나도 설정 변경 최소화
```

---


# 7. NBMA 환경에서의 Dynamic Routing 이슈

## 🔹 NBMA (Non-Broadcast Multi-Access)

- Frame-Relay Multipoint 인터페이스는 **NBMA 네트워크**이다.
- NBMA는 이름 그대로 **브로드캐스트 트래픽을 기본적으로 지원하지 않으며**,  
  브로드캐스트 및 멀티캐스트 패킷이 **자동으로 전달되지 않는다**.
- 따라서 Ethernet 환경과 달리 **동적 라우팅 프로토콜이 사용하는 멀티캐스트 패킷이 원격 라우터까지 전달되지 못한다**.

---

## 🔹 동적 라우팅 프로토콜의 멀티캐스트 주소

| Protocol | Multicast Address |
|----------|-------------------|
| RIPv2 | `224.0.0.9` |
| EIGRP | `224.0.0.10` |
| OSPF | `224.0.0.5`, `224.0.0.6` |

---

## 🔹 문제 현상

- Frame-Relay Map에 **broadcast 옵션이 설정되어 있지 않으면**  
  멀티캐스트 패킷이 DLCI를 통해 전달되지 않는다.
- 이 경우 **Ping(유니캐스트)은 정상**이지만,  
  **RIPv2 업데이트 / EIGRP Hello / OSPF Hello (멀티캐스트)는 전달되지 않는다**.
- 결과적으로 **동적 라우팅 프로토콜이 이웃(Neighbor)을 형성하지 못하며**  
  Routing Table에는 **Connected Route만 표시**된다.

---

## 🔹 해결책 1 - `broadcast` 옵션 추가

NBMA 구간에서 Dynamic Routing Protocol을 구성하기 위해서는  
`frame-relay map` 설정 시 **`broadcast`** 옵션을 추가해야 한다.

```cisco
interface serial 1/0.123 multipoint
 frame-relay map ip 192.168.123.1 102 broadcast
 frame-relay map ip 192.168.123.3 103 broadcast
```

---

## 🔹 문제 현상 2 - Split-Horizon

- `broadcast` 옵션 설정 후에도 **Spoke ↔ Spoke 네트워크 정보는 교환되지 않는다**.
- **원인**: Distance Vector 계열(RIP, EIGRP)의 **Split-Horizon** 기능
  - "동일 인터페이스로 수신한 정보를 같은 인터페이스로 다시 송신하지 않는다"
- Hub(R2)가 R1에게서 Sub-Interface(`s1/0.123`)로 수신한 `192.168.1.0/24` 정보를  
  동일한 Sub-Interface로 연결된 R3에게 송신하지 못한다.

---

## 🔹 주 Interface vs Sub-Interface Split-Horizon

| 인터페이스 종류 | Split-Horizon |
|-----------------|---------------|
| 주 Interface | **OFF** (기본) |
| Sub-Interface (Multipoint / P2P) | **ON** (기본) |

---

## 🔹 해결책 2 - Hub에서 Split-Horizon 해제

### RIP
```cisco
interface serial 1/0.123
 no ip split-horizon
```

### EIGRP
```cisco
interface serial 1/0.123
 no ip split-horizon eigrp 100
```

> OSPF는 Link-State Protocol이므로 Split-Horizon 영향을 받지 않음  
> 대신 **DR/BDR 선출 이슈**와 **Network Type 설정**이 필요함 (Lab 10 참조)
