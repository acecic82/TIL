# WorkStealingPool

흔히 알고 있는 forkJoinPool 이라고 생각하면 편하다
이건 잘 몰라서 일단 특징을 보고 가자

ForkJoinPool의 특징
<br/>
- WorkStealingPool은 Java 8에서 도입된 ForkJoinPool 기반의 스레드 풀입니다. Kotlin에서도 Java의 WorkStealingPool을 동일한 방식으로 사용할 수 있습니다.

- WorkStealingPool은 병렬 작업을 효율적으로 수행하기 위해 **작업 분할(fork)**과 작업 도난(work-stealing) 전략을 사용하며, 특히 비균등한 작업 부하를 처리하는 데 최적화되어 있습니다.

#### WorkStealingPool의 주요 개념

#####  Fork/Join 패러다임
<br/>
작업 분할(Fork): 큰 작업을 작은 작업으로 나누어 병렬로 처리.

작업 병합(Join): 분할된 작은 작업의 결과를 병합하여 최종 결과 생성.

##### Work-Stealing 기법
스레드가 자신의 작업 큐가 비어 있을 때, 다른 스레드의 작업 큐에서 남은 작업을 가져옴.

작업은 각 스레드의 **작업 큐(Deque)**에 저장되고, 대기 중인 작업을 **꼬리(tail)**에서 가져와 실행.

다른 스레드의 큐에서 작업을 훔칠 때는 **머리(head)**에서 가져옴.

##### WorkStealingPool 특징
**비균등한 작업 분배:**

작업이 고르게 분배되지 않아도 유휴 스레드가 다른 스레드의 작업을 훔쳐가므로 작업 부하가 균형을 이룸.

**ForkJoinPool 기반:**

ForkJoinPool의 모든 기능을 상속하며, 멀티스레드 환경에서 효율적으로 동작.

**병렬성 제어:**
스레드 풀 크기는 기본적으로 **Runtime.getRuntime().availableProcessors()**에 비례.

예시 코드

```kotlin
import java.util.concurrent.Executors
import java.util.concurrent.RecursiveAction

fun main() {
    val array = intArrayOf(9, 3, 7, 1, 6, 2, 8, 5, 4)
    val pool = Executors.newWorkStealingPool() // WorkStealingPool 생성

    pool.invoke(SortTask(array, 0, array.size))
    println("Sorted array: ${array.joinToString(", ")}")

    pool.shutdown() // 풀 종료
}

class SortTask(
    private val array: IntArray,
    private val start: Int,
    private val end: Int
) : RecursiveAction() {

    companion object {
        const val THRESHOLD = 2 // 작업 분할 기준
    }

    override fun compute() {
        if (end - start <= THRESHOLD) {
            array.sort(start, end) // 작은 작업은 직접 정렬
        } else {
            val mid = (start + end) / 2
            val leftTask = SortTask(array, start, mid)
            val rightTask = SortTask(array, mid, end)

            invokeAll(leftTask, rightTask) // 병렬 작업 실행
        }
    }
}
```