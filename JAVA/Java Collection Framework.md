# What Is a Collections Framework?
> 컬렉션 프레임워크는 컬렉션을 표현하고 조작하기 위한 통합 아키텍처다.

# Interfaces
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
  
# 1. List 계열 
## 1.1 ArrayList
```java  
// ArrayList는 내부적으로 Object[] 배열을 사용한다. 
List<String> list = new ArrayList<>(); 

// 내부구조 
transient Obejct[] elementData; 

list.add("A");
list.add("B");
```

배열 기반이기 때문에 메모리가 연속적으로 배치된다. 

### 시간 복잡도 
- get(index) -> O(1) 
- add(맨 뒤) -> 평균O(1)
- add(중간) -> O(n)
- remove -> O(n)

### 메모리 특징 
- 연속된 메모리 공간 사용 
- CPU 캐시 친화적
- 내부 capacity가 실제 size보다 클 수 있음 
- 용량이 부족하면 1.5배씩 증가 (배열 복사 발생)

## 1.2 LinkedList 
노드 기반 구조 
```java 
List<String> list = new LinkedList<>();

// 각 노드의 구조 
data + prev + next 
```
### 시간 복잡도 
  - get(index) -> O(n)
  - add(중간) -> O(1)
  - remove -> O(1)



