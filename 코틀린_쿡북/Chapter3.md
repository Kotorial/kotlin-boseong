## Chapter 3
### Recipe 3.6 나중 초기화를 위해 lateinit 사용하기
#### 문제
생성자에 속성 초기화를 위한 정보가 충분하지 않으면 해당 속성을 널 비허용 속성으로 만들고 싶다.
#### 해법
속성에 `lateinit` 변경자를 사용한다.

>이 레시피의 의존성 주입의 경우 유용하지만, 일반적으로 가능하다면 지연 평가 같은 대안을 먼저 고려하자

널 비허용으로 선언된 클래스 속성은 생성자에서 초기화되어야 한다.

하지만 상황에 따라 속성에 할당할 값이 생성 시점에 충분하지 않은 경우가 존재한다.
주로 의존성 주입 프레임워크, 유닛 테스트의 설정 메서드 안에서 발생한다.

```kotlin
class UnitTest {
    @Autowired
    lateinit var service: Service
    
    @Autowired
    lateinit var repository: Repository
}
```
- 코틀린 1.2부터는 최상위 속성, 지역변수에도 lateinit을 사용할 수 있다. 이전에는 클래스 몸체에서만 선언할 수 있었다.
- 사용자 정의 `getter`와 `setter`가 없는 `var`에만 사용 가능하다.
- 초기화되기 이전에 접근하면 `UninitializedPropertyAccessException` 예외가 발생한다.
- `isInitialized`를 통해 초기화 여부를 확인할 수 있다.

**`lateinit`과 `lazy`의 차이**
- `lazy`는 `val`에만, `lateinit`은 `var`에만 사용 가능
- `lateinit`은 속성에 접근 가능한 모든 곳에서 초기화 가능
```kotlin
lateinit var str
val len: String by lazy { str.length }
str = "hello world!"
print(str)
```

### Recipe 3.7 equals 재정의를 위해 안전 타입 변환, 레퍼런스 동등, 엘비스 사용하기
#### 문제
논리적으로 동등한 인스턴스인지를 확인하도록 클래스의 equals 메서드를 잘 구현하고 싶다.
#### 해법
레퍼런스 동등 연산자(`===`), 안전 타입 변환 함수(`as?`), 엘비스 연산자(`?:`)를 함께 사용한다.

자바에서는 `==`연산자가 코틀린의 `===`처럼 동작한다. 코틀린의 `==`연산자는 자동으로 `equals` 메서드를 호출한다.

equals 의 구현은 다음 성질들이 요구된다.
- 반사성(reflexive): null이 아닌 모든 참조값 x에 대해, `x.equals(x)`는 항상 true이다.
- 대칭성(symmetric): null이 아닌 모든 참조값 x, y에 대해 `x.equals(y)`와 `y.equals(x)`는 항상 같다.
- 추이성(transitive): null이 아닌 모든 참조값 x, y, z에 대해 `x.equals(y)`, `y.equals(z)`가 모두 true이면 `x.equals(z)`도 true이다.
- 일관성(consistent): 몇번을 호출해도 두 대상 객체의 내용이 변하지 않는다면, 결과는 항상 같아야 한다.

코틀린 표준 라이브러리 스타일
```kotlin
class Customer(val name: String) {
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        val otherCustomer = (other as? Customer) ?: return false
        return this.name == otherCustomer.name
    }
}
```
Intellij의 자동 생성되는 스타일
```kotlin
class Customer(val name: String) {
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (javaClass != other?.javaClass) return false
        other as Customer
        
        return this.name == other.name
    }
}
```
차이점
- Intellij 생성 코드는 타입 변환 전에 `javaClass`를 검사한다.
- as 연산자를 사용한다.
- `name`을 검사하기 위해 스마트 타입 캐스팅의 결과에 의존한다.
