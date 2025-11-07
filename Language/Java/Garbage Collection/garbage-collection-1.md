
# 가비지 컬렉션 (Garbage Collection, GC)

&nbsp;

가비지 컬렉터(GC)는 Java의 중요한 메모리 관리 도구로서 JVM 힙 영역의 쓰레기 객체를 청소한다.

JVM의 힙 영역에 객체들이 할당되면 각 객체는 메모리를 점유하고 있게 된다.
하지만 해당 객체들이 더 이상 필요없음에도 메모리를 여전히 점유하고 있다면, 이는 자원 손실 및 성능 저하 요소가 된다.

이처럼 메모리 상의 필요없는 객체를 쓰레기로 분류하고, 이들을 처리하는 메모리 관리자가 바로 GC이다.
일반적으로 Java에서는 개발자가 프로그램 코드에 따로 객체에 대한 메모리 해제을 명시적으로 작성하지 않기 때문에 이 GC의 역할은 매우 중요하다.

GC는 이른바, Mark and Sweep 과정을 통해 메모리 공간을 확보한다.
이는 스택 영역에서 도달할 수 없는 힙 영역의 객체(Unreachable Object)를 탐색하고, 이들을 우선적으로 메모리에서 제거하는 과정이다.
GC가 실행되면 스택 영역의 모든 변수들을 스캔하여 각각 어떤 객체를 참조하고 있는지 Marking하고, 이후 Marking되어 있지 않은 쓰레기 객체들을 Sweep한다.

&nbsp;
&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Garbage%20Collection/assets/java_gc_marking.png?raw=true)

&nbsp;
&nbsp;

일반적으로 GC의 대상이 되는 쓰레기 객체는 아래의 경우에 해당한다.

&nbsp;

- **모든 객체 참조가 null인 경우**
- **객체가 블록 안에서 생성되고, 해당 블록이 종료된 경우**
- **부모 객체가 null이 된 경우 (자식 또는 포함된 객체)**

&nbsp;

JVM은 GC를 실행하기 위해서 애플리케이션(GC를 실행하는 스레드를 제외한 모든 스레드)의 실행을 일시적으로 멈추고, 처리 완료 후 중단되었던 스레드들을 다시 시작한다.

이 단계를 'STW(Stop-the-world)'라고 하며, GC 튜닝 목적의 대부분은 이 STW의 소요 시간을 줄이는 것이다.

&nbsp;

## ✅ 힙 영역의 구조​

GC는 'Weak Generational Hypothesis'라고 불리는 아래의 전제 조건을 바탕으로 탄생했다.

&nbsp;

1. **대부분의 객체는 금방 접근 불가 상태(Unreachable)가 된다.**
2. **오래된(Old) 객체가 젊은(Young) 객체를 참조하는 경우는 매우 적다.**

&nbsp;

가장 일반적인 JVM 모델인 HotSpot VM은 이 조건의 장점을 최대한 적용하기 위해 힙 영역을 크게 2개의 물리적인 공간으로 나누었다.
이렇게 나누어진 공간은 Young 영역과 Old 영역으로 구분된다.

&nbsp;

### ● Young Generation

새롭게 생성한 객체의 대부분이 이곳에 위치한다.
대부분의 객체가 금방 접근 불가능 상태가 된다는 전제하에 수많은 객체가 Young 영역에 생성되었다가 사라진다.

'Minor GC'는 이 Young 영역에서 수행되는 GC를 의미한다.

&nbsp;

### ● Old Generation

계속 사용되어 Young 영역에서 살아남은(접근 불가능 상태가 되지 않은) 객체가 이곳에 복사된다.
대부분 Young 영역보다 크게 할당되며, 크기가 큰 만큼 Young 영역보다 GC가 적게 발생한다.

이 Old 영역의 GC를 'Major GC'라고 한다.

&nbsp;​
&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Garbage%20Collection/assets/java_gc_card_table.png?raw=true)

&nbsp;​
&nbsp;

또한 Old 영역에는 카드 테이블(Card Table)이 존재한다.

Old 영역의 객체가 Young 영역의 객체를 참조하는 경우, 이를 처리하기 위해 해당 테이블에 정보를 표시한다.
Minor GC를 실행하는 경우, Old 영역 내 모든 객체의 참조를 확인하는 대신 카드 테이블만 확인하여 GC의 대상인가를 식별한다.

결과적으로 약간의 오버헤드가 발생하지만 전반적인 GC 시간을 절약할 수 있다.

&nbsp;

> **Full CG**
>
> 힙 영역 전체(Young + Old) 및 일부 메타데이터 영역까지 포함하는 CG를 의미한다.

&nbsp;

## ✅ Young 영역의 GC (Minor GC)

Young 영역은 다시 3개의 영역으로 구분된다.

&nbsp;

- **Eden 영역**
- **Survivor 영역 (0/1, 2개)**

​&nbsp;

우선, 새로 생성된 객체는 대부분 Eden 영역에 위치하며, Survivor 영역은 비워진 상태로 시작된다.
이후 Eden 영역이 꽉 차게 되면 Minor GC가 발생한다.

​&nbsp;
​&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Garbage%20Collection/assets/java_gc_minor.png?raw=true)

​&nbsp;
​&nbsp;

최초에 Minor GC가 발생하게 되면 Eden 영역에서 살아남은 객체들을 Survivor 영역(0/1) 중 한 곳으로 이동된다.
단, 살아남은 객체들을 이동시킬 때는 오직 1개의 Survivor 영역만 사용한다.
이는 Survivor 영역 중 한 곳은 반드시 비어있는 상태로 남아 있어야 함을 의미한다.

그다음, 한 번의 GC 수행 결과로 살아남은 모든 객체가 Survivor 영역으로 이동되었다면 남은 쓰레기 객체들은 Eden 영역이 비워짐과 동시에 메모리에서 제거된다.

이후 Minor GC에서 한 Survivor 영역이 가득 차게 되면 살아남은 모든 객체를 다른 Survivor 영역으로 옮긴 뒤, 자신을 비워진 상태로 만든다.

이처럼 Minor GC가 계속 발생함에 따라 살아남은 객체들은 Survivor 영역 사이를 번갈아 이동하면서 한곳에 쌓이게 된다.
이때, 살아남은 객체들은 Survivor 영역 사이를 이동하면서 Age 값이 증가한다.

따라서 위 Minor GC 과정들이 반복되면서 살아남은 객체들은 계속 Age 값이 증가하게 되며, 기준 Age를 초과한 객체들이 Old 영역으로 옮겨진다.

​&nbsp;​
&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Garbage%20Collection/assets/java_gc_heap.png?raw=true)

&nbsp;​
&nbsp;

이 과정을 'Promotion'이라고 한다.

즉, Minor GC가 반복되면 Promotion도 계속 발생한다.

​&nbsp;

## ✅ Old 영역의 GC (Major GC)

이제 Minor GC와 Promotion 작업이 반복되면서 Old 영역이 가득 차게 되면 Major GC가 발생한다.
Major GC는 다양한 GC 수행 방식에 따라 절차가 달라지며, 이 과정은 Minor GC와 비교하여 수행 시간(STW)이 매우 길다.

Major GC의 방식은 Java 8 이전을 기준으로 총 5가지가 있다.

​&nbsp;

### ● Serial GC

Serial GC는 CPU 코어가 1개일 경우 사용하기 위해 만든 방식이다.
따라서 Serial GC를 운영 서버에 적용하게 되면 심각한 성능 저하 유발하기 때문에 절대 사용하면 안되는 방식이다.

기본적으로 이 방식은 Mark-Sweep-Compact 알고리즘을 사용한다.

먼저, 첫 단계로 Old 영역에 살아 있는 객체를 식별(Mark)한다.
그다음, 힙 영역의 앞 부분부터 확인하면서 살아남은 객체만 남긴다(Sweep).
마지막으로 Sweep 단계 이후 비워진 힙 공간에 살아남은 객체들이 연속되게 쌓이도록 힙 영역의 가장 앞부분부터 채운다(Compact).

​&nbsp;

### ● Parallel GC

Parallel GC는 Serial GC와 같은 알고리즘을 사용하지만 GC를 처리하는 스레드가 여러 개인 방식이다.
따라서 Serial GC보다 빠르게 객체들을 처리할 수 있다.

​&nbsp;

### ● Parallel Old GC (Parallel Compacting GC)

Parallel Old GC는 Parallel GC와 비교하여 알고리즘만 다른 방식이다.
기존의 Mark-Sweep-Compact 알고리즘과 다르게 Mark-Summary-Compaction 과정을 수행한다.

Summary 단계는 GC를 수행한 영역에 대해서 별도로 살아 있는 객체를 식별한다는 점에서 기존 Sweep 단계와 차이가 있고, 더 복잡한 과정이 수행된다.

​&nbsp;

### ● Concurrent Mark & Sweep GC (CMS CG, Java 9 이후 Deprecated)

CMS GC는 애플리케이션의 응답 속도가 중요할 때 사용하는 방식으로 'Low Latency GC'라고도 한다.

​&nbsp;
​&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Garbage%20Collection/assets/java_gc_serical_and_cms.png?raw=true)

​&nbsp;
​&nbsp;

1. 초기 Initial Mark 단계에서는 클래스 로더에서 가장 가까운 객체 중 살아 있는 객체만 찾는 것으로 종료된다. 따라서 멈추는 시간은 매우 짧다.
2. Concurrent Mark 단계에서는 직전에 살아있다고 확인한 객체에서 참조하고 있는 객체들을 따라가면서 확인한다.이 단계는 다른 스레드가 실행 중인 상태에서 동시에 진행된다.
3. Remark 단계에서는 Concurrent Mark 단계에서 새롭게 추가되거나 참조가 끊긴 객체를 확인한다.
4. 마지막으로 Concurrent Sweep 단계에서 쓰레기 객체를 제거하는 작업을 수행한다. 물론 이 작업도 다른 스레드가 실행되고 있는 상황에서 동시에 진행된다.

​&nbsp;

위와 같은 과정으로 진행되는 GC 방식으로 STW 시간이 매우 짧고, 필요 시 Full GC에서 Compaction을 수행한다는 특징을 갖는다.

하지만 다른 GC 방식보다 메모리와 CPU를 더 많이 사용하고, Compaction을 자주 수행하지 않는다는 점에서 단편화(Fragmentation)가 발생할 수 있는 단점이 있다.

따라서 CMS GC는 신중하게 선택해야 한다. 예를 들어, Compaction 작업은 다른 GC 방식의 STW 시간보다 훨씬 긴 시간이 소요되기 때문에 이 Compaction 작업이 얼마나 자주, 오랫동안 수행되는지 확인할 필요가 있다.

​&nbsp;​

### ● Garbage First GC (G1 CG, Java 9 이후 기본 적용)

G1 GC는 CMS GC의 대안으로 고안되었으며, 성능이 매우 뛰어나다는 장점이 있다.

​&nbsp;
​&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Garbage%20Collection/assets/java_gc_g1.png?raw=true)

​&nbsp;
​&nbsp;

메모리를 바둑판처럼 각각의 영역(Region 단위)으로 구분하고, 각 Region에 Young/Old 영역 역할을 동적으로 할당하여 GC를 실행하는 방식으로 기존의 물리적으로 분리되어 Young/Old 영역에서 진행되었던 CG 처리를 한 영역 안에서 모두 진행한다.


​&nbsp;

## 🔍 내용 참조 및 출처
 - https://d2.naver.com/helloworld/1329
