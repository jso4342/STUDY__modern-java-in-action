## 1. 코드 가독성 개선

일반적으로 코드 가독성이 좋다는 것은 **어떤 코드를 다른 사람도 쉽게 이해할 수 있음**을 의미한다. 즉, 코드 가독성을 개선한다는 것은 우리가 구현한 코드를 다른 사람이 쉽게 이해하고 유지보수할 수 있게 만드는 것을 의미한다. 람다, 메소드 참조, 스트림 API를 활용해서 코드 가독성을 개선할 수 있는 세가지 리팩터링 방법이 있다.

- 익명 클래스를 람다 표현식으로 리팩터링하기
- 람다 표현식을 메소드 참조로 리팩터링하기
- 명령형 데이터 처리를 스트림으로 리팩터링하기

### 1) 익명 클래스를 람다 표현식으로 리팩터링하기

익명 클래스는 코드를 장황하게 만들고 쉽게 에러를 일으키기 때문에 더 간결하고 가독성이 좋은 코드를 구현하기 위해 람다 표현식을 이용하는 것이 좋다. 하지만 모든 익명 클래스를 다 람다 표현식으로 변환할 수 있는 것은 아니다.

- 익명 클래스에서 사용한 this와 super의 의미가 다르다
    - 익명 클래스에서 this는 익명 클래스 자신, 람다에서는 람다를 감싸는 클래스를 가리킨다.
- 익명 클래스는 감싸고 있는 클래스의 변수(섀도 변수)를 가릴 수 있다. 하지만 람다 표현식으로는 변수를 가릴 수 없다.
- 익명 클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있다.
    - 넷빈즈와 IntelliJ 등을 포함한 대부분의 통합 개발 환경에서 제공하는 리팩터링 기능을 이용하면 해결된다.

### 2) 람다 표현식을 메소드 참조로 리팩터링하기

람다 표현식은 쉽게 전달할 수 있는 짧은 코드지만 메소드 참조를 이용하면 더 가독성을 높일 수 있다. 메소드 참조의 메소드명으로 코드의 의도를 명확하게 알릴 수 있기 때문이다.

```java
// 람다 표현식
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream()
                        .collect(groupingBy(dish -> {
                                    if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                                    else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                                    else return CaloricLevel.FAT;
}));

// 메소드 참조
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream()
                        .collect(groupingBy(Dish::getCaloricLevel));
```

또한 comparing, maxBy같은 정적 헬퍼 메소드를 활용하는 것도 좋다. 그리고 sum, maxinum 등 자주 사용하는 리듀싱 연산은 메소드 참조와 함께 사용할 수 있는 내장 헬퍼 메소드를 제공한다. 내장 컬렉터를 이용하면 더 명확한 코드를 작성할 수 있다.

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
// comparing 활용
inventory.sort(comparing(Apple::getWeight));

int totalCalories = menu.stream().map(Dish::getCalories).reduce(0, (c1, c2) -> c1 + c2)
// 내장 컬렉터 summingInt 사용
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories))
```

### 3) 명령형 데이터 처리를 스트림으로 리팩터링하기

스트림 API는 데이터 처리 파이프라인의 의도를 더 명확하게 보여준다. 

```java
List<String> dishNames = new ArrayList<>();
for(Dish dish: menu) {
    if(dish.getCalories() > 300) {
        dishNames.add(dish.getName());
    }
}
//스트림 API 이용과 병렬화한 코드
menu.parallelStream().filter(d -> d.getCalories() > 300)
                     .map(Dish::getName)
                     .collect(toList());
```

첫번째 코드는 전체 구현을 자세히 살펴본 이후에야 전체 코드의 의도를 이해할 수 있다. 게다가 코드를 병렬로 실행시키는 것은 매우 어렵다.

하지만 스트림 API를 이용하면 문제를 더 직접적으로 기술할 수 있고 쉽게 병렬화까지 할 수 있다.

## 2. 람다로 객체지향 디자인 패턴 리팩터링하기

디자인 패턴은 다양한 패턴을 유형별로 정리한 것으로 공통적인 소프트웨어 문제를 설계할 때 재사용할 수 있는, 검증된 청사진을 제공한다. 디자인 패턴에 람다 표현식이 더해지면 이전에 디자인 패턴으로 해결하던 문제를 더 쉽고 간단하게 해결할 수 있고 기존의 많은 객체 지향 디자인 패턴을 제거하거나 간결하게 재구현할 수 있다.

### 1) 전략 패턴(Strategy Pattern)

전략 패턴은 **한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법**이다. 다양한 기준을 갖는 입력값을 검증하거나, 다양한 파싱 방법을 사용하거나, 입력 형식을 설정하는 등의 시나리오에 적용할 수 있다. 전략패턴은 다음 세 부분으로 구성된다.

- 알고리즘을 나타내는 인터페이스 (Strategy 인터페이스)
- 다양한 알고리즘을 나타내는 한 개 이상의 인터페이스 구현 (ConcreteStrategyA, ConcreteStrategyB 같은 구체적인 구현 클래스)
- 전략 객체를 사용하는 한 개 이상의 클라이언트

```java
public interface ValidationStrategy {
	boolean execute(String s);
}

public class IsAllLowerCase implements ValidationStrategy {
	public boolean execute(String s) {
		return s.matches("[a-z]+");
	}
}

public class IsNumeric implements ValidationStrategy {
	public boolean execute(String s) {
		return s.matches("\\d+");
	}
}
```

위와 같이 소문자 또는 숫자로 이루어져야 하는 등 텍스트 입력이 다양한 조건에 맞게 포맷 되어 있는지 검증한다고 가정했을 때의 구현이다. 

```java
//람다 사용 전
Validator numericValidator = new Validator(new IsNumberic());
boolean b1 = numericValidator.validate("aaaa");
Validator lowerCaseValidator = new Validator(new IsAllLowwerCase());
boolean b2 = lowerCaseValidator.validate("bbbb");

//람다 사용 후
Validator numericValidator = new Validator((String s) -> s.matches("[a-z]+"));
boolean b1 = numericValidator.validate("aaaa");
Validator lowewrCaseValidator = new Validator((String s) -> s.matches("\\d+"));
boolean b2 = lowerCaseValidator.validate("bbbb");
```

위 코드에서 확인할 수 있듯이 람다 표현식을 이용하면 전략 디자인 패턴에서 발생하는 자잘한 코드를 제거할 수 있고 코드 조각(또는 전략)을 캡슐화 한다.

### 2) 템플릿 메소드****(Template Method Pattern)****

**알고리즘의 개요를 제시한 다음, 알고리즘의 일부를 고칠 수 있는 유연함을 제공해야 할 때**는 템플릿 메서드 ****디자인 패턴을 이용하는 것이 좋다.

다음은 온라인 뱅킹 애플리케이션의 동작을 정의하는 추상메소드이다.

```java
//람다 사용 전
abstract class OnlineBanking {
    public void processCustomer(int id) {
        Customer c = Database.getCustomerWithId(id);
        makeCustomerHappy(c);
    }
    abstract void makeCustomerHappy(Customer c);
}
//람다 사용 후
public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
    Customer c = Database.getCustomerWithId(id);
    makeCustomerHappy.accept(c);
}
new OnlineBankingLambda().processCustomer(1337, (Customer c) ->
            System.out.println("Hello " + c.getName())
```

람다 사용전에는 각각의 지점들은 OnlineBanking 클래스를 상속받아 makeCustomerHappy 메소드가 원하는 동작을 수행하도록 구현해야한다. 하지만 람다를 사용한다면 OnlineBanking 클래스를 상속받지 않고 직접 람다 표현식을 전달해 다양한 동작을 추가할 수 있다.

### 3) 옵저버 패턴(Observer Pattern)

어떤 이벤트가 발생했을 때 **주체가 되는 한 객체가 옵저버라 불리는 다른 객체 리스트에게 자동으로 알림을 보내야 하는 상황**에서 옵저버 디자인 패턴을 사용한다.

실제 코드로 옵저버 패턴이 어떻게 동작하는 지 살펴보면 다양한 옵저버를 그룹화할 Observer 인터페이스가 필요하고 인터페이스는 새로운 트윗이 있을 때 주제(Feed)가 호출할 수 있도록 notify라고 하는 하나의 메소드를 제공한다.

그리고 트윗에 포함된 다양한 키워드에 다른 동작을 수행할 수 있는 여러 옵저버를 정의할 수 있다.

```java
class Feed implements Subject {
    private final List<Observer> observers = new ArrayList<>();
    public void registerObserver(Observer o) {
        this.observers.add(o);
    }
    public void notifyObservers(String tweet) {
        observers.forEach(o -> o.notify(tweet));
    }
}

Feed f = new Feed();
f.registerObserver(new NYTimes());
f.registerObserver(new Guardian());
f.registerObserver(new LeMonde());
```

여기서 Observer 인터페이스를 구현하는 모든 클래스는 하나의 메소드 notify를 구현했다. 즉, 트윗이 도착했을 때 어떤 동작을 수행할 것인지 감싸는 코드를 구현한 것이다. 지금까지 살펴본 것처럼 람다는 불필요한 감싸는 코드를 제거하는데 좋은 모습을 보인다. 즉, 세 개의 옵저버를 명시적으로 인스턴스화하지 않고 람다 표현식을 직접 전달해서 실행할 동작을 지정할 수 있다.

```java
f.registerObserver((String tweet) -> {
    if(tweet != null && tweet.contains("money")) {
        System.out.println("Breaking news in NY! " + tweet);
    }
});
f.registerObserver((String tweet) -> {
    if(tweet != null && tweet.contains("queen")) {
        System.out.println("Yet more news from London... " + tweet);
    }
});
```

이 예제에서는 실행해야 할 동작이 비교적 간단하므로 람다 표현식으로 불필요한 코드를 제거하는 것이 바람직하지만 옵저버가 상태를 가지며, 여러 메서드를 정의하는 등 복잡하다면 람다 표현식보다 기존의 클래스 구현방식을 고수하는 것이 바람직할 수도 있다.

### 4) 의무 체인(Chain of Responsibility Pattern)

작업 처리 객체의 체인(동작 체인 등)을 만들 때는 **의무 체인 패턴**을 사용한다. 한 객체가 어떤 작업을 처리한 다음에 다른 객체로 결과를 전달하고, 다른 객체도 해야 할 작업을 처리한 다음에 또 다른 객체로 전달하는 식이다.

```java
public abstract class ProcessingObject<T> {
    protected ProcessingObject<T> successor;
    public void setSuccessor(ProcessingObject<T> successor) {
        this.successor = successor;
    }

    public T handler(T input) {
        T r = handleWork(input);
        if (successor != null) {
					return successor.handler(r);
				}
	      return r;
    }

    abstract protected T handleWork(T input);
}
```

handler 메소드를 보면 일부 작업을 어떻게 처리해야 할지 전체적으로 기술한다. ProcessingObject 클래스를 상속받아 handle 메소드를 구현하여 다양한 종류의 작업 처리 객체를 만들 수 있다.

이 패턴을 어떻게 활용할 수 있는지 실질적인 예제를 살펴보자. 다음의 두 작업 처리 객체는 텍스트를 처리하는 예제이다.

```java
public class HeaderTextProcessing extends ProcessingObject<String> {
    public String handleWork(String text) {
        return "From Raoul, Mario and Alan: " + text;
    }
}

public class SpellCheckerProcessing extends ProcessingObject<String> {
    public String handleWork(String text) {
        return text.replaceAll("lambda", "lambda");
    }
}

ProcessingObject<String> p1 = new HeaderTextProcessing();
ProcessingObject<String> p2 = new SpellCheckerProcessing();
p1.setSuccessor(p2); // 두 작업 처리 객체를 연결한다. 
String result = p1.handle("Aren't labdas really sexy?!!");
System.out.println(result);
```

이 패턴은 함수 체인(함수 조합)과 비슷하다. 작업 처리 객체를 Function<String, String>, 더 정확히 말하자면 UnaryOperator<String>형식의 인스턴스로 표현할 수 있다. andThen 메서드로 이들 함수를 조합해서 체인을 만들 수 있다.

```java
UnaryOperator<String> headerProcessing = 
    (String text) -> "From Raoul, Mario and Alan: " + text;
UnaryOperator<String> spellCheckerProcessing = 
    (String text) -> text.replaceAll("labda", "lambda");
Function<String, String> pipeline = 
    headerProcessing.andThen(spellCheckerProcessing); // 동작 체인으로 두 함수를 조합한다. 
String result = pipeline.apply("Aren't labdas really sexy?!!");
```

### 5) 팩토리****(Factory Pattern)****

인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 팩토리 디자인 패턴을 사용한다. 예를 들어 은행에서 취급하는 대출, 채권, 주식 등 다양한 상품을 만들어야 한다고 가정하자.

다음 코드에서 보여주는 것처럼 다양한 상품을 만드는 Factory 클래스가 필요하다.

```java
public class ProductFactory {
    switch(name) {
        case "loan" : return new Load();
        case "stock" : return new Stock();
        case "bond" : return new Bond();
        default: throw new RuntimeException("No such product " + name):
    }
}
```

여기서 Loan, Stock, Bond는 모두 Product의 서브형식이고 createProduct 메소드는 생상된 상품을 설정하는 로직을 포함할 수 있다. 이는 부가적인 기능일 뿐 위 코드의 진짜 장점은 생성자와 설정을 외부로 노출하지 않음으로써 클라이언트가 단순하게 상품을 생산할 수 있다는 것이다.

```java
Product p = ProductFactory.createProduct("loan")
```

생성자도 메소드 참조처럼 접근할 수 있다. 예를 들어 다음은 Loan 생성자를 사용하는 코드다.

```java
Supplier<Product> loanSupplier = Loan::new;
Loan loan = loanSupplier.get();
```

이제 다음 코드처럼 상품명을 생성자로 연결하는 Map을 만들어서 코드를 구현할 수 있다.

```java
final static Map<String, Supplier<Product>> map = new HashMap<>();

static {
    map.put("loan", Loan::new);
    map.put("stock", Stock::new);
    map.put("bond", Bond::new);
}
```

이제 Map을 이용해 팩토리 디자인 패턴에서 했던 것처럼 다양한 상품을 인스턴스화할 수 있다.

```java
public static Product createProduct(String name) {
    Supplier<Product> p = map.get(name);
    if(p != null) return p.get();
    throw new IllegalArgumentException("No such product + " + name);
}
```

팩토리 패턴이 수행하던 작업을 자바 8의 새로운 기능으로 깔끔하게 정리했다. 하지만 팩토리 메소드 createProduct가 상품 생성자로 여러 인수를 전달하는 상황에서는 이 기법을 적용하기 어렵다. 단순한 Supplier 함수형 인터페이스로는 이 문제를 해결할 수 없다.

예를 들어 세 인수를 받는 경우에는 세 인수를 지원하기위해 TriFunction이라는 특별한 함수형 인터페이스를 만들어야 한다. 결국 Map의 시그니처가 복잡해진다.

```java
public interface TriFunction<T, U, V, R> {
	R apply(T t, U u, V v);
}
Map<String, TriFunction<Integer, Integer, String, Product>> map = new HashMap<>();
```

## 3. 람다 테스팅

개발자의 최종 업무 목표는 제대로 작동하는 코드를 구현하는 것이지 깔끔한 코드를 구현하는 것이 아니다.

일반적으로 좋은 소프트웨어 공학자라면 프로그램이 의도대로 동작하는지 확인할 수 있는 `단위 테스팅`을 진행한다. 우리는 소스 코드의 일부가 예상된 결과를 도출할 것이다. 단언하는 테스트 케이스를 구현한다. 예를 들어 다음처럼 그래픽 애플리케이션의 일부인 **Point** 클래스가 있다고 가정하자.

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() { return x; }
    public int getY() { return y; }
    public Point moveRightBy(int x) {
        return new Point(this.x + x, this.y);
    }
}
```

다음은 moveRightBy 메서드가 의도한 대로 동작하는지 확인하는 단위 테스트이다.

```java
@Test
public void testMoveRightBy() throws Exception {
    Point p1 = new Point(5, 5);
    Point p2 = p1.moveRightBy(10);
    assertEquals(15, p2.getX());
    assertEquals(5, p2.gety());
}
```

이 테스트는 문제없이 작동한다. 하지만 람다는 익명이므로 테스트 코드 이름을 호출할 수 없다. 하지만 람다는 익명이므로 테스트 코드 이름을 호출할 수 없다. 따라서 필요하다면 람다를 필드에 저장해서 재사용할 수 있으며 람다의 로직을 테스트할 수 있다.

람다 표현식은 함수형 인터페이스의 인스턴스를 생성한다는 사실을 기억하자. 따라서 생성된 인스턴스의 동작으로 람다 표현식을 테스트할 수 있다. 다음은 Comparator 객체 compareByXAndThenY 에 다양한 인수로 compare 메서드를 호출하면서 예상대로 동작하는지 테스트하는 코드다.

```java
@Test
public void testComparingTwoPoints() throws Exception {
    Point p1 = new Point(10, 15);
    Point p2 = new Point(10, 20);
    int result = Point.compareByXAndThenY.compare(p1, p2);
    assertTrue(result < 0);
}
```

### 1) 람다를 사용하는 메서드의 동작에 집중하라

람다의 목표는 정해진 **동작을** 다른 메서드에 사용할 수 있도록 하나의 조각으로 **캡슐화**하는 것이다. 그러려면 세부 구현을 포함하는 람다 표현식을 공개하지 말아야 한다. 람다 표현식을 사용하는 메서드의 동작을 테스트함으로써 람다를 공개하지 않으면서도 람다 표현식을 검증할 수 있다.

```java
public static List<Point> moveAllPointsRightBy(List<Point> points, int x) {
    return points.stream()
                .map(p -> new Point(p.getX() + x, p.getY())
                .collect(toList());
}
```

위 코드에 람다 표현식 p -> new Point(p.getX() + x, p.getY()); 를 테스트하는 부분은 없다. 그냥 moveAllPointsRightBy 메서드를 구현한 코드일 뿐이다. 이제 moveAllPointsRightBy 메서드의 동작을 확인할 수 있다.

```java
@Test
public void testMoveAllPointsRightBy() throws Exception {
    List<Point> points = Arrays.asList(new Point(5, 5), new Point(10, 5));
    List<Point> expectedPoints = Arrays.asList(new Point(15, 5), new Point(20, 5));
    List<Point> newPoints = Point.moveAllPointsRightBy(points,10);
    assertEquals(expectedPoints, newPoints);
}
```

위 단위 테스트에서 보여주는 것처럼 Point 클래스의 equals 메서드는 중요한 메서드다. 따라서 Object의 기본적인 equals 구현을 그대로 사용하지 않으려면 equals 메서드를 적절하게 구현해야 한다.

### 2) 복잡한 람다를 개별 메서드로 분할하기

복잡한 람다 표현식을 접할 수 있다. 그런데 테스트 코드에서 람다 표현식을 참조할 수 없는데, 복잡한 람다 표현식을 어떻게 테스트할 것인가? 한 가지 해결책은 람다 표현식을 `메서드 참조`로 바꾸는 것이다. 그러면 일반 메서드를 테스트하듯이 람다 표현식을 테스트할 수 있다.

### 3) 고차원 함수 테스팅

함수를 인수로 받거나 다른 함수를 반환하는 메서드는 좀 더 사용하기 어렵다. 메서드가 람다를 인수로 받는다면 다른 람다로 메서드의 동작을 테스트할 수 있다.

```java
@Test
public void testFilter() throws Exception {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4);
    List<Integer> even = filter(numbers, i -> i % 2 == 0);
    List<Integer> smallerThanThree = filter(numbers, i -> i < 3);
    assertEquals(Arrays.asList(2, 4), even);
    assertEquals(Arrays.asList(1, 2), smallerThanThree);
}
```

테스트해야 할 메서드가 다른 함수를 반환한다면 어떻게 해야 할까? 이때는 Comparator에서 살펴봤던 것러럼 함수형 인터페이스의 인스턴스로 간주하고 함수의 동작을 테스트할 수 있다.

## 4. 디버깅

문제가 발생한 코드를 디버깅할 때 개발자는 다음 두 가지를 가장 먼저 확인한다.

- 스택 트레이스
- 로깅

하지만 람다 표현식과 스트림은 기존의 디버깅 기법을 무력화한다.

### 1) 람다와 스택 트레이스

유감 스럽게도 람다 표현식은 이름이 없기 때문에 조금 복잡한 스택 트레이스가 생성된다. 다음은 고의적으로 문제를 일으키도록 구현한 간단한 코드다.

```java
import java.util.*;

public class Debugging {
    public static void main(String[] args) {
        List<Point> points = Arrays.asList(new Point(12, 2), null);
        points.stream().map(p -> p.getX()).forEach(System.out::println);
    }
}
```

프로그램을 실행하면 다음과 같은 스택 트레이스가 출력된다.

> Exception in thread "main" java.lang.NullPointerExceptionat Debugging.lambda$0(Debugging.java:6)at Debugging$$Lambda$5/284720968.apply(Unknow Source)at java.util.stream.ReferencePipeline$3$1 .accept(ReferencePipeline.java:193)at java.util.Spliterators$ArraySpliterator.forEachRemaining(Spliterators.java:948)
> 

points 리스트의 둘째 인수가 null이므로 프로그램의 실행이 멈췄다. 스트림파이프라인에서 에러가 발생했으므로 스트림 파이프라인 작업과 관련된 전체 메서드 호출 리스트가 출력되었다. 이와 같은 이상한 문자는 람다 표현식 내부에서 에러가 발생했음을 가리킨다. 람다 표현식은 이름이 없으므로 컴파일러가 람다를 참조하는 이름을 만들어낸 것이다. lambda$main$0는 다소 생소한 이름이다. 클래스에 여러 람다 표현식이 있을 때는 골치 아픈 일이 벌어진다.

메서드 참조를 사용해도 스택 트레이스에는 메서드명이 나타나지 않는다. 메서드 참조를 사용하는 **클래스와 같은 곳에 선언되어 있는 메서드를 참조할 때는 메서드 참조 이름이 스택 트레이스에 나타난다**.

따라서 람다 표현식과 관련한 스택 트레이스는 이해하기 어려울 수 있다는 점을 인지하자.

### 2) 정보 로깅

스트림의 파이프라인 연산을 디버깅한다고 가정했을 때 스트림은 최종연산 구문을 호출하는 순간 전체 스트림이 바로 소비된다. 스트림 파이프라인에 적용된 각각의 연산(map, filter, limit)이 어떤 결과를 도출하는지 확인할 수 있다면 좋을 것 같다. 바로 이때, `peek`이라는 스트림 연산을 활용할 수 있다. peek은 스트림의 각 요소를 소비할 것처럼 동작을 실행한다. 하지만 forEach처럼 실제로 스트림의 요소를 소비하지는 않는다. peek는 자신이 확인한 요소를 파이프라인의 다음 연산으로 그대로 전달한다.

### 5. 마치며

- 람다 표현식으로 가독성이 좋고 더 유연한 코드를 만들 수 있다.
- 익명 클래스는 람다 표현식으로 바꾸는 것이 좋다. 하지만 이때 this, 변수 섀도 등 미묘하게 의미상 다른 내용이 있음을 주의하자.
- 메소드 참조로 람다 표현식보다 더 가독성이 좋은 코드를 구현할 수 있다.
- 반복적으로 컬렉션을 처리하는 루틴은 스트림 API로 대체할 수 있을지 고려하는 것이 좋다.
- 람다 표현식으로 전략, 템플릿 메소드, 옵저버, 의무 체인, 팩토리 등의 객체지향 디자인 패턴에서 발생하는 불필요한 코드를 제거할 수 있다.
- 람다 표현식도 단위 테스트를 수행할 수 있다. 하지만 람다 표현식 자체를 테스트하는 것보다는 람다 표현식이 사용되는 메소드의 동작을 테스트하는 것이 바람직하다.
- 복잡한 람다 표현식은 일반 메소드로 재구현할 수 있다.
- 람다 표현식을 사용하면 스택 트레이스를 이해하기 어려워진다.
- 스트림 파이프라인에서 요소를 처리할 때 peek 메소드로 중간값을 확인할 수 있다.
