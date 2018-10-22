---
title:  "c# 기본서 - 다형성"
date:   2018-10-15 21:10:00
tags: [c#, 객체지향, 기본, delegate, class, interface]
categories: study-log
---

### 중첩클래스
- 멤버와 같이 취급되므로 접근제한자 생략되면 private가 된다
- cf. 클래스는 원래 접근제한자 없으면 internal

### 추상클래스
- 구현해야하니까 private안됨
- new로 인스턴스화 할 수 없다. 구현도 안한걸 어떻게...?
- 상속받으면 반드시 완성되어야한다
- 전부 껍데기만있는 인터페이스와 달리 내부에 일반 메서드도 정의 가능하다. 단 추상메서드(가상메서드)는 안됨.


### Delegate 
- 델리게이트는 메서드를 인자로 받는 '타입'이다
- 즉 아래와같이 기본 형태에서 식별자의 이름을 바꾸고 예약어를 추가한 상태이다
{% highlight c# %}
int clean (object arg); //반환값 식별자 인자
delegate int FuncDelegate(object arg); // 예약어 반환값 식별자 인자

//어떤 클래스의 함수 Clean을 delegate로 사용하면 다음과 같다.
FuncDelegate cleanFunc = new FuncDelegate(클래스인스턴스.Clean);
//위와 완벽하게 동일한 역할을 하는 구문
FuncDelegate otherCleanFunc = 클래스인스턴스.Clean

//위 구문을 main함수에서 실행시키면
cleanFunc(arg)
otherCleanFunc(arg)
{% endhighlight %}
- 메서드를 인자로 받는 콜백메서드도 델리게이트 사용의 한 방법으로 이해할 수 있다


### Interface 
- 기본적으로 계약이다. 해당 인터페이스를 상속할 경우 인터페이스 내의 메서드를 재정의할것이라는 약속. 이것도 반드시 완성해야함.
- 하지만 메서드의 묶음 개념으로, 프로퍼티도 포함할 수 있다. 그리고 추상과 달리 한 클래스가 여러개의 인터페이스를 다중상속할 수 있다.
- is 연산자를 통해 상속 여부(인터페이스를 사용했을 경우에는 계약이행여부)를 확인할 수 있다.