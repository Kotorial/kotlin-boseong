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

### Recipe 8.3 값이 null이 될 수 없게 만들기
#### 문제
처음 접근이 일어나기 전에 값이 초기화되지 않았다면 예외를 던지고 싶다.
#### 해법
notNull 함수를 이용해 값이 설정되지 않았다면 예외를 던지는 대리자를 제공한다.

```kotlin
var hi: String by Delegates.notNull() // null인 상태에서 사용하면 IllegalStateException 발생

public fun <T : Any> notNull(): ReadWriteProperty<Any?, T> = NotNullVar()

private class NotNullVar<T : Any>() : ReadWriteProperty<Any?, T> {
    private var value: T? = null

    public override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return value ?: throw IllegalStateException("Property ${property.name} should be initialized before get.")
    }

    public override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        this.value = value
    }
}
```
내부에서 NotNullVar 클래스를 사용한다. 그리고 NotNullVar는 아래와 같이 동작한다.
- `getValue`에서 value가 null이면 IllegalStateException를 발생시킨다.
- `setValue`는 value를 그대로 설정한다.

### Recipe 8.4 observable과 vetoable대리자 사용하기
#### 문제
속성의 변경을 가로채서, 필요에 따라 변경을 거부하고 싶다.
#### 해법
변경 감지에는 `observable` 함수를 사용하고, 변경의 여부를 결정할 때는 `vetoable` 함수와 람다를 사용하자.

```kotlin
 var watched: Int by Delegates.observable(1) { property, oldValue, newValue -> 
     println("${prop.name} changed from $oldValue to $newValue")
 }
 
 var checked: Int by Delegates.vetoable(0) { property, oldValue, newValue ->
    new >= 0
}
```
`observable`: 이 변수가 변하는 경우 받은 함수(onChanged)를 실행
`vetoable`: 이 변수가 변경할 때 받은 함수가 true를 반환해야 값을 변경함. false 경우는 무시

```kotlin
public inline fun <T> observable(initialValue: T, crossinline onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Unit):
        ReadWriteProperty<Any?, T> =
        object : ObservableProperty<T>(initialValue) {
            override fun afterChange(property: KProperty<*>, oldValue: T, newValue: T) = onChange(property, oldValue, newValue)
        }

public inline fun <T> vetoable(initialValue: T, crossinline onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Boolean):
        ReadWriteProperty<Any?, T> =
        object : ObservableProperty<T>(initialValue) {
            override fun beforeChange(property: KProperty<*>, oldValue: T, newValue: T): Boolean = onChange(property, oldValue, newValue)
        }

public abstract class ObservableProperty<V>(initialValue: V) : ReadWriteProperty<Any?, V> {
    private var value = initialValue
    
    protected open fun beforeChange(property: KProperty<*>, oldValue: V, newValue: V): Boolean = true
    protected open fun afterChange(property: KProperty<*>, oldValue: V, newValue: V): Unit {}

    public override fun getValue(thisRef: Any?, property: KProperty<*>): V {
        return value
    }

    public override fun setValue(thisRef: Any?, property: KProperty<*>, value: V) {
        val oldValue = this.value
        if (!beforeChange(property, oldValue, value)) {
            return
        }
        this.value = value
        afterChange(property, oldValue, value)
    }
}
```

- `observable`과 `vetoable` 모두 ObservableProperty 타입을 리턴한다.
- ObservableProperty는 ReadWriteProperty를 구현하므로 getValue와 setValue를 구현한다.
- ObservableProperty는 beforeChange, afterChange 메서드를 가진다.
- setValue 함수에서 값을 변경하기 전, beforeChange가 false인 경우 값을 변경하지 않는다.
- setValue 함수에서 값을 변경한 후 afterChange 함수를 호출한다.
- `observable`과 `vetoable` 각각 afterChange와 beforeChange를 활용하여 구현한다.
