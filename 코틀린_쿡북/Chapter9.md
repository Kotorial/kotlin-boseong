### Recipe 9.3 기본 인자와 함께 도움 함수 사용하기
#### 문제
테스트 객체를 빠르게 생성하고 싶다.
#### 해법
기본 인자를 가진 `Helper 함수`를 제공한다. `copy` 함수를 사용하거나 필요하지도 않은 생성자 기본 인자를 명시하지 않는다.

```kotlin
data class Book(
        val isbn: String,
        val title: String,
        val author: String,
        val published: LocalDate
)
```
Book 클래스의 팩토리 함수
```kotlin
fun createBook(
        isbn: String = "12345",
        title: String = "kotlin cookbook",
        author: String = "Ken Kousen",
        published: LocalDate = LocalDate.parse("2023-02-13")
) = Book(isbn, title, author, published)
```
- 이 방법의 핵심은 원본 클래스에 영향을 주지 않는다는 것이다.
- `copy` 함수를 사용할 수도 있지만, 특히 중첩 구조에서 가독성이 떨어진다.

### Recipe 9.4 여러 데이터에 JUnit 5 테스트 반복하기
#### 문제
데이터 값 세트를 제공해서 JUnit 5 테스트를 실행하고 싶다.
#### 해법
JUnit 5의 `@ParameterizedTest`와 동적 테스트를 사용한다.

명시적으로 호출
```kotlin
@Test
fun `Fibonacci numbers(explicit)`() {
    assertAll(
            { assertThat(fibinacci(4), `is`(3)) },
            { assertThat(fibinacci(9), `is`(34)) }
    )
}
```
Csv 소스로 `@ParameterizedTest`
```kotlin
@ParameterizedTest
@CsvSource("1, 1", "2, 1", "3, 2", "4, 3", "5, 5")
fun `first 5 fibonacci numbers(csv)`(n: Int, fib: Int) = assertThat(fibonacci(n), `is`(fib))
```
JUnit은 여러 입력 인자를 결합시킬 수 있는 Arguments.of 팩토리 메서드를 제공한다.

팩토리 메소드를 사용해 테스트 데이터를 생성할 수 있다.
- 팩토리 메소드는 어떠한 인자도 받을 수 없다.
- `@TestInstance(Lifecycle.PER_CLASS)`가 명시되지 않은 경우 팩토리 메소드는 반드시 `static`으로 선언되어야 한다.
- 리턴 값은 JUnit이 순회할 수 있는 `stream`, `collection`, `iterable`, `iterator`, `array` 같은 타입이어야 한다.

`Lifecycle.PER_CLASS`가 명시된 경우
```kotlin
// 팩토리 메소드
private fun fibNumbers() = listOf(
        Arguments.of(1, 1),
        Arguments.of(2, 1),
        Arguments.of(3, 2),
        Arguments.of(4, 3),
        Arguments.of(5, 5)
)

@ParameterizedTest(name = "fibonacci({0}) == {1}")
@MethodSource("fibNumbers")
fun `first 5 fibonacci numbers(instance numbers)`(n: Int, fib: Int) = assertThat(fibonacci(n), `is`(fib))
```
테스트 수명 주기가 기본 옵션(`Lifecycle.PER_METHOD`)인 경우
```kotlin
companion object {
    @JvmStatic // 팩토리 메소드 소스를 static으로 간주하므로 추가
    fun fibNumbers() = listOf(
            Arguments.of(1, 1),
            Arguments.of(2, 1),
            Arguments.of(3, 2),
            Arguments.of(4, 3),
            Arguments.of(5, 5)
    )    
}

@ParameterizedTest(name = "fibonacci({0}) == {1}")
@MethodSource("fibNumbers")
fun `first 5 fibonacci numbers(instance numbers)`(n: Int, fib: Int) = assertThat(fibonacci(n), `is`(fib))
```