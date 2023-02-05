## Chapter 8
### Recipe 8.2 lazy 대리자 사용하기
#### 문제
어떤 속성이 필요할 때까지 해당 속성의 초기화를 지연시키고 싶다.
#### 해법
코틀린 표준 라이브러리의 lazy 대리자를 사용하자.

```kotlin
val a = Int by lazy {
    3
}
fun <T> lazy(initializer: () -> T): Lazy<T>
fun <T> lazy(mode: LazyThreadSafetyMode, initializer: () -> T): Lazy<T>
fun <T> lazy(lock: Any?, initializer: () -> T): Lazy<T>
```
- SYNCHRONIZED

    하나의 스레드만 Lazy 인스턴스를 초기화할 수 있게 락을 사용
- PUBLICATION

    초기화 함수가 여러번 호출될 수 있지만, 첫 번째 리턴 값만 사용됨
- NONE

    락을 사용하지 않음

1. 기본. 스스로 동기화
   - 동기화 방법으로 SYNCHRONIZED 사용
2. 동기화 방법을 명시
3. 동기화 락을 위해 제공된 객체를 사용

lazy의 구현체
- mode를 설정하지 않으면 기본으로 SYNCHRONIZED 사용
- lock에 해당하는 객체가 주어지지 않으면, 스스로를 사용
```kotlin
public fun <T> lazy(initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer) // 1의 예시

@JvmVersion
private class SynchronizedLazyImpl<out T>(initializer: () -> T, lock: Any? = null) : Lazy<T>, Serializable {
    private var initializer: (() -> T)? = initializer
    @Volatile private var _value: Any? = UNINITIALIZED_VALUE
    // final field is required to enable safe publication of constructed instance
    private val lock = lock ?: this

    override val value: T
        get() {
            val _v1 = _value
            if (_v1 !== UNINITIALIZED_VALUE) {
                @Suppress("UNCHECKED_CAST")
                return _v1 as T
            }

            return synchronized(lock) {
                val _v2 = _value
                if (_v2 !== UNINITIALIZED_VALUE) {
                    @Suppress("UNCHECKED_CAST") (_v2 as T)
                }
                else {
                    val typedValue = initializer!!()
                    _value = typedValue
                    initializer = null
                    typedValue
                }
            }
        }
}
```