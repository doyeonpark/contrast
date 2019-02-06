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

### 타입 매개변수에 대해 메서드 제약 조건을 서정하려면 델리게이트를 사용하라
- 특정 메서드를 구현하기위해서 인터페이스를 설정하기보다, 해당 메서드를 호출부에서 구현하는 것이 더 편리할때가 있다.
  - 가령
  - 특정 매개변수를 받는 생성자 시퀀스 함수를 호출할 때, 타입 제약조건으로 '매개변수가 있는' 생성자를 설정할 순 없다. 여기서 델리게이트를 사용하면 제약을 둘 수 있다. 
    {% highlight c# %}
    {
        List<Point> values = new List<Point>(
            Utilities.Zip(xValues, yValues, (x, y) -> new Point(x, y))
        )
    }
    {% endhighlight %}
  - 인터페이스 구현시 다양한 타입이 전달되면 타입별로 오버라이드를 해야할 수 있다.


### 베이스 클래스나 인터페이스에 대해서 제네릭을 특화하지 말라
- 오버로드된 메서드가 여러개인 경우, 컴파일러가 이 중 하나를 어떻게 선택하는지 정확히 알고있어야 한다.
- 잘못된 오버라이드 함수가 선택되었을때 메서드 내에서 타입확인을 하여 런타임에러를 방지할 수 있겠지만,
  - 제네릭은 본래 런타임에 타입확인을 수행하지 않기 위해 만들어졌다.
  - 박싱/언박싱을 통한 런타임 오버헤드도 문제가 될 수 있다.

### 타입 매개변수로 인스턴스 필드를 만들 필요가 없다면 제네릭 메서드를 정의하라
- 제너릭'클래스'의 경우 호출시마다 타입을 명시적으로 지정해야한다.
  - 이렇게하면 우선 귀찮은 문제가 있고
  - 해당 클래스마다 제네릭 클래스가 구현되었는지 확인해봐야한다.
- 반면 일반 클래스내에서 오버로딩을 통해 제네릭 함수를 구현하면
  - 캐스팅을 통한 문제(박싱/언박싱 오버헤드, 런타임 에러)가 발생하지 않는다


### 제네릭 인터페이스와 논제네릭 인터페이스를 함께 구현하라
- 버전문제!

### 인터페이스는 간략히 정의하고 기능의 확장은 확장메서드를 사용하라
- 인터페이스를 통해 기능추가를 하면 기존에 인터페이스를 구현하고 있는 클래스를 수정해야한다.
- 반면에 확장메서드를 사용하면
  - 모든 구현부의 모든 클래스에서 호출이 가능하고
  - 이용자에게 메소드의 구현방식에대한 가이드가 된다
- 물론 동일한 이름의 확장메서드 명이 이미 구현이 되어있는경우 확장메서드가 호출되는 문제가 있을 수 있지만, 이경우 이름을 바꾸거나 완벽하게 동일한 기능을 하도록 작성하여 피할 수 있다.

### 확장 메서드를 이용하여 구체화된 제네릭 타입을 개선하라
- 확장메서드를 사용하지 않으면 구체화된 제네릭타입을 상속해서 새로운 타입을 만들어야 하는데,
  - 이렇게 하면 상속타입에 대한 제약이 발생한다.
  - 가령 이터레이터 확장메서드를 통해 메서드 구현을 하면될것을 List타입을 상속받아 클래스 내에서 구현하면 다른 이터레이터 형태의 메서드 구현이 어려워진다.