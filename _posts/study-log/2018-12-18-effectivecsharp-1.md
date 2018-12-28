---
title:  "effective c# - 1. 언어요소"
date:   2018-12-18 16:41:00
tags: [effectivec#, c#]
categories: study-log
---

### 지역변수를 선언할때는 var를 사용하는 것이 낫다
1. c#은 익명함수를 지원하면서 타입을 암시적으로 선언하는 방법을 제공한다.
2. 정확한 타입을 알지 못하는 상태에서 명시적으로 선언할경우 득보다 실이 많다
    - 예1 잘못된 형변환: IEnumerable<T>를 상속받는 IQueryable<T>을 IEnumerable 타입으로 선언하는 경우
      - IQueryable<T> 타입은 원격 데이터를 참조할 경우, 여러단계에서 수행되는 LINQ 쿼리식을 SQL쿼리로 합한 후 그 값을 순회하는 시점까지 SQL쿼리 수행을 연기한다. (=네트워크 트래픽을 적게 씀)
      - 반면 IEnumerable<T>는 단계별로 모두 원격에 SQL쿼리를 전달하고 로컬에 반환한다.
    - 예2 숫자 형변환: 특정 숫자값을 반환하는 메서드를 var 변수를 통해 통해 호출해오면 예상치 못한 결과에 대한 디버깅이 어려울 수 있다.

### const보다 readonly
- const는 값이 컴파일시에 평가되는 컴파일 상수, readonly는 값이 런타임에 평가되는 런타임 상수다.
  - const는 성능이 약간 더 낫지만 유연성이 떨어져 사용이 제한적이다
  - const/readonly 값을 참조하는 코드를 작성할경우 const는 직접 값을 대입하지만 readonly 는 참조코드를 넣는다.
    - const와 readonly가 있는 클래스의 값을 바꾼 dll로 교체하면 const는 호출값이 바뀌지 않지만 readonly는 값이 바뀐다. 즉 const는 전체를 빌드해야한다!


### 캐스트보다는 is, as가 좋다
- 캐스트 연산의 경우 형변환 실패시 `InvalidCastException`이 발생한다. 따라서 안전한 캐스팅을 위해서는 실패시 null 리턴을 반환하는 is/as를 적극 활용하는 것이 좋다.
- foreach구문에서는 참조형식과 값형식 모두 형변환이 가능해야해서 캐스팅 형변환이 일어나기때문에 캐스팅 익셉션이 발생할 수 있다.
  -  다만 컴파일 단계에서 IEnumerator.Current로 변환가능한지(System.Object), 또 다시 for문 앞에 루프변수로 변환가능한지만 확인한다.
    {% highlight c# %}
    {
        //foreach 구문
        public void UseCollection(IEnumerable theCollection)
        {
            foreach(MyType t in theCollection)
                t.DoStuff();
        }

        //실제 foreach가 동작하는 구문
        public void UseCollection(IEnumerable theCollection)
        {
            IEnumerator it = theCollection.GetEnumerator();
            while (it.MoveNext())
            {
                MyType t = (MyType)it.Current;
                t.DoStuff();
            }
        }
    }
    {% endhighlight %}
- 시퀀스 요소를 특정한 타입으로 형변환해주는 IEnumerator.Cast<T>()함수가 있는데, 이 또한 변환 타입에 제약이 발생하므로 캐스팅 연산을 수행한다. 다만, 제너릭타입 컬렉션에 대해서는 호출이 불가능하다.


### 델리게이트를 이용하여 콜백을 표현하라
- 인터페이스가 아닌 델리게이트를 통해 콜백을 표현하면 클래스간의 결합도를 낮춰준다.
- 다만 델리게이트는 한번만 호출해도 델리게이트에 추가된 모든 대상함수가 호출되므로
  - 반환값이 멀티캐스트 체인의 마지막으로 호출된 함수 반환값이 되며, 이전 함수의 반환값은 무시된다는 무시된다는 문제가 있다.
  - 때문에 아래와 같은 루프연산을 통해 반환값을 전체 받을 수 있도록 해야한다.
    {% highlight c# %}
    {
        public void LengthyOperation(Func<bool> pred)
        {
            bool bContinue =ture;
            foreach(ComplicatedClass cl in container)
            {
                cl.DoLengthyOperation();
                foreach(Func<bool> pr in pred.GetInvocationList())
                    bContinue &= pr();

                if (!bContinue)
                    return;
            }
        }
    }
    {% endhighlight %}

### 이벤트 호출시에는 null 연산자를 사용하라
- 다른 스레드로 전환이 된 다음 이벤트 해제가 일어나는 경우가 있을 수 있으므로, 다른 변수에 이벤트 객체를 할당하여 주소복사를 해놓거나 ? 연산자를 통해 nullcheck을 하는 것이 좋다.

### 박싱과 언박싱을 최소화 하라

### 베이스 클래스가 업그레이드 된 경우에만 new 한정자를 사용하라
- 상속받은 메서드에 new 한정자를 사용하면 그 기능에 있어서 개발자에게 혼란을 줄 수 있으므로 다른 이름을 사용해야한다.