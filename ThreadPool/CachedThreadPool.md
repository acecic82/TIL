# CachedThreadPool

필요할 때마다 스레드 생성, 사용하지 않으면 제거.

```kotlin
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

core 가 0 max 는 MAX 로 지정해둔다. 그리고 workQueue 는 SynchronousQueue 를 사용한다.
60초를 타임 리밋으로 둔다.
FixedThread 에서 설명이 있지만 살펴보자 같은 코드지만 core 가 0 이기 때문에 동작이 다를듯하다.

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
우선 corePoolSize 가 0이기 때문에 첫 번째 if 문에 못타게 된다. offer 를 살펴보자

offer
```kotlin
public boolean offer(E e) {
        if (e == null) throw new NullPointerException();
        return transferer.transfer(e, true, 0) != null;
    }
```

transfer 내용을 살펴보면 이런 설명이 써있다.

* 1. If queue apparently empty or holding same-mode nodes,
*    try to add node to queue of waiters, wait to be
*    fulfilled (or cancelled) and return matching item.
*
* 2. If queue apparently contains waiting items, and this
*    call is of complementary mode, try to fulfill by CAS'ing
*    item field of waiting node and dequeuing it, and then
*    returning matching item.

이런식으로 돌아간다고 한다.

take 할때는 그럼 어떨까?

```kotlin
public E take() throws InterruptedException {
        E e = transferer.transfer(null, false, 0);
        if (e != null)
            return e;
        Thread.interrupted();
        throw new InterruptedException();
    }
```

특이한 것은 똑같은 transfer 를 사용하는데 첫번째 인자가 null 이다.
null 로 들어가면 dequeue 하게 된다.

특징
필요한 만큼 스레드를 생성하여 작업을 처리.
기존 스레드가 사용되지 않을 경우 일정 시간이 지나면 제거.
작업이 많아질 경우 무제한으로 스레드 생성 가능.(인트맥스)
작업 시간이 짧고, 작업 빈도가 높은 경우 적합.