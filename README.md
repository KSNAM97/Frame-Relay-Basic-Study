![Cisco](https://img.shields.io/badge/Cisco-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Frame-Relay](https://img.shields.io/badge/Frame--Relay-FF6600?style=for-the-badge&logo=cisco&logoColor=white)
![WAN](https://img.shields.io/badge/WAN-Layer2-blue?style=for-the-badge)
![Routing](https://img.shields.io/badge/Routing-RIP%2FEIGRP%2FOSPF-green?style=for-the-badge)
![GNS3](https://img.shields.io/badge/GNS3-Lab-success?style=for-the-badge&logo=gnome-terminal&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)

# 🌐 Cisco Frame-Relay Study

**Cisco Frame-Relay (DLCI / LMI / VC / Sub-Interface / NBMA / Hub & Spoke / Dynamic Routing)**

> - **Part 1 Frame-Relay 기초 (L2 구성)** ✅  
> - **Part 2 Frame-Relay + Dynamic Routing (RIP / EIGRP / OSPF)** ✅

---

Cisco IOS 기반 **WAN Layer 2 Protocol** 중 **Frame-Relay** 학습 정리 저장소입니다.  
**DLCI / LMI / Sub-Interface / Multipoint / Point-to-Point** 기본 구성부터,  
**NBMA 환경에서의 Dynamic Routing Protocol (RIPv2, EIGRP, OSPF)** 동작 원리와 트러블슈팅까지 다룹니다.

|  |  |
| --- | --- |
| **대상 장비** | Cisco IOS Router (c7200), Frame-Relay Switch |
| **주요 기술** | Frame-Relay, DLCI, LMI, Sub-Interface, NBMA, Hub&Spoke, RIPv2, EIGRP, OSPF |
| **실습 환경** | GNS3 / Dynamips |
| **핵심 명령어** | `frame-relay map ip ... broadcast`, `no ip split-horizon`, `ip ospf network`, `neighbor` |

---

## 📁 디렉토리 구조

```
NETWORK-BASIC-STUDY/
└── WAN/Frame-Relay/
    ├── 01_개념정리.md
    ├── 02_Sub-Interface.md
    ├── 03_Lab_Full-Mesh.md
    ├── 04_Lab_Multipoint.md
    ├── 05_Lab_Point-to-Point.md
    ├── 06_Lab_종합실습.md
    ├── 07_NBMA_라우팅이슈.md
    ├── 08_Lab_HubSpoke_RIP.md
    ├── 09_Lab_HubSpoke_EIGRP.md
    ├── 10_Lab_HubSpoke_OSPF.md
    └── images/
```

---

## 📚 챕터 (Chapters)

### Part 1 — Frame-Relay 기초

| # | 주제 | 설명 |
|---|------|------|
| 01 | [개념 정리](./01_개념정리.md) | WAN L2 Protocol, DLCI / LMI / VC / PVC·SVC / Status |
| 02 | [Sub-Interface](./02_Sub-Interface.md) | 주 / Multipoint / P2P 비교 및 호환성 |
| 03 | [Lab - Full-Mesh](./03_Lab_Full-Mesh.md) | 주 Interface 기반 모든 VC 직접 연결 |
| 04 | [Lab - Multipoint](./04_Lab_Multipoint.md) | Sub-Interface Multipoint (NBMA) |
| 05 | [Lab - Point-to-Point](./05_Lab_Point-to-Point.md) | Sub-Interface P2P, 구간별 별도 네트워크 |
| 06 | [Lab - 종합 실습](./06_Lab_종합실습.md) | Multipoint + P2P 혼합 (R2~R6) |

### Part 2 — Frame-Relay + Dynamic Routing

| # | 주제 | 설명 |
|---|------|------|
| 07 | [NBMA 라우팅 이슈](./07_NBMA_라우팅이슈.md) | NBMA 특성, broadcast 옵션, Split-Horizon 동작 |
| 08 | [Lab - Hub&Spoke + RIPv2](./08_Lab_HubSpoke_RIP.md) | `broadcast` 옵션, `no ip split-horizon` |
| 09 | [Lab - Hub&Spoke + EIGRP](./09_Lab_HubSpoke_EIGRP.md) | `no ip split-horizon eigrp 100` |
| 10 | [Lab - Hub&Spoke + OSPF](./10_Lab_HubSpoke_OSPF.md) | OSPF Network Type, `neighbor` 명령 |

---

## 💡 핵심 개념

- **Frame-Relay**: FR-Switch 기반 WAN L2 Switching Protocol
- **DLCI**: 가상 회선 식별 번호 (로컬 의미)
- **NBMA (Non-Broadcast Multi-Access)**: 브로드캐스트/멀티캐스트 미지원 → 동적 라우팅 문제 발생
- **broadcast 옵션**: `frame-relay map`에 추가 시 멀티캐스트 트래픽 DLCI로 전달 가능
- **Split-Horizon**: Distance Vector 라우팅에서 동일 인터페이스로 정보 재송신 차단
  - 주 Interface : Split-Horizon **OFF**
  - Sub-Interface : Split-Horizon **ON** (해제 필요)

---

## 🔄 OSPF Network Type 비교

| Network Type | Hello/Dead | DR/BDR | 통신 환경 |
|--------------|-----------|--------|-----------|
| **Broadcast (BMA)** | 10 / 40 | 선출함 | Ethernet, FastEthernet |
| **Non-Broadcast (NBMA)** | 30 / 120 | 선출함 | F/R Multipoint, NBMA 구간 |
| **Point-to-Point** | 10 / 40 | 선출 안 함 | PPP, HDLC, F/R P2P |
| **Point-to-Multipoint** | 30 / 120 | 선출 안 함 | OSPF 전용 환경 |

---

## 📢 멀티캐스트 주소 정리

| Protocol | Multicast Address |
|----------|-------------------|
| RIPv2 | `224.0.0.9` |
| EIGRP | `224.0.0.10` |
| OSPF | `224.0.0.5`, `224.0.0.6` |

> NBMA 환경에서는 위 멀티캐스트가 자동 전달되지 않으므로 `broadcast` 옵션 필수

---

## 🛠 실습 환경

|  |  |
| --- | --- |
| **GNS3** | 가상 네트워크 시뮬레이션 |
| **Dynamips** | Cisco IOS 에뮬레이션 |
| **Cisco IOS (c7200)** | Router / FR-Switch |
| **MobaXterm** | 콘솔 접속 |
| **Git / GitHub** | 버전 관리 및 문서화 |

---

## License

[MIT License](./LICENSE)
