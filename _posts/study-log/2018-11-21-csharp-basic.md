---
title:  "c# 기본서 - BCL: Collection, File, Direcotry, Path"
date:   2018-11-21 23:18:00
tags: [c#, bcl, FileStream, File, Directory, Path]
categories: study-log
---

### Collection 
- 정해지지 않은 크기의 배열을 구현한 것을 컬렉션이라고 한다.
- 콜렉션타입 리스트들은 키와 값으로 들어가는 자료형이 Object형식이라 박싱문제가 있다.
- ArrayList
    - 이를 해결하기 위해서는 닷넷 2.0부터 지원되는 Generic이 적용된 List<T>을 사용하는 것이 원장된다.
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


### System.IO.FileStream
- MemoryStream은 메모리에 할당된 바이트배열을 읽고 쓰지만
- FileStream은 디스크 파일을 읽고 쓴다
- 읽을때는 StreamWriter, 가독성을 무시하고 효율적으로 기록을 하려면 BinaryWriter를 사용하면 된다.
- Mode/Access/Share
    - Mode: Open, Create, OpenOrCreate, Truncate(기존데이터 삭제하고 열기), Append(마지막으로 읽던 position으로 이동해서 무조건 열기(Write접근만 가능) - 로깅목적으로 사용함)
    - Access: Read, Write, ReadWrite(읽기쓰기 목적으로 열기), 
    - Share: None(두번이상 열면 무조건 실패, 파일을 맨 처음 열고있는 스트림만 사용하기), Read, Write, ReadWrite(같은 파일을 서로 다른 스트림에서 읽고 쓰는게 가능함)


### System.IO.File, System.IO.FileInfo
- File: 자주 사용되는 조작기능을 담은 정적클래스
    - Copy, Exists, Move(파일이동), Read, Write 등
- FileInfo는 File타입의 기능을 인스턴스 멤버로 일부 구현하고 사용법은 거의 동일하다.


### System.IO.Directory, System.IO.DirectoryInfo, System.IO.Path
- 마찬가지로 Directory는 정적 타입이고, DirectoryInfo는 Directory기능의 일부를 구현한 인스턴스멤버다
- GetFiles, GetDirectories 등을 통해 지정된 경로에 있는 하위 디렉토리/파일 목록을 문자열로 갖고올 수 있다
- 이 경우 '?'와 '*' 와일드카드 사용이 가능하다.
- 가령 GetFiles(targetPath, '???.dll')과 같이 쓸 경우 해당 경로에 있는 파일중 파일명이 3글자인 dll목록을 갖고올 수 있다.
- Move를 통해 디렉토리를 이동할 수 있다


### System.IO.Path
- 경로와 관련한 유용한 정적메서드를 제공하므로 참조할 것 (426p)
- 특히 Path.Combine메서드는 'folder\' + '\temp.exe'와 같이 통일이 안된 경로 문자열을 적절한 방식으로 혼합해준다.
- 사용자로부터 경로를 입력받는 경우 허용되지 않는 문자가 포함될때 exception이 발생할수도 있는데,
- GetInvalidFileNameChars, GetInvalidPathChars같은 경우는 폴더명이나 경로에 허용되지 않는 문자열을 찾아낸다.
  
