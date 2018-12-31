---
title:  "c# 기본서 - 빌드환경과 GC, 힙과 스택"
date:   2018-11-06 23:10:00
tags: [c#, build, GC, Stack, Heap, Attribute, 예약어, dll, 자원수거]
categories: study-log
---

### 전처리기 지시문 #if #endif
- 잔처리기를 사용하면 컴파일시 적용되는 구문을 분기처리할 수 있다.
- DEBUG와 TRACE상수의 경우 빌드 옵션에서 세팅할수 있고, 별도로 세팅을 하고싶으면 [아래대로 하면](https://stackoverflow.com/questions/507704/will-if-release-work-like-if-debug-does-in-c) 된다
1. Project Properties -> Build
2. Select Release Mode
3. in the Conditional compilation symbols textbox enter: RELEASE
- 이 밖에도 #warning, #error, #line, #region, #endregion, #pragma 등이 있다.

### [Attribute]
- 개발자로하여금 컴파일 후에도 EXE/DLL파일에 남길 수 있는 주석처럼 사용되는 코드
- 라고는 하는데 우리팀은 캐싱이나 인증 로직을 조건처럼 구현할때 사용했다.
- 직접 특성(Attribute)을 만들수도있는데, System.Attribute를 상속받으면 된다
- 특성을 적용받는 타겟을 옵션으로 줄 수도 있다.
- 참조: [Conditional("DEBUG")] 이렇게 입력하면 전처리기지시문처럼 해당 함수를 DEBUG때만 빌드하게 할 수 있다.


### 예약어 checked, unchecked
- 산술 범위가 넘어가면서 중요한 수치가 되는 값들이 증발하는 경우(overflow/underflow)를 막기위해 사용된다
- checked 블록을 만들고 연산을 수행하면 예외를던지고
- checked옵션을 전체 적용하고 unchecked블록을 생성하면 해당 블록 내에서 over/underflow가 일어나도 예외가 나타나지 않는다

### 가변 매개변수 params
- 파라미터가 여러개가 될수도 있을 때 아래와같이 표기하면 다양하게 값을 넣을 수 있다.
{% highlight c# %}
static int Add(params int[] args)
{
    //연산
}
static void DoSomething(params object[] args)
{
    //연산
}
{% endhighlight %}

### 빌드 구성요소: dll, app.config
- dll파일을 위한 함수는 public으로 만든다
- 한 dll에 여러 프로세스가 접근할 수 있다
- 그러나 프로세스별로 동일 dll의 다른 버전을 요구할수도 있다. 이때는 공개키토큰이나 어셈블리 서명방식을 통해 구분할 수 있다.
- app.config는 개발자 코드 실행전 단계인 CLR환경 초기화에사 어떤 값을 전달해야할때 사용되는 변수/설정사항 등을 선언해놓는 곳이다
- app.config를 통해 프레임워크를 설정할 수 있다. 닷넷프레임워크는 4.0이후부터 업데이트시 파일이 대체되므로 구버전을 설치한 경우 이전 버전들이 남아있다.
- MS는 타입과 타입이 정의된 어셈블리를 느낌표로 구분한다
    - ex) mscorlib.dll에 구현된 System.Object의 경우 -> mscorlib!System.Object

### 릴리즈와 디버깅모드
- Release의경우 코드최적화를 하기때문에 StackTrace로 남는 내용이 정확하게 보여지지 않을 수 있다.
- Trace는 Debug와 달리 릴리즈모드에도 실행된다. 즉 추적을 위한 네임스페이스이다.

### 플랫폼 선택 (AnyCPU, x86, x64)
- 일단 AnyCPU를 사용할것을 권장.
- x86은 어느윈도우든 32비트exe로 실행되고
- x64는 64비트 윈도우에서 64비트exe로만 실행된다. 32비트 윈도우에선 실행 되지 않는다.
- AnyCPU인 경우는 각 비트에 맞게 실행된다. 다만 오직 AnyCpu에서만 옵션을 통해 32비트 기본사용을 선택할 수 있다.
- 32비트 기본사용 옵션이 필요한 이유는 32비트exe인 dll이 매우 많기 때문. 가령 ActiveX 
  
### 예외처리와 자원수거
- finally가 필요한 이유는 예외가 발생해도 db나 file은 자원수거는 해야하기떄문
    - 자원수거를 자연스럽게 하는 또다른 방법은 using이 있다.
    - 개발자의 실수를 예방하고 방어적인 차원에서 해제코드를 넣는 것이 소멸자. 해제코드가 언제 실행될지는 모르기때문에 사용은 하지 말자.
- throw는 단독으로 사용하는게 좋다. throw ex를 하면 예외시점부터의 스택이 호출된다.

### 스택
- 힙과 스택 둘 다 데이터를 위한 메모리이다.
- 스택은 스레드 생성이 되면 기본적으로 1mb용량으로 할당된다.
- 스택은 메서드의 실행, 메서드 인자, 지역변수를 처리한다. 떄문에 재귀호출로인한 에러는 스택오버플로우이다.
- 스택 에러발생시 메모리가 모두 소비되었기 때문에 StackTrace를 알려주는 메서드 호출이 불가하다

### 힙
- CLR에서는 기본적으로 관리힙을 가리킨다. GC의 할당, 해지를 관리한다
- new로 할당되는 모든 참조형 객체는 힙에 할당된다. 얘넨 모두 GC가 직접 관리한다.
- 박싱 언박싱은 참조형데이터와 값데이터간의 이동을 의미하는데, 박싱이 일어날경우 값데이터를 참조형으로 변경하게 되므로 GC가 매우 바빠진다. 대표적인 예가 값데이터를 함수의 인자로 넣는 경우.
- 대용량 객체 힙(Large Object Heap)이 있다. 가비지 수집마다 LOH를 이동시키는건 부담이 크므로 GC가 직접 관리하지 않는다.

### GC
- CLR의 세대는 2세대가 끝이다.
