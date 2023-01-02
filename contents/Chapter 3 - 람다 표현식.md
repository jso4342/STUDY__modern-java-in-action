# Chapter 3 - 람다 표현식

### 3.1 람다란 무엇인가?
람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단순화한 것이다.

#### 람다의 특징
* **익명** : 보통의 메서드와 달리 이름이 없으므로 **익명**이라 표현 
* **함수** : 특정 클래스에 종속되지 않으므로 함수라고 부른다. 하지만 메서드처럼 파라미터 리스트, 바디, 변환 방식, 가능한 예외 리스트를 포함
* **전달** : 람다 표현식을 메서드 인수로 전달하거나 변수로 저장 가능
* **간결성** : 익명 클래스처럼 많은 코드를 구현할 필요가 없음

그렇다면 람다가 코드를 얼마나 간결하게 바꿀 수 있는지, 아래 예제를 통해 확인해보자. 

**기존 코드**
```
Comparator<Apple> byWeight = new Comparator<Apple>(){
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight));
    }
}
```
**람다 사용** 
```
Comparator<Apple> byWeight = 
    // 람다 파라미터     |화살표|             람다 바디               
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeught());
```
* **파라미터 리스트** : 람다 바디에서 사용할 메서드 파라미터 명시 
* **화살표** : 람다의 파라미터 리스트와 바디를 구분
* **람다 바디** : 람다의 반환값에 해당하는 표현식 
* (parameters) -> expression         
  *  () -> "Kiyoung"
* (parameters) -> { statements; }
  * (String s) -> {return "Kiyoung";}


### 3.2 어디에, 어떻게 람다를 사용하는가?
#### 함수형 인터페이스
함수형 인터페이스는 **오 하나의 추상 메서드를 지정하는 인터페이스**이다. 아무리 많은 디폴트 메서드가 존재하더라도 추상 메서드가 오직 하나이면 함수형 인터페이스이다. 람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로 **전체 표현식을 함수형 인터페이스의 인스턴스로 취급**할 수 있다.  


함수형 인터페이스에는 @FuntionalInterface 노테이션을 붙여주자. @FuntionalInterface는 함수형 인터페이스임을 가리키는 노테이션이다. 만약 애노테이션을 선언했지만 실제로 함수형 인터페이스가 아니면 컴파일러가 에러를 발생시킨다.

#### 함수 디스크립터 
함수형 인터페이스의 추상 메서드는 람다 표현식의 시그니처를 묘사한다. 함수형 인터페이스의 추상 메서드 시그니처를 **함수 디스크립터**라고 부른다.

### 3.3 실행 어라운드 패턴
코드를 **설정**과 **과정**을 두 과정이 둘러싸는 형태의 패턴을 실행 어라운드 패턴(execute around pattern)이라고 부른다. 이때 실제 자원을 처리하는 코드의 동작을 파라미터화 하고 람다를 통해 동작을 전달할 수 있다.

### 3.4 함수형 인터페이스 사용
함수형 인터페이스인 Predicate, Consumer, Function 인터페이스를 알아보자.

#### Predicate

(T) → boolean

```java
@FuncationalInterface
public interface Predicate<T> {
	boolean test(T t);
}
```

#### Consumer

(T) → void

```java
@FuncationalInterface
public interface Consumer<T> {
	void accept(T t);
}
```

#### Function

(T) → R

```java
@FuncionalInterface
public interface Function<T, R> {
	R apply(T t);
}
```

#### 함수형 인터페이스와 예외

java.util.function의 함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않는다. 즉, *예외를 던지는 람다 표현식을 만드려면 확인된 예외를 선언하는 함수형 인터페이스를 직접 정의*하거나 *람다를 try/catch 블록으로 감싸*야 한다.

### 3.5 형식 검사, 형식 추론, 제약

람다 표현식이 어떻게 동작하게 되는지 실제 형식을 파악하자.

#### 형식 검사

람다가 사용되는 콘텍스트(context)를 이용해서 람다의 형식(type)을 추론할 수 있다. 어떤 콘텍스트에서 기대되는 람다 표현식의 형식을 **대상 형식(target type)**이라고 부른다. 형식검사는 다음과 같은 과정으로 진행된다.

1. 람다가 사용된 메서드의 선언을 확인한다.
2. 람다가 사용된 메서드의 파라미터로 대상 형식을 기대한다.
3. 기대하는 파라미터의 함수형 인터페이스를 파악한다.
4. 그 함수형 인터페이스의 함수 디스크립터를 묘사한다.
5. 전달받은 인수의 람다가 그 요구사항을 만족해야 한다.

#### 형식 추론

제네릭을 사용할 때 선언부에 타입 매개변수를 명시하면 생성자에서는 빈 다이아몬드 연산자로 남겨두어도 자바 컴파일러는 생성 객체의 타입을 추론할 수 있다. 람다 표현식도 마찬가지이다. 자바 컴파일러는 람다 표현식이 사용된 콘텍스트를 이용해서 람다 표현식과 관련된 함수형 인터페이스를 추론한다.

```java
// 형식 추론을 하지 않음
Comparator<Apple> c =
	(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

// 형식을 추론함
Comparator<Apple> c = 
	(a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
```

### 지역 변수 사용(제약)

람다 표현식에서는 익명 함수가 하는 것 처럼 자유 변수(파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 활용할 수 있다. 이를 람다 캡처링(lamda capturing)이라 부른다. 하지만 그러려면 지역 변수는 명시적으로 final로 선언되어 있어야 하거나 실질적으로 final로 선언된 변수와 똑같이 사용되어야 한다(이후 재 할당 불가).

인스턴스 변수는 힙에 저장되는 반면 지역 변수는 스택에 위치한다. 람다에서 지역 변수에 바로 접근할 수 있다는 가정하에 람다가 스레드에서 실행된다면 지역 변수를 할당한 스레드가 사라져서 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근하려 할 수 있다. 따라서 자바 구현에서는 원래 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공한다. 따라서 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생긴것이다.

### 3.6 메서드 참조

명시적으로 메서드 명을 참조함으로써 **가독성을 높일 수 있다**. 메서드 참조는 메서드명 앞에 구분자(**::**)를 붙이는 방식으로 사용할 수 있다. Class::method 형식을 취한다. 메서드 참조는 세 가지 유형으로 구분할 수 있다.

1. 정적 메서드 참조 - Integer::parseInt
2. 다양한 형식의 인스턴스 메서드 참조 - String::length
3. 기존 객체의 인스턴스 메서드 참조 - Apple::getWeight

#### 생성자 참조
ClassName::new 처럼 클래스명과 new 키워드를 이용해서 기존 생성자의 참조를 만들 수 있다. 이는 정적 메서드의 참조를 만드는 방식과 비슷하다.

### 람다 표현식을 조합할 수 있는 유용한 메서드

함수형 인터페이스에서는 다양한 유틸리티 메서드를 지원한다. Comparator, Function, Predicate 같은 함수형 인터페이스는 람다 표현식을 조합할 수 있도록 유틸리티 메서드를 제공하며, 간단한 여러 개의 람다 표현식을 조합해서 복잡한 람다 표현식을 만들수 있다. 이 유틸리티 메서드는 **디폴트 메서드**로 제공되어 함수형 인터페이스의 정의를 해치지 않으며 여러 조합을 가능케 하는 유틸리티를 제공한다.

#### Comparator 연결 

- comparing - 비교에 사용할 키를 추출하는 Function 기반의 Comparator를 반환 
- reversed() - 역정렬
- thenComparing - 동일한 조건에 대하여 추가적인 비교환, 즉 두 번째 비교자 생성 
```java
inventory.sort(comparing(Apple::getWeight)
         .reserved()                            // 무게를 내림차순으로 정렬    
         .thenComparing(Apple::getCountry));    // 무게가 같다면 국가별로 정렬 
```

#### Predicate 조합 

- and - and 연산
- or - or 연산
- negate - not 연산
```java
// 빨간색이면서 무거운(150g 이상) 사과 또는 녹색 사과 
Predicate<Apple> redAndHeavyAppleOrGreen = 
    readApple.and(apple -> apple.getWeight() > 150)
             .or(apple -> GREEN.equals(a.getColor())); // Predicate 메서드를 연결해서 더 복잡한 Predicate 객체 생성 
```

#### Function 조합 

- andThen - 이후에 처리할 function 추가
- compose - 이전에 처리되어야할 function 추가
```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h1 = f.andThen(g);
Function<Integer, Integer> h2 = f.compose(g);
int result1 = h1.apply(1); // 4
int result2 = h2.apply(1); // 3
```