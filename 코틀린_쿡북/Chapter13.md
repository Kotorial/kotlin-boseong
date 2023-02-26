### Recipe 13.4 자바 스레드 풀에서 코루틴 실행하기
#### 문제
코루틴을 사용하는 사용자 정의 스레드 풀을 제공하고 싶다.
#### 해법
자바 `ExecutorService`의 `asCoroutineDispatcher` 함수를 사용한다.

코틀린의 라이브러리는 `ExecutorService`를 `ExeutorCoroutineDispatcher`로 변환해주는 `asCoroutineDispatcher` 함수를 추가했다.

스레드 풀 종료하기 - 수동
```kotlin
val pool = ExecutorService.newFixedThreadPool(10)
withContext(pool.asCoroutineDispatcher()) {
    // ...
}
pool.shutdown() // close가 아닌 shutdown 호출
```
코틀린에서 자바의 `try-with-resource`와 같은 기능을 사용하기 위해서는 `use`함수를 사용해야 함.

하지만 `close`가 아닌 `shutdown`을 호출해야 하기 때문에 `use` 함수를 사용할 수 없음.

따라서 `ExeutorCoroutineDispatcher`가 `Closeable`을 구현하도록 하여 해결

```kotlin
abstract class ExeutorCoroutineDispatcher : CoroutineDispatcher(), Closeable
```

> 자바에선 만들 때 부터 `AutoCloseableExecutorService`로 생성하면 된다.

### Recipe 13.5 코루틴 취소하기
#### 문제
코루틴 내의 비동기 처리를 취소하고 싶다.
#### 해법
`launch 빌더` 또는 `withTimeout`, `withTimeoutOrNull` 같은 함수가 리턴하는 `Job 레퍼런스`를 사용한다.

job을 통해 종료시키기
```kotlin
fun main() = runBlocking {
    val job = launch {
        // some operations..
    }
    job.cancel()
    job.join() // job.cancelAndJoin()
}
```

코루틴 실행에 `timeout` 적용하기
```kotlin
fun main() = runBlocking {
    withTimeout(1000L) {
        repeat(50) { i -> 
            println("job: I'm waiting $i...")
            delay(100L)
        }
    }
}
```
- 위 코드 실행 시 timeout이 발생한 시점에 `TimeoutCancellationException`이 발생한다.
- `try-catch`로 처리하거나 예외 대신 null을 리턴하는 `withTimeoutNull`을 사용한다.

### Recipe 13.6 코루틴 디버깅
#### 문제
코루틴의 실행 정보가 더 필요하다.
#### 해법
JVM에서 `-Dkotlinx.coroutines.debog` 플래그를 사용해서 실행한다.

- 디버그 모드는 실행된 모든 코루틴에 고유한 이름을 부여한다. 기본으로 `"@coroutine#" + 숫자`
- `Thread.currentThread().name`에 디버그 모드로 실행 시 위의 고유한 코루틴 이름이 추가된다.
  - ex. `DefaultDispatcher-worker-1 @coroutine#1`

코루틴에 직접 이름을 부여할 수도 있다.
```kotlin
coroutineScope {
    async(Dispatchers.IO + CoroutineName("async")) { // 연산자 오버로드
        // ... Thread.currentThread().name 출력
    }
}
```
- `Thread.currentThread().name` 실행 시 `DefaultDispatcher-worker-1 @async#1`가 출력 됨.