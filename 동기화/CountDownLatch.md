# CountDownLatch

카운트 다운 방식으로 동작하는 동기화 도구

countDown() 으로 숫자를 줄일 수 있다.
await() 로 모두 0이 될 때까지 대기할 수 있다.

```kotlin
import java.util.concurrent.CountDownLatch
import kotlin.concurrent.thread

fun main() {
    val latch = CountDownLatch(3) // 카운트다운 초기값

    repeat(3) { id ->
        thread {
            println("Worker $id is working...")
            Thread.sleep(1000L) // 작업 시뮬레이션
            println("Worker $id finished.")
            latch.countDown() // 카운트 감소
        }
    }

    println("Main thread is waiting for workers...")
    latch.await() // 카운트가 0이 될 때까지 대기
    println("All workers finished. Main thread resumes.")
}
```

재사용이 불가능하다

### Dive Deep

왜 재사용이 불가능할까? 내부를 봐야한다.