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