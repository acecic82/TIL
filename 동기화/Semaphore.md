# Semaphore

동시에 이용할 수 있는 자원의 수를 한정해두고 그만큼만 사용하도록 하는 방법

### 주요 키워드
- acquire
- release

자원이 사용이 가능할 때 acquire 로 획득하고 다 쓰면 release 를 하여 반환해줘야 다른 쓰레드에서 사용이 가능하다

```kotlin
import java.util.concurrent.Semaphore
import kotlin.concurrent.thread

fun main() {
    val semaphore = Semaphore(2) // 동시에 2개의 작업 허용
    val threads = mutableListOf<Thread>() // 스레드 리스트

    // 5개의 작업 생성
    repeat(5) { id ->
        val worker = thread {
            println("Worker $id is waiting for a permit...")
            semaphore.acquire() // 허가 요청
            println("Worker $id got a permit. Working...")
            Thread.sleep(2000L) // 작업 시뮬레이션
            println("Worker $id is releasing a permit.")
            semaphore.release() // 허가 반환
        }
        threads.add(worker) // 생성된 스레드 리스트에 추가
    }

    // 모든 스레드가 끝날 때까지 대기
    threads.forEach { it.join() }
    println("All workers completed.")
}

```
그리고 사용 후 재사용이 가능하다.

### Deep Dive