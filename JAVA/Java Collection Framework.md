# What Is a Collections Framework?
> 컬렉션 프레임워크는 컬렉션을 표현하고 조작하기 위한 통합 아키텍처입니다.

## Interfaces
- 인터페이스는 컬렉션을 나타내는 추상 자료형이다. 구현 세부사항과 독랍적으로 컬렉션을 다룰 수 있게 한다.<br>
구현에 의존하지 않고, 추상에 의존한다. 
```java  
// 선호
List<String> list = new ArrayList<>(); 

// 비선호 
ArrayList<String> list = new ArrayList<>();

// 표현과 구현의 분리 (코드 수정이 거의없이 교체 가능)
List<String> list = new LinkedList<>();
```
## Implementations
- 구현체는 인터페이스의 구체적 구현이다.<br>
  본질적으로 재사용 가능한 구조이다. 
  - ArrayList -> 배열 기반 
  - LinkedList -> 연결리스트 기반 
  - HashSet -> 해시 기반 
  - TreeSet -> 트리 기반 
```java 
// 개발자가 알아야하는 것 
Set<String> set 
```
## Algorithms 
- 알고리즘은 검색,정렬 등 유용한 계산을 수행하는 메서드이다. 
```java 
// Collections 클래스 
Collections.sort(list); 
Collections.reverse(list); 
```
- 이러한 알고리즘은 **다형성(polymorphic)** 을 가짐 -> 동일한 메서드를 해당 컬렉션 인터페이스의 다양한 구현체에서 사용할수 있음을 의미함 
```java 
/**
 * 예시 
 * ArrayList에서도 작동 
 * LinkedList에서 작동 
 * (List인터페이스를 따르기 때문)
 **/
Collections.sort(list); 

List<String> list = new ArrayList<>();
Collections.sort(list); 

List<String> list = new LinkedList<>(); 
Collections.sort(list); 

// 같은 코드 , 다른 내부 동작 
```

## 컬렉션 구조 
- Collection 
  - List    
    - ArrayList 
    - LinkedList 
  - Set 
    - HashSet
    - LinkedHashSet 
    - TreeSet 
  - Queue 
    - LinkedList 
    - PriorityQueue

- Map (별도 계열)
  - HashMap 
  - LinkedHashMap 
  - TreeMap 
  
## 1. List 계열 
### List 

