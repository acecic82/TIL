# FixedThreadPool

고정된 개수의 스레드를 사용. 고정된 스레드 수 이상으로 작업이 추가되면 대기열에 들어감

### 코드를 살펴보자

#### 우선 생성자

```kotlin
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```
corePoolSIze, maximumPoolSize
보아하니 corePoolSize 와 maximumPoolSize 를 동일하게 선언하는 방법인듯한다.
그리고 LinkedBlockingQueue 가 보이는데 이것의 역할은 workQueue 이다.

#### execute

```kotlin
int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
```

corePoolSize 보다 적은 쓰레드가 실행중이면 새로운 스레드로 command 를 실행한다.
addWorker 가 안되는 상황이면 밑 진행

이게 안되면 workerQueue 에 추가함

우선 offer 만 간단하게 살펴보고 넘어가자

```kotlin
final Node<E> node = new Node<E>(e);
        final ReentrantLock putLock = this.putLock;
        putLock.lock();
        try {
            if (count.get() == capacity)
                return false;
            enqueue(node);
            c = count.getAndIncrement();
            if (c + 1 < capacity)
                notFull.signal();
        } finally {
            putLock.unlock();
        }
```

e 는 커맨드이며 락 잡고 enqueue 한다.

언제 꺼내올까?

private Runnable getTask() 여기서 꺼내온다.

```kotlin

var wc: Int = ThreadPoolExecutor.workerCountOf(c)
// Are workers subject to culling?
var timed: Boolean = allowCoreThreadTimeOut || wc > corePoolSize

if ((wc > maximumPoolSize || (timed && timedOut))
    && (wc > 1 || workQueue.isEmpty())
) {
    if (compareAndDecrementWorkerCount(c)) return null
    continue
}

try {
    Runnable r = timed ?
    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
    workQueue.take();
    if (r != null)
        return r;
    timedOut = true;
} catch (InterruptedException retry) {
    timedOut = false;
}
```

workerCount 가 맥스쓰레드풀보다 작으면 꺼내와서 추가한다.
poll 이나 Take 를 하게 되는데 결국 dequeue 를 하는것이다.

특징
- 고정된 크기의 스레드 풀을 생성.
- 미리 지정한 수의 스레드로 작업을 처리.
- 풀 크기를 초과한 작업은 큐에 저장되어 순차적으로 처리.
- 스레드 수가 일정하므로 CPU 집약적 작업에 적합.