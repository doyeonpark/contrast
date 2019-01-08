---
title:  "effective c# - 3. 제네릭 활용"
date:   2019-01-03 12:19:00
tags: [effectivec#, c#]
categories: study-log
---

### 반드시 필요한 제약조건만 설정하라
- 제너릭 타입을 활용할때 타입 조건을 넣고싶다면 제약조건 예약어 [where](parkdoyeon.github.io/study-log/2018-12-05-csharp-2.0-Generic)를 사용하는 것이 좋다
- 그러나 기본값을 할당하는데 있어서 default()와 new()는 분명한 차이가 있으므로, 제약조건에 new()를 넣을땐 이용자가 구현에 어려움을 겪을수도 있다는 것을 생각해야한다.

### 런타입에 타입을 확인하여 최적의 알고리즘을 사용하라
- 제약조건을 사용하지 않고 제너릭 타입을 사용하게되면 런타임에 타입확인이 이뤄진다.
- 이때, 런타임 환경을 최대한 고려하여 최적화된 타입별 생성자/메소드 오버로드등을 작성하는 것이 좋다.

### IComparable<T>와 IComparer<T>를 이용하여 객체의 선후관계를 정의하라
- ICompareable<T>는 현재 객체의 비교대상을 지정해 리턴하고, IComparer<T>는 T타입의 모든 객체의 비교 함수를 정의한다. [참조](https://stackoverflow.com/questions/9316918/what-is-the-difference-between-iequalitycomparert-and-iequatablet)
  - **IComparable** - Defines an interface for an object with a CompareTo() method that takes another object of the same type and compares **the current object** to the passed one. It internalizes the comparison to the object, allowing for a more inline comparison operation, and is useful when there's only one logical way, or an overwhelmingly common default way, to compare objects of a type.
  - **IComparer** - Defines an interface with a Compare() method that **takes two objects of another type** (which don't have to implement IComparable) and compares them. This externalizes the comparison, and is useful when there are many feasible ways to compare two objects of a type, or when the type doesn't implement IComparable (or the IComparable implementation compares a different way than what you want) and you don't have control over that type's source.
- 이때 제네릭 타입이 구현안된 IComparable, ICompar er도 하위 호환(.NET 2.0 이전버전)을 위해 구현하면 좋다.

### 타입 매개변수가 IDisposable을 구현한 경우를 대비하여 제네릭 클래스를 작성하라
- 타입 매개변수가 IDisposable을 구현했을때 using(a as IDisposable) 구문을 사용하여 dispose 호출이 가능하다
  - 만약 a가 IDisposable을 구현하지 않았다면 null이 리턴되고 dispose 호출을 하지 않기때문에, 깔끔한 코드 작성이 된다.
    {% highlight c# %}
    {
        public void GetThingsDone()
        {
            T driver = new T();
            using(driver as IDisposable)
            {
                driver.DoWork();
            }
        }
    }
    {% endhighlight %}


### 공변성과 반공변성을 지원하라 (고민해볼 것)

