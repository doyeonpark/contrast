---
title:  "c# 기본서 - BCL: Thread, Monitor, Lock, Interlocked"
date:   2018-11-27 00:14:00
tags: [c#, bcl, thread, monitor, lock, interlocked]
categories: study-log
---

### Thread Context 
- 스레드는 CPU의 명령어 실행과 관련된 정보를 보관한다. 이를 스레드 문맥이라고 한다.
- 즉, 언제 실행하고 또 이전에 얼만큼 실행했는지를 기억하는 것이다.
- 초기에는 단일 CPU였기 떄문에 다중스레드는 1개의 CPU에서 조금씩 실행시간을 나누어 실행하는 형태였지만
- 멀티 CPU/Core시대가 오면서 진정한 다중 스레드 실행이 가능해졌다


### 스레드 제대로 이해하기: System.Threading.Thread
{% highlight c# %}
static voic Main(string[] args) {
    Thread t = new Thread(threadFunc);
    t.background = true;
    t.start(); 
}
static void ThreadFunc() {
    Console.WriteLine("60초 후 종료");
    Thread.Sleep(1000*60);
    Console.WriteLine("종료");
}
{% endhighlight %}
- 위의 코드를 보면 t는 배경 스레드이기때문에 메인스레드 실행과 무관하게 실행된다. 때문에 콘솔이 찍히지 않거나 아주 낮은확률로 첫번째 콘솔라인만 찍히게 된다.
- 여기서 backgroud옵션을 제거하고 실행하면 전경스레드(foreground thread)에 속하게 되므로 스레드 실행 종료 후에 메인스레드가 종료된다.
- 스레드에 인자를 넘길 수 있다. (436p~)



### 다중스레드 자원관리: System.Threading.Monitor
- 한개의 스레드에 할당된 스택의 용량은 1MB이며, 메모리가 허용하는한 원하는만큼 생성할 수 있다.
    - 가령 32비트 윈도우에서 32비트 프로세스는 2GB메모리가 허용되므로, 오직 스레드에만 메모리가 사용된다고 가정해도 2000개를 넘을 수 없다.
    - 그러나 64비트 시대가 열리면서 tera바이트 단위로 메모리 할당이 가능해져 스레드 갯수의 제한이 풀렸다봐도 무방하다.
- 코드상 여러개의 스레드를 동시의 실행시키면 그 실행순서는 담보할 수 없다.
    - 이 경우 '공유 리소스 동기화' 문제가 생긴다. 
    - 이를 해소하기위한 BCL 제공 클래스가 Monitor이다.
{% highlight c# %}
static void ThreadFunc(object inst) {
    Program pg = inst as Program;
    for(int i=0; i<100000; i++) {
        Monitor.Enter(pg);
        try {
            pg.number = pg.number + 1;
        } finally {
            Monitor.Exit(pg);
            //Enter와 Exit 코드 사이에 위치한 모든 코드는 스레드 하나만 진입해서 실행할 수 있다. 즉, 해당 코드가 한 스레드에 의해 점유된 상태면 다른 스레드는 대기상태가 된다.
        }
    }
}
{% endhighlight %}

### 다중스레드 자원관리: lock
- C#은 Monitor와 동일하지만 더 간결한 lock예약어를 제공한다. 다음과 같다.
{% highlight c# %}
static void ThreadFunc(object inst) {
    Program pg = inst as Program;
    for(int i=0; i<100000; i++) {
        Monitor.Enter(pg);
        lock(pg)
        {
            pg.number = pg.number + 1;
        }
    }
}
{% endhighlight %}
- lock이나 Monitor를 사용하지 않는 객체 인스턴스 사용은 thread-safe하지 않다고 한다. MSDN은 BCL 도움말에서 타입별 메서드의 스레드 안정성을 명시하고 있다.
    - 가령 ArrayList타입의 정적멤버는 다중 스레드 접근에 안전하지만 인스턴스는 안전하지 않다고 명시한다.
    - 그럼 전부 thread safe하게 만들면 되지 않나 할 수 있지만 그럼 성능이 나지 않는다!


### 다중스레드 자원관리: interlocked
- BCL은 다중 스레드에서의 공유자원을 사용하는 몇몇 패턴에 대해 정적메서드를 제공한다.
- 가령 32/64비트 숫자 타입의 일부 연산은 interlocked를 통해 처리할 수 있다
{% highlight c# %}
class MyData {
    int number = 0;
    public int Number {get {return number;}}
    public void Increment() {
        Interlocked.Increment(ref number);
    }
}
{% endhighlight %}
- 위와 같은 연산을 원자적인 연산, 즉 쪼갤수 없는 단일한 연산이라고 할 수 있는데, 이는 프로그래밍적인 의미에서 비트 연산을 생각하면 된다.
