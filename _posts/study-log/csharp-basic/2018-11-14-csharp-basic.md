---
title:  "c# 기본서 - BCL: 시간계산, Collection, 문자열처리, 직렬화와 Stream"
date:   2018-11-14 23:10:00
tags: [c#, bcl, DateTime, collection, encoding, regex, string, serialization, stream]
categories: study-log
---

### UTC, GMT
- 영국 기준 동쪽인 곳은 시간이 증가하므로 대한민국은 GMT +9이다. 이 시간을 Korean Standard Time이라고 한다.
- 세계협정시(Universal Time Coordinated)가 GMT를 제치고 표준시로 인정받고있지만 GMT와 소수점 차이로 시간차이가 있기 때문에 일반인 입장에선 영향받는 부분이 없다.
- DateTime.Now 는 local time을 반환하므로 UTC +9를 반환한다
- 닷넷 DateTime은 1년 1월 1일 기준으로 시간 기준값을 담는다.
- 그러나 자바의 DateTime은 1970년 1월 1일(Epoch) 기준값을 담는다. 이 시간을 Epoch Time이라고 한다.
- 때문에 System.currentTimeMillis == (DateTime.UtcNow.Ticks-1970년1월1일까지의 Ticks)*10000

### 시간 계산
- DateTime 연산중에 유일하게 가능한 것이 빼기이고, 연산 결과는 TimeSpan에 담긴다.
- 더 정확한 시간계산은 StopWatch를 통해 가능하다. 성능측정시 자주쓰인다.
{% highlight c# %}
Stopwatch st = new Stopwatch();
st.Start();
//동작 수행
st.Stop();
Console.WriteLine("Millisecond: "+st.ElapsedTicks/10000);
{% endhighlight %}


### Collection 
- 정해지지 않은 크기의 배열을 구현한 것을 컬렉션이라고 한다.
- 콜렉션타입 리스트들은 키와 값으로 들어가는 자료형이 Object형식이라 박싱문제가 있다.
- ArrayList
    - 이를 해결하기 위해서는 닷넷 2.0부터 지원되는 Generic이 적용된 List<T>을 사용하는 것이 권장된다.
    - 같은 타입 Object를 사용하면 Sort()를 통해 순서를 정리할 수 있다.
    - 사용자 지정타입 Object의 경우 Sort의 델리게이트 인자를 받아 구현하면 된다.
- Hashtable
    - value뿐만이 아니라 바로 hash값이 있는 key가 있는 배열이다.
    - 때문에 ArrayList와 비교했을때 빠른 검색속도에 유리하다.
    - 중복 key가 들어가면 Argument Exception이 발생하므로 유의
- SortedList
    - Hashtable과 유사하지만,
    - key자체가 정렬되어 값의 순서에 영향을 준다.
- Stack, Queue


### 문자열 처리
- IndexOf() 문자열의 위치 반환하고 없으면 -1 반환
- string.Format()을 통해 문자열의 반환 형태를 조정할 수있다. 아래 참조.
{% highlight c# %}
string.Format("날짜: {0, -20:D}, 판매 수량: {1, 15:N}", DateTime.Now, 267)
//Format 함수에 0번째 인자를 상세 날짜형식(D)으로 쓰고, 왼쪽부터 작성해서 20번 인덱스 위치까지 여백은 오른쪽에 남긴다.
//Format 함수에 1번째 인자를 숫자 형식(N)으로 쓰고, 오른쪽에 붙여서 작성해서 15번찌 위치까지 여백은 왼쪽에 남긴다.
{% endhighlight %}
- string에 대한 모든 변환은 새로운 메모리 할당을 발생시킨다. 
- 때문에 StringBuilder를 사용한 다음에 toString()으로 변환시켜주자.

### 인코딩, 정규식
- 문자가 숫자(컴퓨터가 이해할 수 있는 언어)로 표현되는 것을 인코딩(부호화)라고 한다.
- Regex는 생성자의 두번쨰 인자로 대소구분을 할것인지 말것인지 옵션을 줄 수 있다.
- Regex객체의 Replace()기능은 두번째 인자로 델리게이트 메소드를 받는다. 첫번쨰 인자와 일치하는 값을 찾아서 변환해준다.
{% highlight c# %}
string txt = "Hello, World! Welcome to my world!";
Regex regex = new Regex("world", RegexOptions.IgnoreCase);
string result = regex.Replace(txt, funcMatch);

static string funcMatch(Match match)
{ return "Universe"; }
{% endhighlight %}

### 직렬화/역직렬화
- 네트워크선을 타고 이동하는 데이터는 결국 byte데이터이다
- 좁은의미에서 데이터를 일련의 바이트 데이터로 변환하는걸 직렬화(serialization)라고 하고
- 복원할 수 있는 모든 작업을 역직렬화(deserialization)이라고 한다
- 직렬화를 위한 클래스는 MemoryStream, StreamWriter, BinaryWriter 등이 있다

### 일반 데이터의 직렬화: MemoryStream, StreamWriter, BinaryWriter
- MemoryStream은 메모리에 바이트 데이터를 순서대로 읽고 쓰는 작업을 한다
    - position을 0으로 잡고 지정한 바이트 공간만큼 쓰는 작업을 한다.
    - 읽어올땐 Converter를 사용한다
- StreamWriter는 MemoryStream에서 바이트 변환시 인코딩 방식을 지정하는 불편함을 해소하기위해 나타났다
    - 즉 문자열 Stream에 쉽게 쓸 수 있도록 MS가 포함시킨것
    - Write로 들어온 문자열을 내부 버퍼에 보관하고 있다가 일정 크기에 다다르면 한꺼번에 쓰기 작업을 한다.
    - Flush는 다 차기 전에 무조건 Stream에 쓰는 역할을 하기 때문에 마지막에 한번만 호출하도록 한다.
- BinaryWriter는 문자열이 아닌 이진데이터에 특화된 기능에 쓰인다.


### 사용자 정의 클래스의 직렬화: BinaryFormatter
- 사용자 정의 클래스는 직렬화를 하기 위해 BitConverter 등을 통해 배열로 바꾼다음 Stream에 쓰면 되지만 포매터를 사용하면 더 편리하다.
- BinaryFormatter를 사용하려면
    - 클래스 상위에 [Serializable] Attribute를 지정해주면 되고,
    - 특정 필드 값을 제외하고 싶다면 그 값 위에 [NonSerialized] Attribute를 달아주면된다.
    - 문제는 BinaryFormatter는 닷넷 내부에서 정의된 것이므로 닷넷간의 통신만 가능하다는 점이다.
- 그 대안으로 XmlSeralizer가 있다
    - Xml형식으로 직렬화를 하는데, 태그를 열고 닫기 위해 문자열을 지나치게 많이 사용한다는 문제가 있다.
    - 결론적으로 닷넷간에는 BinaryFormatter, 이기종간에는 XmlSerializer를 쓰면 되는데...
- 대세는 JsonSerializer
    - Xml이 사용하는 문제열 문제를 괄호{}를 통해 해소했다.
    - 다만 닷넷에선 System.Runtime.Serialization.dll을 참조추가해야한다. 
