# JVM (Java Virtual Machine)

&nbsp;

일반적인 프로그램 또는 프로세스는 OS의 메모리 상에서 동작한다.
하지만 Java 프로그램은 OS에 독립적으로 실행되는데, 그 이유는 'JVM'이라는 특별한 가상 머신이 OS 위에서 하나의 프로세스로 실행되며, OS가 할당한 메모리 공간을 내부적으로 관리하면서 Java 프로그램을 실행한다.

결과적으로 이는 Java 프로그램이 OS에 종속적이지 않고, 어떤 디바이스 또는 환경에서도 JVM 위에서 프로그램을 실행할 수 있다는 큰 장점이 된다.
하지만 OS으로부터 직접 제어 받는 방식보다 속도 면에서 느리다는 단점이 있다.

&nbsp;
&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Memory%20Structure/assets/jvm.jpg?raw=true)

&nbsp;
&nbsp;

## JVM의 구조 및 컴파일 과정

컴파일 관점에서 JVM은 크게 클래스 로더(Class Loader), 실행 엔진(Execution Engine), 런타임 데이터 영역(Runtime Data Area)으로 구성된다.

&nbsp;

### ● 클래스 로더 (Class Loader)

우선 기본적으로 Java를 지원하는 IDE에서 작성된 Java 코드 파일은 .java 확장자로 저장된다.
그 후 Java 파일의 빌드가 이루어지면 Java 컴파일러는 javac 명령어를 통해 .class 파일을 생성한다.

&nbsp;
&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Memory%20Structure/assets/jvm_compiler.jpg?raw=true)

&nbsp;
&nbsp;

.class 파일은 바이트코드(Byte-code, 반기계어)이기 때문에 OS에서 바로 실행될 수 없다.
그래서 JVM은 바이트코드를 CPU가 실행할 수 있는 네이티브(Native) 코드로 변환하거나 해석하여 실행한다.
이제 런타임 시점이 되면 클래스 로더에 의해 JVM 내부에 로드된다.

&nbsp;

### ● 실행 엔진 (Execution Engine)

이후 JVM 내에 로드된 바이트 코드는 실행 엔진에 의해 기계어로 해석되어 메모리 상(Runtime Data Area)에 배치된다.
실행 엔진은 인터프리터와 JIT(Just In Time) 컴파일러로 구성된다.
앞서 코드가 JVM을 통해 해석되기 때문에 OS으로부터 직접 제어 받는 방식보다 속도가 느리다는 단점을 보완할 수 있는 JIT 컴파일러가 존재한다.

&nbsp;
&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Memory%20Structure/assets/jvm_execution_engine.png?raw=true)

&nbsp;
&nbsp;

인터프리터는 바이트 코드를 한 줄씩 실행하기 때문에 속도가 느리다.
반면에 JIT 컴파일러는 바이트 코드를 어셈블러와 같은 네이티브 코드로 컴파일하여 직접 실행한다.

프로그램 실행 중 빈번하게 호출되는 메서드(Hotspot)를 감지하여, 해당 부분을 네이티브 코드로 컴파일하는 방식으로 JIT 컴파일러에 의해 해석된 코드는 캐시에 보관되기 때문에 한번 컴파일 된 후에는 빠르게 실행되지만 미리 한번에 코드 전체를 컴파일하는 작업으로 인해 인터프리터 방식보다 훨씬 오래 걸리게 된다.

이러한 이유로 실행 엔진은 인터프리터 방식으로 실행하다가 적절한 시점에 JIT 컴파일러 방식을 선택한다.
예를 들어, 한번만 실행되는 코드는 인터프리터 방식을 사용하는 것이 유리하다.

&nbsp;
&nbsp;

## 런타임 데이터 영역 (Runtime Data Area)

&nbsp;
&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Memory%20Structure/assets/jvm_rutime_data_area.png?raw=true)

&nbsp;
&nbsp;

### ● 메서드 영역 (Method Area)

JVM 시작시 생성되고 프로그램 종료 시까지 모든 스레드가 공유하는 영역이다.
Metaspace(HotSpot VM 기준, Java 8 이후 PermGen이 Metaspace로 대체됨)라고 표현하며, JVM 구동 중 사용될 클래스 파일들을 로드할때 클래스 메타데이터를 포함해 런타임 상수 풀, 정적(static) 변수 등을 저장한다.

&nbsp;

> **런타임 상수 풀 (Runtime Constant Pool)**
>
> 클래스의 심볼(Symbol), 상수(Constant), 레퍼런스(Reference) 정보 등을 저장하는 역할이며, 이들의 물리적인 메모리 위치를 참조할 경우에 사용된다.
> 단, 문자열 상수는 Metaspace가 아닌 힙 영역 내 문자열 리터럴 풀(Intern Pool)에 저장된다.

&nbsp;

### ● 스택 영역 (Stack Area)

각 스레드마다 하나씩 존재하며 스레드가 시작될 때 할당되는 영역이다.
만일 추가적인 스레드 생성이 없다면 메인 스레드에 대한 한 개의 스택만 할당된다.
클래스 내의 메서드에서 사용되는 정보들이 저장되는 공간으로서 메서드 호출 시에 해당 메서드에 대한 프레임(Frame)을 추가하고, 메서드 종료와 함께 제거한다.
각 프레임은 LIFO(Last In First Out) 방식으로 실행 순서에 따라 생성/소멸되며 메서드의 매개 변수, 지역 변수, 리턴 값 등의 정보가 저장된다.

메서드 호출 시 생성된 프레임 내부의 지역 변수 공간에는 각 블록마다 기본(Primitive) 타입 변수 또는 참조(Reference) 타입 변수가 추가된다.
이 블록이 생성되는 시점은 지역 변수가 선언되었을 때이다.
기본 타입 변수는 스택 영역에 실제 값을 가지지만 참조 타입 변수는 실제 값이 아닌 힙 영역이나 메서드 영역에 대한 주솟값를 가진다.

&nbsp;

### ● 힙 영역(Heap Area)

런타임시 동적으로 할당하여 사용하는 영역으로 주로 긴 생명 주기(Life Cycle)를 가지는 데이터들을 저장한다.
힙 영역에는 클래스들에 대한 객체 또는 배열 등의 참조 타입 변수 정보가 저장된다.

힙 영역에 생성된 객체는 JVM 스택 영역의 변수나 다른 클래스의 필드 등에서 참조된다.
이는 객체의 실제 값이 저장된 힙 영역의 주솟값을 참조함을 뜻한다.
만일 이를 참조하는 다른 변수나 필드가 없다면 쓰레기로 취급되어 GC(Garbage Collector)에 의해 제거된다.

&nbsp;

### ● PC 레지스터 영역 (PC Register Area)

스레드마다 하나씩 생성하고, 현재 실행 중인 JVM 명령의 주솟값이 저장된다.

&nbsp;

### ● 네이티브 메서드 스택 영역 (Native Method Stack Area)

Java 이외 다른 언어의 함수 호출(예 : C/C++ 메서드)를 위해 할당되는 영역이다.

&nbsp;
&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Memory%20Structure/assets/jvm_stack_area.png?raw=true)

&nbsp;
&nbsp;

## 가비지 콜렉터 (Garbage Collector, GC)

GC는 Java의 중요한 메모리 관리 도구이다.

JVM의 힙 영역에는 객체들이 할당되는데, 이는 각 객체가 메모리를 점유하고 있는 것이다.
하지만 해당 객체가 필요하지 않음에도 불구하고 메모리를 점유하고 있다면, 메모리 관리 측면에서 자원 손실로 인한 심각한 성능 저하를 유발할 것이다.
Java에서는 이와 같이 더 이상 필요가 없어진 객체를 쓰레기로 분류한다.

그렇다면 누군가가 이러한 쓰레기 객체들을 처리하여 메모리 손실을 방지해야 한다.
바로 Java의 GC가 JVM 힙 영역에 대한 메모리 관리자로서의 역할을 수행한다.
결과적으로 GC의 주요 업무는 효과적으로 힙 영역의 쓰레기 객체들을 찾아서 처리하는 것이다.

&nbsp;
&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Memory%20Structure/assets/jvm_gc.png?raw=true)

&nbsp;
&nbsp;

GC는 참조되지 않는 객체(Unreachable Object)를 우선적으로 메모리에서 제거함으로써 메모리 공간을 확보한다.
이 과정은 간단하게 'Mark and Sweep'으로 표현한다.

Mark 단계에서 GC Root(스택 프레임, 클래스 로더, JNI 레퍼런스 등)에서부터 도달 가능한 객체들을 탐색하고 표시한다.
이 작업을 위해 모든 스레드는 일시 중단되는데, 그래서 이 작업을 'Stop the world'라고 표현한다.

이후 Sweep 단계에서는 Marking되지 않은 모든 객체들을 힙 영역에서 제거한다.

&nbsp;
&nbsp;

## Java 8 메모리 구조 개선 (Permanent Generation 제거)

&nbsp;
&nbsp;

![](https://github.com/soeongje-kim-94/backend-study/blob/main/Language/Java/Memory%20Structure/assets/jmv_heap.png?raw=true)

&nbsp;
&nbsp;

Java 8 이전 기준으로 힙 영역과 PermGen은 서로 격리되어 있지만, 이들이 사용하는 물리적 메모리는 연속적이다.
또한 종종 JVM에 의해 GC가 메서드 영역까지 확장됨으로써 GC의 대상에 포함되었다.
이는 메모리 관리 측면에서 기존 PermGen이 JVM 자체 프로세스 내에 있었다는 것을 의미한다.

그래서 PermGen은 C-Heap이라고도 하는 JVM에 의해 관리되는 메모리 안에 포함되어 고정 크기로 할당되어 있었다.
하지만 이 PermGen의 크기가 강제된다는 점으로 인해 힙 영역의 상한이 제한되고, 메모리 누수 문제까지 연결될 수 있는 위험성이 존재했다.

Java 8 메모리 구조 개선 사항으로 PermGen은 Metaspace으로 대체되었고, JVM이 아닌 OS 레벨에서 관리하는 네이티브 영역에 포함되도록 변경되었다.
따라서 기존 PermGen의 정보를 OS 레벨에서 관리함으로써 더 이상 고정 크기가 아닌 동적으로 크기를 조절할 수 있게 되었다.
즉, 변경될 가능성이 아주 적은 여러 메타 정보들을 OS가 관리하는 영역으로 옮겨 PermGen의 크기 제한을 없앤 것이다.
결과적으로 이전에 개발자가 PermGen을 확보하기 위해 직접 메모리 튜닝을 고민해야 했던 불편을 해결할 수 있게 되었다.

추가적으로 정적(static) 변수의 값과 리터럴 상수를 힙 영역에 위치시켜, 최대한 CG의 대상이 될 수 있도록 변경되었다.
