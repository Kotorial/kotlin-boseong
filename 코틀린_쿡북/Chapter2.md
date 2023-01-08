## Chapter 2
### Recipe 2.3 자바를 위한 메서드 중복
#### 문제
기본 파라미터를 가진 코틀린 함수가 있는데, 자바에서 각 파라미터의 값을 직접적으로 명시하지 않고 해당 코틀린 함수를 호출하고 싶다.
#### 해법
`@JvmOverloads` 애노테이션을 해당 함수에 추가한다.

```kotlin
fun addProduct(name: String, price: Double = 0.0, desc: String? = null)
```
위 함수를 코틀린에서는 아래와 같이 호출할 수 있다.
```kotlin
addProduct("Name", 5.0, "Desc")
addProduct("Name", 5.0)
addProduct("Name")
```
하지만 자바에서는 메서드 기본 인자가 없기 때문에 항상 모든 인자를 넣어야 한다.
자바에서도 기본 인자가 지원되는 코틀린처럼 호출하고 싶다면 해당 함수에 `@JvmOverloads`를 붙인다.
생성된 바이트코드를 살펴보면 오버로드된 함수가 추가되어있다.

생성자에도 마찬가지로 `@JvmOverloads`를 통해 사용 가능하다.
단, `constructor` 키워드를 반드시 붙여줘야 한다.
```kotlin
data class Product @JvmOverloads constructor( // constructor 키워드 필수
        val name: String,
        val price: Double = 0.0,
        val desc: String? = null
)
```
---
### Recipe 2.4 명시적으로 타입 변환하기
#### 문제
코틀린은 자동으로 기본 타입을 더 넓은 타입으로 승격하지 않는다.(`Int` -> `Long` 으로 자동 승격하지 않음)
#### 해법
더 작은 타입을 명시적으로 변환하려면 `toInt`, `toLong` 등 구체적인 변환 함수를 사용한다.

자바에서는 다음 코드처럼 `int`가 더 큰 타입인 `long`에 들어갈 때 자동으로 형변환 되는 것이 자연스럽다.
```java
int myInt = 5;
long myLong = aInt;
```

코틀린에서는 기본 타입을 직접적으로 제공하지 않는다. 개발자가 코드를 작성할 때는 클래스를 다룬다.
```kotlin
val intVar: Int = 3
val longVar: Long = intVar.toLong()
```
코틀린은 연산자 오버로딩(중복)을 지원하기 떄문에 아래 코드는 명시적 형변환이 필요하지 않다.
```kotlin
val longSum = 3L + intVar
```
+연산자는 Int의 값을 자동으로 long으로 변환하고 long 리터럴에 그 값을 더한다.
