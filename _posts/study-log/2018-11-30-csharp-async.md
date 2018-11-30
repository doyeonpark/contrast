---
title:  "c# 기본서 - BCL: 비동기 호출"
date:   2018-11-30 12:02:00
tags: [c#, bcl, thread, async]
categories: study-log
---

### 비동기 호출
- 일반적으로 동기호출을 Blocking 호출이라고 한다. 동기호출을 하면 해당 스레드는 아무것도 수행하지 못한다. 예시로 disk I/O가 있다.
- 비동기 호출은 결국 또 다른 스레드를 만들어 실행시킨다는 점에서 스레드의 한 종류로 볼 수 있다.
- ThreadPool과 비교를 했을때 비동기 스레드는 바로 작업을 수행한다는 점에서 조금 빠를 수 있는데, 이정도 의미가 있는 경우는 이용자 수가 아주 많은경우이다(460p 참조)


### 비동기 호출: Delegate
- I/O말고도 일반 메서드에서도 비동기 호출을 할 수 있는 수단으로, BCL이 제공하는 기능이다.
- 간결하기때문에 좋다.
{% highlight c# %}
public delegate long CalcMethod(int start, int end);
static void Main(string[] args) {
    CalcMethod calc = new CalcMethod(Calc.Cumsum);
     
    IAsyncResult ar = calc.BeginInvoke(1, 100, null, null);
    //delegate타입의 BeginInvoke 메서드 호출하며 ThreadPool의 스레드에서 실행된다

    ar.AsyncWaitHandle.WaitOne(); 
    //AsyncWaitHandle은 EventWaitHandle타입이고, Cal.CumSum이 완료될때까지 현재 스레드를 대기시킨다.

    long result = calc.EndInvoke(ar);
    //반환 값을 얻기 위해 호출하는 메서드, 없어도 반드시 호출하는 것을 권장한다.
    //실행을 차단할 수있으므로 사용자 인터페이스 제공하는 스레드에서는 호출하지 말아야한다
}
public class Calc {
    public static long Cumsum(int start, int end) {
        long sum = 0;
        for(int i=start; i<=end; i++;) {
            sum += i;
        }
        return sum;
    }
}
{% endhighlight %}


### FileStream.BeginRead 구현과 유사하게 비동기 구현하기
- MSDN문서를 보면 delegate구현 방식에 따라 FileStream의 BeginRead/EndRead가 구현되었다.
- BeginInvoke의 세번째 인자로 콜백메서드를 지정해주면된다.
- 네번째 인자는 StateObject로, 해당 메서드를 여러 곳에서 비동기 호출 했을 때 각각의 비동기 호출을 구분하는 역할을 갖는다.(stateObject - A user-provided object that distinguishes this particular asynchronous read request from other requests.) [참고](http://www.java2s.com/Tutorials/CSharp/System.IO/FileStream/C_FileStream_BeginRead.htm)
  
  {% highlight c# %}
public delegate long CalcMethod(int start, int end);
static void Main(string[] args) {
    CalcMethod calc = new CalcMethod(Calc.Cumsum);
    calc.BeginInvoke(1, 100, calcCompleted, calc);
}
static void calcCompleted(IAsyncResult ar) {
    CalcMethod calc = ar,AsyncState as CalcMethod;
    long result = calc.EndInvoke(ar);
}
{% endhighlight %}