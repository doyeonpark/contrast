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
- ICompareable<T>는 객체의 비교대상을 지정해 리턴하고, IComparer<T>는 