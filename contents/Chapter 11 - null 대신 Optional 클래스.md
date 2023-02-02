# null참조의 문제점과 null을 멀리해야하는 이유

### 값이 없는 상황을 어떻게 처리해야하는가?

```java
public class Person {
	private Car car;
	public Car getCar() {return car;}
}

public class Car {
	private Insurance insurance;
	public Insurance getInsurance() {return insurance;}
}

public class Insurance {
	private String name;
	public String getName() {return name;}
}
```

```java
public String getCarInsuranceName(Person person) {
	return person.getCar().getInsurance().getName();
}
```

위의 코드를 바탕으로 getCarInsuranceName()메서드를 호출할때 어떤 문제가 발생할까?
person중 car를 가지고 있지 않는다면 getInsurance는 null참조의 정보를 반환하려 할 것이므로 런타임에 NPE가 발생할 것이다. 또 다른 문제로 person이 null이라면? 아니면 getInsurance가 null을 반환하게 된다면?

### 보수적인 자세로 NullPointerException줄이기

예기치 않은 NPE를 피하려면 대부분의 프로그래머들은 필요한 곳에 null확인 코드를 추가하여 해결하려 할 것이다. 

```java
public String getCarInsuranceName(Person person) {
	if(person != null) {
		Car car = person.getCar();
		if(car != null) {
			Insurance insurance = car.getInsurance();
			if(insurance ! = null) {
				return insurance.getName();
			}
		}
	}
	return "Unknown";
}
```

위의 예시코드를 살펴보면 메서드 내에서 모든 변수가 Null인지 확인하기 위해 변수에 접근할 때마다 중첩된 if가 추가되면서 코드 들여쓰기 수준이 증가한다. (이와 같은 반복 패턴코드를 ‘깊은 의심’이라고 부른다)

이를 반복하다보면 코드이 구조가 엉망이 되고 가독성도 떨어진다. 

### null때문에 발생하는 문제

- 에러의 근원이다
    - NPE는 자바에서 가장 흔히 발생하는 에러이다
- 코드를 어지럽힌다
    - 때로는 중첩된 null확인 코드를 추가해야하므로null때문에 코드 가독성이 떨어진다
- 아무 의미가 없다
    - null은 아무 의미도 표현하지 않는다. 특히 정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다
- 자바 철학에 위배된다
    - 자바는 개발자로부터 모든 포인터를 숨겼다. 하지만 예외가 있는데 그것이 바로 Null포인터다
- 형식 시스템에 구멍을 만든다
    - null은 무형식적이며 정보를 포함하고 있지 않으므로 모든 참조 형식에 Null을 할당할 수 있다. 이런 식으로 null이 할당되기 시작하면서 시스템의 다른 부분으로 null이 퍼졌을 때 애초에 null이 어떤 의미로 사용되었는지 알 수 없다

# Optional클래스 소개

자바 8은 하스켈과 스칼라의 영향을 받아서 java.util.Optional<T>라는 새로운 클래스를 제공한다. 

Optional은 값이 있으면 값을 감싸고, 값이 없으면 Optional.empty메서드로 Optional을 반환한다. 

- Optional.empty
    - Optional의 특별한 싱글턴 인스턴스를 반환하는 정적 팩토리 메서드다.
    - null참조는 NPE를 발생시키지만 Optional.empty()는 Optional의 객체이므로 다양한 방식으로 활용할 수 있다.

**Null대신 Optional을 사용하면 이는 값이 없을 수 있음을 명시적으로 보여주는 반면,
null을 사용한다면 이것이 올바른 값인지 아니면 잘못된 값인지 판단할 아무 정보가 없다.**

<aside>
💡 옵셔널을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 적다 *-Effective Java*

</aside>

## Optional클래스의 메서드

### Optional 객체 생성

---

`of`: 값이 존재하면 값을 감싸는 Optional을 반환하고, 값이 null이면 `NullPointerException`을 발생함

`ofNullable`: 값이 존재하면 값을 감싸는 Optional을 반환하고, 값이 null이면 빈 Optional을 반환함 

`empty` : 빈 optional인스턴스 반환

### Optional 중간처리

---

`filter`: 값이 존재하며 Predicate와 일치하면 값을 포함하는 Optional을 반환하고, 값이 없거나 일치하지 않으면 빈 Optional을 반환함

`flapMap`: 값이 존재하면 인수로 제공된 함수를 적용한 결과 Optional을 반환하고, 값이 없으면 빈 Optional을 반환함  (이차원 Optional을 일차원 Optional로 평준화 시킨다)

`map`:값이 존재하면 제공된 매핑 함수를 적용함 

### Optional 종단처리

---

`ifPresent`: 값이 존재하면 지정된 Consumer를 실행하고 값이 없으면 아무일도 일어나지 않음

`isPresent`: 값이 존재하면 true를 반환하고 값이 없으면 false를 반환함

`get`: 값이 존재하면 Optional이 감싸고 있는 값을 반환하고, 값이 없으면 `NoSuchElementException` 이 발생함 

`orElse`: 값이 존재하면 값을 반환하고, 값이 없으면 기본값을 반환함 

`orElseGet`: 값이 존재하면 값을 반환하고, 값이 없으면 Supplier에서 제공하는 값을 반환함 

`orElseThrow`: 값이 존재하면 값을 반환하고, 값이 없으면 exception Supplier를 통해 예외를 발생시킴

### Java9 에서 추가된 Optional 메서드

---

`or`: 값이 존재하면 같은 Optional을 반환하고 값이 null이면 빈 Optional을 반환함

`ifPresentOrElse`: 값이 존재하면 지정된 Consumer를 실행하고 값이 없으면 람다함수를 실행

`stream`: 스트림객체로 전환

### Java10 에서 추가된 Optional 메서드

---

`orElseThrow`: 자바 8의 `orElseThrow`와 같지만 인자를 받지 않는다. 

<aside>
💡 **orElse(), orElseGet()의 차이점** 
: `orElse`메서드는 Optional객체가 비어있든 비어있지 않든 **반드시 실행**하는 반면, 
`orElseGet`메서드는 Optional객체가 **비어있으면 실행**한다.  
→ 따라서 기본값을 주고자 할 때에 기본값을 구하는 과정에 리소스가 든다면 `orElseGet`을 사용하자

</aside>

## Optional이 설계된 의도

> The primary use of Optional is as follows:
> 
> 
> `Optional` is intended to provide a **limited mechanism for library method return types** where there is a clear need to represent "no result," and where using null for that is overwhelmingly likely to cause errors.
> 
> A typical **code smell** is, instead of the code using method chaining to handle an `Optional` returned from some method, it **creates an `Optional` from something that's nullable, in order to chain methods and avoid conditionals.**
> 

: Optional은 Null을 반환하면 오류가 발생할 가능성이 매우 높은 경우에 결과없음을 명확히 드러내기 위해 **메서드의 반환 타입으로 사용**되도록 매우 제한적인 경우로 설계되었다. 

## Optional사용시 주의해야할 점

1. 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다 
    - 빈 Optional<List<T>>를 반환하기 보다는 빈 List<T>를 반환하는게 좋다
        - 빈 컨테이너를 그대로 반환하면 클라이언트에 옵셔널 처리 코드를 넣지 않아도 된다.
2. 박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 해야한다.
    - 박싱된 기본 타입을 담는 옵셔널은 기본 타입 자체보다 무거울 수밖에 없다. (값을 두겹이나 감싸기 때문) 
    따라서 int, long, double전용 옵셔널 클래스들을 활용하자
        - OptionalInt, OptionalLong, OptionalDouble
3. 옵셔널을 맵의 값으로 사용하면 안된다
    - 만약 그렇게 한다면 맵 안에 키가 없다는 사실을 나타내는 방법이 두가지가 된다. 
    (하나는 키 자체가 없는경우, 하나는 키는 있지만 속이 빈 옵셔널인 경우) → 쓸데없이 복잡성을 높임
    - 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는게 적절한 상황은 거의 없다.

### 그럼 언제 Optional을 사용해야 하는가?

`값을 반환하지 못할 가능성`이 있고, `호출할 때마다 반환값이 없을 가능성`을 염두에 둬야하는 메서드일 때 사용한다. 단, 옵셔널 반환에는 성능저하(엄연히 새로 할당하고 초기화 해야하는 객체이고 그 안에서 값을 꺼내려면 메서드를 호출해야하니 한 단계를 더 거치는 셈)가 뒤따르니, 성능에 민감한 메서드라면 Null을 반환하거나 예외를 던지는 편이 나을 수도 있다. (+옵셔널을 반환값 이외의 용도로 쓰는 경우는 매우 드물다) 

참고 :

모던자바 인 액션 책

이펙티브 자바 책 

[https://jdm.kr/blog/234](https://jdm.kr/blog/234)

[https://stackoverflow.com/questions/74355847/java-optional-sorting-a-nullable-list-stored-as-a-property-of-a-nullable-objec](https://stackoverflow.com/questions/74355847/java-optional-sorting-a-nullable-list-stored-as-a-property-of-a-nullable-objec)

[https://mangkyu.tistory.com/203](https://mangkyu.tistory.com/203)
