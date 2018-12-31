---
title:  "c# 기본서 - 멤버별 유형확장"
date:   2018-10-25 22:10:00
tags: [c#, 객체지향, readonly, const, event, indexer]
categories: study-log
---

### readonly, const
- 둘다 객체의 내부값 변경불가
- 그러나 const는 byte, int, string 등 기본 자료형에 한해서 적용가능
- const는 선언과 함께 값이 대입되어야함. 즉, 생성자에서 접근할 수 없다.
- const는 컴파일시에 소스코드에 값이 직접 치환됨

### event
- 델리게이트 예약어로, 첫번째로 이벤트를 발생시킨 인자를, 두번째 인자로 해당 이벤트에대한 값을 제공받는다.
- 델리게이트 만으로도 구현이 가능하지만, 이벤트 예약어로 구현하면 콜백함수를 구독/해지할 수 있도록 함수를 담아두는 클래스선언을 하지 않아도 된다.
{% highlight c# %}
class SomeClass
{
    public event EventHandler CallbackGroup
}
{% endhighlight %}

### 클래스를 배열처럼 사용하게 하는 Indexer
- 프로퍼티를 활용해서 클래스에 배열처럼 접근하도록 돕는 기능
{% highlight c# %}
class SomeClass
{
    public int this[indexer]
    {
        public get
        {
            if(indexer == "냐옹")
                return "고양이";
            else
                return "강아지";
        }
        public set;
        {
            this[indexer] = value;
        }
    }
    public event EventHandler CallbackGroup
}
{% endhighlight %}
