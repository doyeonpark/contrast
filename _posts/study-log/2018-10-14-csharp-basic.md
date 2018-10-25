---
title:  "c# 기본서 - 객체지향"
date:   2018-10-14 18:10:00
tags: [c#, 객체지향, 기본, base, getter, setter, as, static]
categories: study-log
---

### Getter Setter를 사용하면 좋은 이유
- 필드를 직접 호출하여 값변경을 하면 제약을 두고싶을때(ex 환율계산, 값 범위 제한 등 ) 필드값이 사용되는 지점에 직접 가서 변경해야한다.
- Get/Set 프로퍼티 정의를 위한 접근제한을 걸어두면 추후에 유지보수가 용이하다.


### seal
- 다른 클래스가 자신을 상속받지 못하도록 하는 예약어이다


### 접근제한자와 base 예약어
- C#은 접근제한자를 생략하면 기본적으로 internal이 적용된다.
- Class 멤버의 경우 private로 설정된다
- 때문에 부모클래스에서 값을 초기화가 필요한 자식 클래스에서 base 예약어를 통해 직접 초기화해주는 방법을 사용할 수 있다
{% highlight c# %}
class Book
{
    decimal isbn13;
    public Book(decimal isbn13)
    {
        this.isbn13 = isbn13;
    }
}
class EBook : Book
{
    public EBook() : base(0)
    {
    }
    public EBook(decimal isbn) : base(isbn) //이렇게 연계도 가능
    {
    }
}
{% endhighlight %}

### as 연산자
- 부모타입의 자료를 자식타입으로 명시적 형변환하면 컴파일은 일어나지만 실행단계에서 값을 호출하면 오류가 발생한다.
- 런타임 에러를 발생시키는 것은 상당한 부하를 줄 수 있으므로, 형변환 실패시 값을 null로 넣어주는 as 형변환을 권장한다.
- 참조타입에만 사용할 수 있다.

### ToString(), Equals()는 기본타입에 사용될경우 재정의 된다
- equals의 경우 참조 타입을 암시형변환을 통해 재정의 하면 값비교가 가능해진다.

### 정적 생성자의 호출순서
- 최초로 접근하는 시점에 단 한번만 실행된다
- 처음 호출하거나 인스터스 생성자를 통해 객체가 만들어지는 시점에!