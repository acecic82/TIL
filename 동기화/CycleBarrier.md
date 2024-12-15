# CycleBarrier

모든 쓰레드가 특정 지점에 도달할 때까지 대기
(await 가 있다)\
쓰레드가 다 끝났을 때의 동작을 정의한다.

```kotlin
import java.util.concurrent.CyclicBarrier
import kotlin.concurrent.thread

fun main() {
    val barrier = CyclicBarrier(5) { // 동기화 지점: 5개의 스레드가 모이면 실행
        println("All threads reached the barrier. Proceeding...")
    }

    val threads = mutableListOf<Thread>() // 스레드 리스트

    repeat(5) { id ->
        val worker = thread {
            println("Worker $id is performing some work...")
            Thread.sleep((id + 1) * 1000L) // 작업 시뮬레이션
            println("Worker $id is waiting at the barrier.")
            barrier.await() // Barrier 대기
            println("Worker $id passed the barrier.")
        }
        threads.add(worker) // 스레드 리스트에 추가
    }

    // 모든 스레드가 종료될 때까지 메인 스레드 대기
    threads.forEach { it.join() }

    println("All workers completed.")
}
```
사용 후 재사용이 가능하다

### Dive Deep
