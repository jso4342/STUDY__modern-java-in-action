# Chapter 8 - 컬렉션 API 개선

## 8.1 컬렉션 팩토리

기존 작은 컬렉션을 만드는 팩토리 메서드로는`Arrays.asList()` 가 존재했다.

```java
List<String> dogs 
	= Arrays.asList("Poodle", "Jindo", "Schnauzer");
```

하지만 고정 크기의 리스트를 만들었으므로, 요소를 갱신할 순 있지만 새 요소를 추가하거나 요소를 삭제할 순 없다. (`UnsupportedOperationException` 발생)

```java
List<String> dogs = Arrays.asList("Poodle", "Jindo", "Schnauzer");
dog.add("Shiba Inu");
dog.set(0, "Pomeranian");
```

그렇다면 집합의 경우는 어떨까? asList같은 asSet 메소드는 없으니, 리스트를 인수로 받는 HashSet 생성자를 사용하거나 스트림 API를 사용하는 방법이 존재했다. 

```java
Set<String> dogs1 = new HashSet<>(Arrays.asList("Poodle", "Jindo", "Schnauzer"));

Set<String> dogs2 
		= Stream.of("Poodle", "Jindo", "Schnauzer")
						.collect(toSet());
```

하지만 내부적으로 불필요한 객체 할당을 필요로 한다.그리고 결과는 변환할 수 있는 집합이다.

그렇다면 리스트의 새로운 기능부터, 컬렉션을 만드는 새로운 방법을 살펴보자. 

### 자바 9에서 제공되는 팩토리 메서드

**리스트 팩토리** 

- List.of : 변경할 수 없는 불변 리스트를 만든다.
    - `List<String> dogs = List.of("Poodle", "Jindo", "Schnauzer");`
    - 리스트를 변경하려고 하면 `UnsupportedOperationException` 발생
    - 왜 다중 요소를 받을 수 있도록 API를 만들지 않을까?  **오버로딩 VS 가변 변수**
        - 가변 인수는 추가 배열을 할당하고 리스트로 감쌈 → **배열 할당, 초기화, 가비지 컬렉션을 하는 비용을 지불해야함.** 하지만 고정된 숫자의 요소를 API로 정의하면 이런 비용을 제거. 물론 List.of로 열 개 이상의 요소를 가진 리스트를 만들 수도 있지만 이 때는 가변 인수를 이용하는 메소드가 사용된다.

**집합 팩토리** 

- Set.of : 변경할 수 없는 불변 집합을 만든다.
    - `Set<String> dogs = Set.of("Poodle", "Jindo", "Schnauzer");`
    - 중복된 요소로 집합 생성 시 `IllegalArgumentException`이 발생한다.

**맵 팩토리** 

- Map.of : 키와 값을 번갈아 제공하는 방법으로 맵을 만들 수 있다.
    - `Map<String, Integer> numberOfDogs = Map.of("Poodle", 5, "Jindo", 2, "Schnauzer", 4);`
    - 열 개 이하의 키와 값을 가진 작은 맵을 만들 때는 `Map.of` 메소드가 유용하다.
- Map.ofEntries : Map.Entry<K, V> 객체를 인수로 받아 맵을 만들 수 있다.
    
    ```java
    import static java.util.Map.entry;
    
    Map<String, Integer> numberOfDogs 
    		= Map.ofEntries(entry("Poodle", 5),
    										entry("Jindo", 2),
    										entry("Schnauzer", 4));
    ```
    

## 8.2 리스트와 집합 처리

자바 8의 List와 Set 인터페이스에는 다음과 같은 메서드들이 추가되었다.

**removeIf** : Predicate를 만족하는 요소를 제거

- 코드가 단순해질 뿐더러 버그도 예방할 수 있다.

```java
// 숫자로 시작되는 참조 코드를 가진 트랜잭션을 삭제하는 코드 
for (Iterator<Transaction> iterator = transactions.iterator();
		iterator.hasNext(); ) {
	Transaction transaction = iterator.next();
	if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
		iterator.remove();
		// transactions.remove(transaction);  => (X) 
		// 반복자의 상태는 컬렉션의 상태와 서로 동기화되지 않음 
	}
}

// removeIf 메소드 사용 
transactions.removeIf(transaction -> 
	Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

**replaceAll** : List에서 UnaryOperator 함수를 이용해 요소를 바꿈 

- 예시로 기존 리스트의 앞글자를 대문자로 바꾸는 코드를 작성해보자.

```java
// 기존 스트림 API 
List<String> dogs = Arrays.asList("poodle", "jindo", "schnauzer");
dogs.stream().map(dog -> Character.toUpperCase(dog.charAt(0)) + 
		dog.substring(1))
				.collect(Collectors.toList())
				.forEach(System.out::println);
```

- *하지만 스트림은 "새 문자열 컬렉션"을 만든다. 

만약 기존 컬렉션을 바꾸고 싶다면, ListIterator 객체 (요소를 바꾸는 set() 메소드 지원)을 이용할 수 있다.*

```java
// ListIterator 객체 사용
List<String> dogs = Arrays.asList("poodle", "jindo", "schnauzer");
System.out.println(dogs); // [poodle, jindo, schnauzer]

for (ListIterator<String> iterator = dogs2.listIterator();
      iterator.hasNext(); ) {
      String dog =iterator.next();
      iterator.set(Character.toUpperCase(dog.charAt(0)) + dog.substring(1));
}
System.out.println(dogs); // [Poodle, Jindo, Schnauzer]
```

- *하지만 컬렉션 객체를 Iterator 객체와 혼용하면 반복과 컬렉션이 동시에 이루어져 쉽게 문제가 발생한다.*

```java
// replaceAll 메소드 사용 
dogs.replaceAll(dog -> Character.toUpperCase(code.charAt(0)) + code.substring(1));
```

**sort** : List를 정렬 

```java
Collections.sort(dogs);
System.out.println(dogs); // [Jindo, Poodle, Schnauzer]

dogs.sort(Comparator.naturalOrder());
System.out.println(dogs); // [Jindo, Poodle, Schnauzer]
```

위 메소드들은 스트림 동작과 달리, 기존 컬렉션을 바꾼다. 

## 8.3 맵 처리

**forEach**

- Map.Entry<Key, Value>의 반복자를 이용해 키와 값을 반복할 수 있다.

```java
for(Map.Entry<String, Integer> entry: numberOfDogs.entrySet()) {
		String dog = entry.getKey();
		Integer num = entry.getValue();
		System.out.println("There are " + num + " " + dog + "s");
}
```

- 자바 8부터 Map 인터페이스는 BiConsumer (키와  값을 인수로 받음)를 인수로 받는 forEach 메소드를 지원한다.

```java
numberOfDogs.forEach((dog, num) -> Systemout.println("There are" + num + dog + "s"));
```

**정렬 메서드**

맵의 항목을 값 또는 키를 기준으로 정렬할 수 있다. 스트림 API의 sorted()내부에 정렬 메서드를 인자로 전달한다.

- `Entry.comparingByValue`
- `Entry.comparingByKey`

```java
Map<String, String> favouriteColors
                = Map.ofEntries(entry("Kinu", "Blue"),
                entry("Rachel", "Green"),
                entry("Leo", "Purple"));

        favouriteColors
                .entrySet()
                .stream()
                .sorted(Map.Entry.comparingByKey())
                .forEachOrdered(System.out::println);
//Kinu=Blue
//Leo=Purple
//Rachel=Green

        favouriteColors
                .entrySet()
                .stream()
                .sorted(Map.Entry.comparingByValue())
                .forEachOrdered(System.out::println);
//Kinu=Blue
//Rachel=Green
//Leo=Purple
```

**getOrDefault 메서드**

찾으려는 키가 존재하지 않으면 널이 반환되므로 NPE 방지를 위해 요청 결과가 널인지 확인하는 로직이 필요하다. `getOrDefault` 메서드로 이 문제를 해결할 수 있다. 단, 키가 존재하더라도 값이 널은 상황은 `getOrDefault`가 널을 반환할 수 있다.

```java
Map<String, String> favouriteColors
                = Map.ofEntries(entry("Kinu", "Blue"),
                entry("Rachel", "Green"));

System.out.println(favouriteColors.getOrDefault("Kinu", "Red")); // Blue
System.out.println(favouriteColors.getOrDefault("Leo", "Red"));  // Red 
```

**계산 패턴**

- `computeIfAbsent` : 제공된 키에 해당하는 값이 없거나 null이면, 키를 이용해 새로운 값을 계산하고 맵에 추가, 존재하면 기존 값을 반환
- `computeIfPresent` : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가
    - 계산한 값이 null일 때 맵에 추가하지 않으면, 존재하던 key 또한 제거
- `compute` : 제공된 키로 새 값을 계산하고 맵에 저장

```java
Map<Key, Value> map = new HashMap();

// 특정 키에 해당 값이 없으면 새로 만들어서 맵에 넣어주는 형태의 코드 
Value value = map.get(key);
if (value == null) {
    value = getNewValue(key);
    map.put(key, value);
}

// computeIfAbsent 사용 
Value value = map.computeIfAbsent(key, k -> getNewValue(key));
```

**삭제 패턴**

- 제공된 키에 해당하는 맵 항목을 제거하는 remove 메서드와 더불어, 키가 특정한 값에 연관되어 있을 때만 항목을 제거하면 오버로드 버전 메서드를 제공한다.
- `default boolean remove(Object key, Object value)`

```java
Map<String, String> favouriteColors = new HashMap<>();
favouriteColors.put("Kinu", "Blue");
favouriteColors.put("Rachel", "Green");

String key = "Kinu";
String value = "Blue";

// 기존 코드 
if (favouriteColors.containsKey(key) &&
        Objects.equals(favouriteColors.get(key), value)) {
    favouriteColors.remove(key);
}

// Java 8 
favouriteColors.remove(key, value);
```

**교체 패턴**

맵의 항목을 바꾸는데 사용할 수 있는 메서드이다

- `replaceAll` : BiFunction(두개의 아규먼트로 하나의 결과값)을 적용한 결과로 각 항목의 값을 교체한다. List의 replaceAll과 비슷한 동작을 수행한다.
- `replace` : 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값으로 매핑되었을 때만 값을 교체하는 오버로드 버전도 존재한다.

```java
Map<String, String> favouriteColors = new HashMap<>();
favouriteColors.put("Kinu", "Blue");
favouriteColors.put("Rachel", "Green");
favouriteColors.replaceAll((member, color) -> color.toUpperCase());
System.out.println(favouriteColors); // {Kinu=BLUE, Rachel=GREEN}
```

**합침** 

두 개의 맵을 합칠 때 `putAll` 메소드는 중복된 키가 있다면 원하는 동작이 이루어지지 못할 수 있다.

 새로 제공되는 `merge` 메서드는 중복된 키에 대한 동작(BiFunction)을 정의해줄 수있다.

```java
Map<String, Long> colorsCount = new HashMap<>();
String color = "Red";

// 기존 코드 
long count = colorsCount.get(color);
if (count == null){
    colorsCount.put(color, 1);
} else {
    colorsCount.put(color, count + 1);
}

// Java 8 
colorsCount.merge(color, 1L, (key, count) -> count + 1L);
```

## 8.4 개선된 ConcurrentHashMap

concurrentHashMap는 동시성 친화적이면서 최신 기술을 반영한 HashMap 버전이다. 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용. 또한 동기화된 Hashtable 버전에 비해 읽기 쓰기 연산이 월등함 (HashMap은 비동기로 동작) 

**리듀스와 검색**

- `forEach` : 각 (키, 값) 쌍에 주어진 액션을 수행
- `reduce` : 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
- `search` : 널이 아닌 값을 반환할 때까지 각 (키, 값) 쌍에 함수를 적용

또한 연산에 병렬성 기준값(threshold)를 정해야 한다. 맵의 크기가 기준값보다 작으면 순차적으로 연산을 진행한다. 

- 기준값을 1로 지정하면 공통 스레드 풀을 이용해 병렬성을 극대화할 수 있다.
- 기준값을 Long.MAX_VALUE로 지정하면 한 개의 스레드로 연산을 실행한다.

```java
ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
long a = 1;
Optional<Integer> maxValue =
Optional.ofNullable(map.reduceValuesToInt(a, (Long x) -> x.intValue(), 0, Integer::sum));
```

**계수**

맵의 매핑 개수를 반환하는 `mappingCount` 메서드를 제공한다. 기존에 제공되던 size 함수는 int형으로 반환하지만 long 형으로 반환하는 `mappingCount`를 사용할 때 매핑의 개수가 int의 범위를 넘어서는 상황에 대하여 대처할 수 있을 것이다.

```java
ConcurrentHashMap<Integer, String> conmap = new ConcurrentHashMap<Integer, String>();   
        conmap.put(10, "Java");   
        conmap.put(11, ".net");   
        conmap.put(20, "php");   
        conmap.put(12, "C");   
        conmap.put(30, "C++");   
    
System.out.println("Map : " + conmap);     // Map : {20=php, 10=Java, 11=.net, 12=C, 30=C++}     
System.out.println("keySet : " + conmap.keySet());   // [20, 10, 11, 12, 30]
System.out.println("Map Size  : " + conmap.mappingCount()); // 5 
```

**집합뷰**

ConcurrentHashMap을 집합 뷰로 반환하는 `keySet` 메서드를 제공한다. 맵을 바꾸면 집합도 바뀌고 반대로 집합을 바꾸면 맵도 영향을 받는다. `newKeySet`이라는 메서드를 통해 ConcurrentHashMap으로 유지되는 집합을 만들 수도 있다.
```java
System.out.println("Initial Mappings are: " + conmap);
// Initial Mappings are: {20=php, 10=Java, 11=.net, 12=C, 30=C++}

// 집합 뷰로 변환 
System.out.println("The set is: " + conmap.keySet());
// The set is: [20, 10, 11, 12, 30]
```
