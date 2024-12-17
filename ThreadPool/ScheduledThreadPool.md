# ScheduledThreadPool

지정된 간격으로 반복작업을 실행하거나 특정 시간에 작업을 실행한다는데 살펴보자

생성자를 살펴보자
```kotlin
super(corePoolSize, Integer.MAX_VALUE,
              DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue());
```

어짜피 ThreadPoolExecutor 를 쓰는건 동일하고 다른 부분은 workQueue 부분인것 같다.
처음 보는 WorkQueue 가 보인다. DelayedWorkQueue 를 살펴보자

보다보니 execute 에서 offer 하는것과 getTask 에서 poll or take 하는 부분이 핵심같아
같 workQueue 의 해당 메서드를 보면 될것 같다.

우선 offer

```kotlin
lock.lock();
            try {
                int i = size;
                if (i >= queue.length)
                    grow();
                size = i + 1;
                if (i == 0) {
                    queue[0] = e;
                    setIndex(e, 0);
                } else {
                    siftUp(i, e);
                }
                if (queue[0] == e) {
                    leader = null;
                    available.signal();
                }
            } finally {
                lock.unlock();
            }
```

락 잡고 해제하는 부분인데 size 가 없는 경우는 0번에 runnable 위치시키고 그 이외엔 siftUp 하게 된다.
siftUp 도 살펴보자

```kotlin
private void siftUp(int k, RunnableScheduledFuture<?> key) {
            while (k > 0) {
                int parent = (k - 1) >>> 1;
                RunnableScheduledFuture<?> e = queue[parent];
                if (key.compareTo(e) >= 0)
                    break;
                queue[k] = e;
                setIndex(e, k);
                k = parent;
            }
            queue[k] = key;
            setIndex(key, k);
        }
```

트리형태인것 같다. 그럼 여기서 중요하게 볼것은 key가 주체가 되는 compareTo 인데
어떤식으로 되어있을까?

```kotlin
public int compareTo(Delayed o) {
            return this.future.compareTo(o);
        }
```

parameter 가 딜레이이다.즉 딜레이를 기준으로 compare 을 한다.

정리해보자면 tree 형태로 되어있는 큐에서 parent 를 찾아가면 Delay 를 비교해서 셋팅한다.

그럼 이제 이걸 어떻게 꺼내나 살펴보자

take or poll 형태의 메서드

```kotlin
RunnableScheduledFuture<?> first = queue[0];
                    if (first == null)
                        available.await();
                    else {
                        long delay = first.getDelay(NANOSECONDS);
                        if (delay <= 0L)
                            return finishPoll(first);
                        first = null; // don't retain ref while waiting
                        if (leader != null)
                            available.await();
                        else {
                            Thread thisThread = Thread.currentThread();
                            leader = thisThread;
                            try {
                                available.awaitNanos(delay);
                            } finally {
                                if (leader == thisThread)
                                    leader = null;
                            }
                        }
                    }
```
좀 길지만 핵심만 보자면 queue 0 번째에서 가져와서 딜레이를 체크한다.
딜레이를 보고 0보다 작거나 같으면 finishPoll 을 호출하고 리턴한다.

특징
- 반복 작업 또는 지연된 작업을 처리하기 위한 스레드 풀.
- 스케줄링 기능을 제공하며, 작업을 주기적으로 실행.
- 지정된 간격으로 반복 작업을 실행하거나, 특정 시간에 작업을 실행.
