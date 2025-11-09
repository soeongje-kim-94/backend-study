# 가비지 컬렉션 (Garbage Collection, GC)

&nbsp;

GC 튜닝의 핵심 중 하나는 'STW(Stop-the-word) 시간이 얼마나 걸리는가'이다. JVM은 GC를 담당하는 스레드를 제외한 모든 스레드를 일시 중단시키기 때문에 이 시간을 절약할수록 성능상 이점(Latency 개선)을 가질 수 있다.

&nbsp;

## ✅  Garbage First GC (G1 CG)

Java GC 방식 중 하나인 **G1 GC**의 가장 큰 장점은 성능이다.
병렬 처리를 기반으로 다른 어떤 방식보다 속도가 빠르다.
따라서 다수의 CPU와 큰 메모리를 필요로 하는 프로그램 또는 운용 서버에 적합한 방식이다.
즉, 처리량과 응답 시간에서 높은 성능을 목표로 만들어진 GC 방식이며, Java 7 이후로 도입되었고 Java 9부터 기본 CG로 채택되었다.

G1 GC의 처음 설계 목표는 아래와 같다.

&nbsp;

- **기존 CMS 방식과 같이 다른 스레드와 함께 Concurrently하게 동작해야 한다.**
- **스레드 정지를 유발하는 긴 GC 없이, 여유 공간을 Compaction해야 한다.**
- **스레드 정지 시간은 조금 더 예측 가능해야 한다.**
- **처리량(Throughput)에 대한 성능을 희생하지 않는다.**

&nbsp;

또한 G1 GC는 기존 CMS 방식을 대체하기 위해 설계되었기 때문에 CMS 방식의 단점을 보완한다.

&nbsp;​

- **Compaction 과정을 포함하는 알고리즘 적용**
- **GC 단순화 및 잠재적인 메모리 단편화(Fragmentation) 이슈 제거**
- **GC로 인한 스레드 정지 시간 예측 및 GC 수행 시간 최소화**

​&nbsp;

### ➡️ G1 방식만의 힙 영역

이전까지의 GC 방식들은 모두 힙 영역을 크게 Young/Old 영역으로 나누어 사용하였다.
하지만 G1 GC는 다른 관점으로 힙 메모리의 객체들을 관리한다.

​&nbsp;
​&nbsp;
​
![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Garbage%20Collection/assets/java_gc_g1_allocation.png?raw=true)

​&nbsp;
​&nbsp;​

G1 방식은 메모리를 페이징(Paging) 하듯이 힙 영역을 논리적인 단위인 'Region'으로 나눠서 관리한다.
따라서 전체 힙 메모리는 분할된 영역(Region 단위)의 집합으로 구성된다.

Region에 대한 크기는 처음 JVM 프로세스를 시작할 때 힙 메모리의 크기에 따라 결정되며, 일반적으로 1 ~ 32MB 크기인 약 2000개의 Region들로 구성되도록 목표한다. 이후, 각 Region은 특성에 따라 Eden, Survivor, Old 영역 중 하나로 결정된다.

결과적으로 GC 이후 살아남은 객체들은 상태에 따라 특정 Region에 복사 및 이동되며, 각 Region은 이전처럼 자신의 역할을 수행하면서 고정 크기가 아닌 유연하게 메모리를 할당한다.

​이처럼 G1 방식은 힙 메모리를 논리적인 단위인 Region으로 관리함으로써 한 영역에 대한 연속된 공간을 할당할 필요가 없으며, 분리되어 있는 이들의 크기를 동적으로 할당할 수 있다.
즉, 필요에 따라 각 영역의 크기를 편하게 재조정(Resize)할 수 있고, 연속된 공간에 할당하는 것이 아니기 때문에 유연한 메모리 관리가 가능하다.

​&nbsp;

### ➡️ Garbage First ?

​&nbsp;

1. **전역으로 Concurrent한 Marking을 수행한다.**
2. **어떤 Region에서 많은 쓰레기 객체를 수집할 수 있는지 찾는다.**
3. **가장 많은 여유 공간을 만들어 낼 수 있는 지역을 먼저 청소(Clean-up)하고, Compaction한다.**
​
​&nbsp;

'Garbage First'라고 명명된 이유는 힙 메모리에 가장 많은 여유 공간(Unreachable 객체가 많은)을 만들어 낼 수 있는 Region을 우선적으로 수집하고 청소하기 때문이다.

G1 방식은 이러한 과정에서 STW 시간을 최소화하고자 미리 목표한 STW 시간을 기반으로 수집할 지역의 개수를 결정한다.
이는 이전 데이터를 기반으로 예측하기 때문에 대부분의 경우를 충족시킬 수 있다.
또한 CMS 방식의 단점이었던 메모리 Fragmentation 문제를 해결하기 위한 Compaction을 수행할 수 있다.

​&nbsp;

### ➡️ 정리

G1 GC는 애플리케이션 내 다른 스레드와 동시에 진행되는 Marking을 바탕으로 목표한 STW 시간에 맞춰 우선적으로 많은 여유 공간을 만들어 낼 수 있는 Region들을 수집한다.

그다음, GC 수행 결과로 해당 Region에서 살아남은 객체들을 다른 Region으로 이동시킨 후 남은 Unreachable 객체들을 Clean-up하고, Compaction을 수행한다.

위 과정은 처리량 향상 및 시간 절약을 위해 병렬적(멀티 스레드)으로 진행된다.

​&nbsp;

## ✅ G1 GC의 동작 과정

### ➡️ Young GC

​&nbsp;
​​&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Garbage%20Collection/assets/java_gc_g1_young.png?raw=true)

​&nbsp;
​&nbsp;

우선, Young GC가 발생하면 Eden 또는 Survivor 영역에서 살아남은 객체들은 또다른 Survivor 영역으로 이동하며, 이 결과로 남은 Unreachable 객체들은 제거된다.
물론, GC의 대상이 된 Region들은 목표한 STW 시간을 충족하기 위해 미리 수집된 지역들이다.
STW가 발생하긴 하지만 일련의 과정들(Compaction 포함)이 멀티 스레드로 처리되기 때문에 수행 시간이 최소화된다.

이때, GC는 남은 Eden과 Survivor 영역의 크기를 계산하여 따로 저장한다.
이는 다음 GC 발생에서 예상되는 STW 시간을 설정하고, 수집할 각 지역의 크기를 조정하는데 사용된다.

​​&nbsp;
​&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Garbage%20Collection/assets/java_gc_g1_young_end.png?raw=true)

​&nbsp;
​&nbsp;

Young GC 수행 결과로 살아남은 객체들은 또 다른 Survivor 영역 또는 Old 영역으로 이동하며, Compaction 상태가 된다.
단, Old 영역으로 이동하는 객체들은 기준 Age를 초과한 객체들이다.

​&nbsp;
​
### ➡️ Old GC

G1 방식의 Old GC 또한 병렬 처리와 함께 STW 시간을 최소화하는 것을 목표로 한다.

​&nbsp;

#### 1️⃣ Initial Mark
​
​&nbsp;
​&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Garbage%20Collection/assets/java_gc_g1_init_mark.png?raw=true)

​&nbsp;
​&nbsp;

최초 Intitial Mark 단계에서는 STW가 발생한다.
보통 Young GC의 Mark과 함께 수행되는데, Old 지역의 객체를 참조하고 있는 다른 객체를 가진 Survivor 지역들을 식별한다.

​&nbsp;

#### 2️⃣ Root Region Scan

Root Region Scan 단계는 애플리케이션의 다른 스레드와 함께 진행된다.
앞서 Intitial Mark 단계에서 식별한 Survivor 지역을 스캔하여 Old 지역의 객체를 참조하고 있는 Young 영역의 객체를 식별한다.
이 단계는 다음의 Young GC가 발생하기 전까지 끝나야 한다.

​&nbsp;

#### 3️⃣ Concurrent Mark
​
​&nbsp;
​&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Garbage%20Collection/assets/java_gc_g1_concurrent_mark.png?raw=true)

​&nbsp;
​&nbsp;

Concurrent Mark 단계에서는 힙 영역 전체에 걸쳐, 참조되고 있는 객체(살아있는 객체)를 찾는다.
이는 앞 단계와 마찬가지로 애플케이션 내 다른 스레드와 동시에 진행되며, 목표한 STW 시간에 여유 공간을 가장 많이 만들 수 있는 Region에 대한 계산 작업까지 포함한다. (단, 이 과정은 Young GC에 의해 Interrupt될 수도 있음)

이때, Concurrent Mark 수행 중 완전히 비어있는 Region을 찾는다면 해당 Region을 Unused 상태로 만든다.

​&nbsp;

#### 4️⃣ Remark

Remark 단계에서는 다시 STW가 발생한다.
이전까지 수행했던 힙 영역의 살아있는 객체에 대한 식별 작업을 마저 완료한다.
즉, 앞서 Concurrent Mark 단계에 대한 검증 단계이다.

이 단계에서 'SATB(Snapshot-At-The-Beginning)'이라는 알고리즘이 사용되는데, 이는 CMS 방식의 Remark 단계에서 사용되는 알고리즘과 비교하여 더 빠르다는 장점을 가진다.

Remark 단계가 끝나면 Unused 상태인 Region들을 모두 제거하고, 메모리를 다시 확보함으로써 힙 영역 내 모든 Region 중 살아남은 Region에 대한 계산이 완료된다.

​​&nbsp;

#### 5️⃣Copy & Clean-up

​​&nbsp;
​​&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Garbage%20Collection/assets/java_gc_g1_cleanup.png?raw=true)

​​&nbsp;
​​&nbsp;

마지막으로 Copy & Clean-up 단계에서는 앞서 일련의 과정을 통해 식별한 살아남은 객체들을 비어있는 영역으로 이동시키고, 남겨진 Unreachable 객체들을 제거한다.

그 결과, 비워진 Region들은 Unused 상태가 된다.
또한 복사/이동 및 Compaction 작업에 대한 STW가 발생한다.

단, 살아있는 객체가 아주 적은(Liveness가 아주 낮은) Old Region에 대해서는 '[GC pause (mixed)]'라는 별도의 로그만 표시해놓고, 다음 Young GC 수행 시 함께 수집(Compaction 포함)한다.

​​​&nbsp;
​​&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Garbage%20Collection/assets/java_gc_g1_cleanup_after.png?raw=true)

​​&nbsp;
​​&nbsp;

## ✅ 마무리

G1 GC의 주요 목표는 제한된 GC 응답 시간과 큰 힙 메모리 공간을 필요로 하는 애플리케이션에서 속도 및 처리율을 향상시키고자 하는 것이다.
예를 들어, 6GB 이상의 힙 메모리 크기 및 STW 시간에 대해 0.5초 이내로 예측 가능한 수준을 필요로 하는 프로그램들이다.

만일 CMS GC 또는 Paralled Old GC를 사용하는 애플리케이션이 아래와 같은 경우들에 해당한다면, 이를 G1 GC로 교체함으로써 성능 향상을 기대할 수 있을 것이다.

> CMS GC는 Java 9 이후 deprecated, Java 14에서 제거됨

​​&nbsp;

- **Old GC 시간이 너무 길거나 자주 발생하는 경우**
- **객체 할당 비율과 비교하여 객체들이 살아남아 Old 영역으로 이동하는 비율에 큰 차이를 보이는 경우**
- **GC 및 Compaction 시간에서 성능상 손해가 발생하는 경우 (약 0.5 ~ 1초보다 더 길어질 때)**

​​&nbsp;

결과적으로 GC는 Java 기반의 서비스 애플리케이션에서 성능과 연관된 중요 이슈이기 때문에 이를 정확히 이해하고 사용하는 것이 중요하다. 더 나아가, GC 튜닝을 고려해 본다면 애플리케이션의 전체적인 성능 향상을 위한 좋은 시도가 될 것이다.

​&nbsp;

## 🔍 내용 참조
 - https://d2.naver.com/helloworld/37111
