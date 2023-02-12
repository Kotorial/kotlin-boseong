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
