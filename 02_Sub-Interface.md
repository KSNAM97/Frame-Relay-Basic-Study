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
