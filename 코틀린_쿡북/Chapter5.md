## Chapter 5
### Recipe 5.3 컬렉션에서 읽기 전용 뷰 생성하기
#### 문제
변경 가능한 리스트, 세트, 맵이 있을 때 해당 컬렉션의 읽기 전용 버전을 생성하고 싶다.
#### 해법
toList, toSet, toMap 메소드를 사용해 새로운 읽기 전용 컬렉션을 생성하자.
기존 컬렉션을 바탕으로 읽기 전용 뷰를 만들려면 List, Set 또는 Map 타입의 변수에 기존 컬렉션을 할당한다.

```kotlin
val immutableNums: List<Int> = mutableNums.toList() // 1

val readOnlyNums: List<Int> = mutableNums // 2
```
mutable 컬렉션을 read only로 만드는 방법은 위와 같이 2가지 방법이 있다.
1. 대상 mutable 컬렉션을 복제하여 새로운 객체를 만든다.
2. 대상 컬렉션을 add 등의 변경 메서드가 없는 타입으로 감싼다. 따라서 변경이 일어나는 경우 read only도 변화가 반영된다.

>Map에 toList를 적용하면 Pair의 List가 반환된다.

```kotlin
public interface List<out E> : kotlin.collections.Collection<E>
public interface MutableList<E> : kotlin.collections.List<E>, kotlin.collections.MutableCollection<E>
```
MutableList의 상위 타입이 List이므로 2번의 방법이 가능하다. List 인터페이스에는 조작하는 add 등의 메서드가 포함되어 있지 않다.

### Recipe 5.4 컬렉션에서 맵 만들기
#### 문제
key 리스트가 있을 때 각각의 키와 생성한 값을 연관시켜서 맵을 만들고 싶다.
#### 해법
associateWith 함수에 각 키에 실행되는 람다를 제공해 사용한다.

```kotlin
val keys = 'a'..'f'
val map1 = keys.associate { it to it.toString().repeat(5) }
// val map = keys.associate { [key] to [value] }
val map2 = keys.associateWith { it.toString().repeat(5) }
val map3 = keys.associate { it.uppercase() to it.toString().repeat(5) }
```

1. map1은 associate 함수를 사용한다. `{ key to value }`형태로 key와 value를 설정할 수 있다.
2. 리스트의 값을 그대로 key로 사용하는 경우 `associateWith`로 더 간단하게 표현할 수 있다.
3. `associate`는 key에 변형이 필요한 경우 사용한다.

### Recipe 5.5 컬렉션이 빈 경우 기본값 리턴하기
#### 문제
컬렉션을 처리할 때 컬렉션의 모든 원소가 선택에서 제외되지만 기본 응답을 리턴하고 싶다.
#### 해법
컬렉션이나 문자열이 비어있는 경우에는 `ifEmpty`와 `ifBlank` 함수를 사용해 기본값을 리턴한다.

```kotlin
fun onSaleProducts_ifEmptyCollection(products: List<Product>) =
        products.filter { it.onSale } // onSale로 필터링
                .map { it.name } // name만 선택
                .ifEmpty { listOf("none") } // 만약 위의 결과가 비어있다면, listOf("none") 반환
                .joinToString(seperator = ", ") // list를 join

fun onSaleProducts_ifEmptyString(products: List<Product>) =
        products.filter { it.onSale } // onSale로 필터링
                .map { it.name } // name만 선택
                .joinToString(seperator = ", ") // list를 join
                .ifEmpty { "none" } // 만약 위의 결과가 비어있다면, "none" 반환
```
`ifEmpty`와 `ifBlank`의 차이
```kotlin
"   ".ifEmpty { "not empty" } // 완전히 비어있으면 true
"   ".ifBlank { "blank" } // 완전비 비어 있거나 모두 공백문자이면 true
```
