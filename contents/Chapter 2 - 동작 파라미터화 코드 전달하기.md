# 동작 파라미터화 코드 전달하기

소비자의 요구 사항은 항상 바뀔 수 있다. 이렇게 시시각각 변하는 사용자 요구사항을 대응하기 위해 우리는 개발 비용을 최소화 하는 것이 좋다. 그 뿐 아니라 새로 추가한 기능은 쉽게 구현할 수 있어야 하며 유지 보수 또한 쉬워야 한다.

**동작 파라미터화**를 이용하면 자주 바뀌는 요구 사항에 효과적으로 대응할 수 있다. 동작 파라미터화란 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록으로, 나중에 프로그램에서 호출한다. 코드 블록의 실행이 나중으로 미뤄지기 때문에 메소드의 인수로 코드 블록을 전달할 수 있고, 코드 블록에따라 메소드의 동작이 파라미터화된다.

## 1. 변화하는 요구사항에 대응하기

변화에 대응하는 코드를 구현하는 것은 어려운 일이다. 예제를 통해 동작 파라미터화가 어떻게 요구 사항에 효과적으로 대응하는지 코드의 개선 과정을 확인할 수 있다. 예제는 농부가 녹색 사과만 필터링하는 프로그램을 요구했다 가정해보자.

### 1) 녹색 사과 필터링

```java
enum Color { RED, GREEN }

public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    
    for(Apple apple : inventory) {
        if **(GREEN.equals(apple.getColor()) {**
            result.add(apple);
        }
    }
    return result;
}
```

굵게 표시된 부분이 녹색 사과만 필터링한 조건이다. 그런데 농부가 빨간 사과도 필터링하고 싶다고 요구사항을 추가하게 되면 어떻게 해야 할까? filterRedApples 메소드를 또 만들어야 할까?

물론 그렇게 해도 빨간 사과를 필터링 할 수는 있겠지만 다른 색깔도 요구한다면 일일히 그것들을 다 만들어 주는 것은 적절한 대응이 아니라고 생각한다. 이런 상황에서 **거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화한다.** 이라는 좋은 규칙이 있다.

### 2) 색을 파라미터화

색을 파라미터화 할 수 있도록 메소드에 파라미터를 추가한다면 변화하는 요구사항에 좀 더 유연하게 대응할 수 있다.

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();
    
    for(Apple apple : inventory) {
        if(apple.getColor().equals(color)) {
            result.add(apple);
        }
    }
}
```

이처럼 color의 값만 바꿔서 넣어주게 되면 원하는 색깔의 사과를 필터링 할 수 있게 된다. 그러나 이번에는 농부가 무게를 기준으로 필터링을 요구를 해 다음과 같이 메소드를 작성하였다.

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
    List<Apple> result = new ArrayList<>();
    
    for(Apple apple : inventory) {
        if(apple.getWeight() > weight) {
            result.add(apple);
        }
    }
}
```

잘못 되진 않았지만 필터링 조건을 적용하는 부분의 코드가 대부분 중복 된다. 이는 소프트웨어 공학의 **DRY**(dont’ repeat yourself) 원칙을 어기는 것이다. 그렇다고 탐색 과정을 고치자니 메소드 전체 구현을 고쳐야 해서 엔지니어링적으로 비싼 대가를 치르게 된다.

### 3) 가능한 모든 속성으로 필터링

다음은 flag를 추가하여 색깔과 무게를 한번에 필터링하는 메소드를 구현한 것이다.

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, 
																															int weight, boolean flag) {
    List<Apple> result = new ArrayList<>();
    
    for(Apple apple : inventory) {
        if((flag && apple.getColor().equals(color)) || 
						(!flag && apple.getWeight() > weight)) {
            result.add(apple);
        }
    }
    return result;
}
```

위와 같이 메소드 구현을 하게 되면 실제로 호출하려면 다음과 같이 사용하게 된다.

```java
List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> heavyApples = filterApples(inventory, null, 150, false);
```

호출 부분만 봤을 때 flag가 의미하는 게 뭔지, 앞으로 요구사항이 바뀌었을 때 유연하게 대응할 수도 없다. 이러한 상황에서 동작 파라미터화를 이용하게 되면 유연하게 대응할 수 있다.

## 2. 동작 파라미터화

앞선 예제에서 파라미터를 추가하는 방법이 아닌 변화하는 요구사항에 좀 더 유연하게 대응할 수 있는 방법이 필요하다는 것을 알 수 있었다. 

이번엔 사과의 속성에 따라 참 또는 거짓을 반환하는 함수(Predicate)를 인터페이스를 통해 구현해 파라미터로 전달해보자.

```java
public interface ApplePredicate {
    boolean test(Apple apple);
}

public class AppleHeavyWeightPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}

public class AppleGreenColorPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return GREEN.equals(apple.getColor());
    }
}
```

이를 전략 디자인 패턴이라고 불리며, 각 알고리즘(전략)을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음 런타임에 알고리즘을 선택하는 기법이다. 

(위의 코드에서 ApplePredicate가 알고리즘 패밀리, AppleHeavyWeightPredicate와 AppleGreenColorPredicate가 전략이다)

### 4) 추상적 조건으로 필터링

ApplePredicate 인터페이스를 사용해 필터링을 하기 위해, 다음처럼 객체를 인수로 받도록 변경할 수 있다. 이렇게 하면 filterApples 메소드 내부에서 컬렉션을 반복하는 로직과 컬렉션의 각 요소에 적용할 동작을 분리할 수 있다는 점에서 소프트웨어 엔지니어링적으로 큰 이득을 얻는다.

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    
    for(Apple apple : inventory) {
        if(p.test(apple)) { // predicate 객체로 검사 조건을 캡슐화함
            result.add(apple);
        }
    }
    return result;
}
```

### 코드/동작 전달하기

앞선 코드에 비해 더 유연한 코드와 동시에 가독성도 좋아졌고 사용하기도 쉬워졌다. 이제 요구사항에 따라 ApplePredicate를 구현하는 클래스만 새롭게 만든다면 쉽게 필터링이 가능할 것이다.

결국 전달하는 ApplePredicate 객체에 따라 filterApples 메소드의 동작이 결정된 것인데, 이는 filterApples 메소드의 **동작을 파라미터화**한 것이다.

### 한 개의 파라미터, 다양한 동작

동작 파라미터화의 강점은 컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다는 것이다. 이를 통해 한 메소드가 다른 동작을 수행하도록 재활용할 수 있어, 유연한 API를 만들 때 중요한 역할을 한다.

## 3. 복잡한 과정 간소화

코드가 많이 유연해졌지만 filterApples 메소드로 새로운 동작을 전달하려면 ApplePredicate 인터페이스를 구현하는 여러 클래스를 정의하고 인스턴스화 해야 한다. 이때 익명 클래스를 이용하면 코드의 양을 줄일 수 있다.

### 5) 익명 클래스 사용

익명 클래스는 말 그대로 이름이 없는 클래스로, 클래스의 선언과 인스턴스화를 동시에 수행할 수 있다. 즉, 바로 필요한 구현을 만들어서 사용할 수 있다.

다음은 익명 클래스를 이용해 ApplePredicate를 구현하는 객체를 만드는 방법으로 필터링 예제를 다시 구현한 코드이다.

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
    public boolean test(Apple apple) {
        **return RED.equals(apple.getColor());**
    }
});
```

하지만 여전히 불필요한 코드가 존재하며 여전히 객체를 만들고 새로운 동작을 정의하는 메소드를 구현해야 한다는 점을 변하지 않았다. 

### 6) 람다 표현식 사용

람다 표현식을 이용하여 위 예제 코드를 다음처럼 간단하게 재구현할 수 있다.

```java
List<Apple> redApples = 
				filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

코드가 훨씬 간결해지면서 원하는 동작을 잘 표현할 수 있게 되었다.

### 7) 리스트 형식으로 추상화

자바의 제네릭 타입을 이용하면 사과말고도 다양한 리스트와 동작을 추가할 수 있게 된다.

```java
public interface Predicate<T> {
    boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList<>();
    for(T e : list) {
        if(p.test(e)) {
            result.add(e);
        }
    }
    return result;
}
```

이런 식으로 구현을 하게 된다면 유연성과 간결함을 모두 잡을 수 있게 될 것이다.

```java
List<Apple> redApples = 
				filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
List<Integer> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);
```

## 4. 마치며

- 동작 파라미터화에서는 메소드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메소드 인수로 전달한다.
- 동작 파라미터화를 이용하면 변화하는 요구사항에 잘 대응할 수 있는 코드를 구현할 수 있으며 엔지니어링 비용을 줄일 수 있다.
- 코드 전달 기법을 이용하면 동작을 메소드의 인수로 전달할 수 있다. 익명 클래스로도 코드를 깔끔하게 만들 수 있지만, **JAVA 8**에서는 인터페이스를 상속받아 여러 클래스를 구현하는 수고를 덜 수 있다.
- 자바 API의 많은 메소드는 정렬, 쓰레드, GUI 처리 등을 포함한 다양한 동작으로 파라미터화할 수 있다. 
