## 1. Java의 GC 발전 과정 개요

---

Java에서 GC(Garbage Collector)는 메모리 관리를 자동화하여 개발자가 메모리 할당 및 해제를 직접 처리하지 않아도 되도록 합니다. 초기에는 **Serial GC**와 **Parallel GC**가 사용되었지만, 애플리케이션의 규모와 성능 요구사항이 증가함에 따라 **CMS (Concurrent Mark Sweep)**, **G1 GC (Garbage First)**, **ZGC (Z Garbage Collector)** 등이 도입되었습니다.

각 GC는 Java의 메모리 관리와 **Pause Time**을 줄이고 **성능**을 향상시키기 위해 설계되었습니다.

## **2. CMS (Concurrent Mark Sweep)**

---

### **2.1 동작방식**

---

- **Initial Mark:** 살아있는 객체를 찾기 위한 초기 마킹 (STW 발생)
    - **Old Generation의 사용량**이 임계치(Threshold)를 초과했을 때.
    - 빠르게 **GC Root에서 접근 가능한 객체**를 마킹
- **Concurrent Mark:** 애플리케이션이 계속 실행되는 동안 **객체 그래프를 탐색**하면서 **생존 객체를 추가로 마킹**합니다.
- **Remark:** 누락된 객체를 마킹 (STW 발생)
- **Sweep:** 사용하지 않는(마킹되지 않는) 객체 제거 (동시 수행)

위 4단계를 거치며

```sql
STW(Stop-The-World), Pause Time:
JVM이 GC를 실행하기 위해 애플리케이션의 모든 쓰레드를 멈추는 현상
```

```sql
초기 상태:
[객체1][객체2][객체3][객체4][객체5]

객체2,4 삭제 후:
[객체1][빈공간][객체3][빈공간][객체5]
```

### 2.2 CMS의 문제점

---

```
메모리 상태:
[객체1][빈2MB][객체3][빈3MB][객체5]

5MB 객체 할당 시도:
- 총 빈 공간: 5MB (2MB + 3MB)
- 하지만 연속된 공간이 없어 할당 실패

```

- **메모리 단편화(Fragmentation):** Mark-Sweep 방식으로 인해 객체 제거 후 메모리가 조각화되어 대형 객체 할당이 어려움.
- **긴 STW (Stop-The-World):** Full GC 시 애플리케이션 중단 시간이 길어짐.
- **Concurrent Mode Failure:** CMS가 Old Generation의 메모리 부족을 감지했을 때 Full GC를 강제로 실행, 장시간 STW 발생.
- Java 9부터 G1 GC가 기본 GC로 채택되었고, CMS는 Java 14에서 완전히 제거되었습니다.

## 3. G1 GC (Garbage First)

---

### 3.1 G1 GC의 동작 방식

---

G1 GC는 **Region 기반 메모리 관리**를 도입했습니다. 메모리를 고정 크기의 Region으로 나누고, **가비지가 많은 영역**을 우선적으로 수집하는 방식입니다. 

![image](https://github.com/user-attachments/assets/5561f57c-00b4-4814-999f-7bee3ae475e6)

- 빨강색 : Young 영역(Eden)
- 빨강색 + S : Young 영역 (Survivor)
- 파랑색 : Old 영역
- 파랑색 + H : Old 영역 (여러 개의 region이 필요한 사이즈가 큰 객체(humongous object)
- 회색 : 비어있는 부분들. 즉, 언제든지 할당가능한 영역

### 3.2 CMS의 문제 개선

---

- **메모리 단편화 해결:** Region을 사용하여 단편화 최소화
- **예측 가능한 Pause Time:** `Pause Time Goal` 설정 가능
- **Mixed GC:** Young + Old Generation을 동시에 수집하여 효율성 개선

### 3.3 G1 GC의 문제점 (Java 11 이전)

---

- **힙 크기가 커질 경우 STW 증가**
- **메모리 과다 사용 및 GC 지연 발생**
    - **힙 크기가 커질수록 Region의 수가 증가** → **더 많은 Region을 스캔**해야 함.
- **Full GC 가능성 존재**
    - 이게 왜 발생하는지?

### 3.4 왜 ZGC가 필요하게 되었는가?

---

- G1 GC도 대규모 힙 (100GB 이상)에서는 한계를 보임.
- **Pause Time**을 최소화하고 **대규모 메모리 환경**을 지원하기 위해 **ZGC**가 도입

## 4. ZGC (Z Garbage Collector)

---

### 4.1 ZGC의 동작 방식

---

ZGC는 **초대형 힙 메모리**를 지원하며, `컬러 포인터(Colored Pointer)`와 `로드 배리어(Load Barrier)`를 활용하여 **Pause Time**을 1ms 미만으로 유지합니다.

- **Mark:** 살아있는 객체 마킹 (Concurrent)
- **Relocate:** 객체 재배치 (Concurrent)
- **Remap:** 객체 참조 업데이트 (Concurrent)

### 4.2 G1 GC의 문제 개선

---

- **Pause Time과 힙 크기 독립성:** 힙 크기가 커져도 Pause Time이 1ms 미만 유지
- **메모리 단편화 감소:** 컬러 포인터와 Region 재배치
- **초대형 힙 지원:** TB 단위 메모리 지원

### 4.3 ZGC의 한계 (Java 11 기준)

---

- **처리량 제한:** Throughput이 G1 GC보다 낮음
    - 대부분의 GC 작업을 **동시에 수행** (Mark, Relocate, Remap 모두 **동시 진행**).
    - CPU 리소스를 사용해 **STW를 최소화**하지만, 처리량은 G1보다 낮아질 수 있음.
- **CPU 사용률 증가:** 동시성 작업 증가로 CPU 사용량 많음

### 4.4 G1 GC와 ZGC의 공통 개선 목표

---

- **Pause Time 최소화**
- **메모리 단편화 해소**
- **대규모 데이터 지원 및 성능 개선**

## 5. Java 11 → Java 17: G1 GC와 ZGC 성능 향상

---

### 5.1 G1 GC 성능 개선

---

- **NUMA(Non-Uniform Memory Access) 지원 강화:** CPU와 가까운 메모리 우선 접근
- **STW 시간 20% 감소:** Young GC 병렬화 최적화
- **Elastic Heap:** 미사용 메모리 OS에 반환
- **Garbage Region 최적화:** 더 적은 Region 스캔
- **Pause Time:** 30% 감소
- **처리량:** 15-20% 향상
- **메모리 사용:** 20% 향상

### 5.2 ZGC 성능 개선

---

- **Pause Time 1ms 유지:** 힙 크기와 관계없이 일정
- **힙 크기 무제한 지원:** TB 단위 메모리 지원
- **메모리 파편화 감소:** 컬러 포인터 최적화
- **Pause Time:** 70% 감소
- **처리량:** 최대 3배 개선
- **메모리 사용:** 25% 향상

## 6. GC 비교

---

### 6.1 CMS, G1 GC, ZGC 비교표

---

| GC 방식 | Pause Time | 메모리 단편화 | 처리량 | 사용 사례 |
| --- | --- | --- | --- | --- |
| CMS | 높음 | 있음 | 낮음 | 소규모 애플리케이션 |
| G1 GC | 중간 | 적음 | 높음 | 일반적인 웹 애플리케이션 |
| ZGC | 매우 낮음 | 거의 없음 | 중간 | 대규모 데이터, 금융 시스템 |

### 6.2 **적합한 GC 선택 가이드**

---

- **G1 GC:** 소규모, 중간 규모의 웹 애플리케이션, 메모리 제한된 환경
- **ZGC:** 실시간 처리 & 대용량 데이터
