# ReentrantLock

lock 을 사용하기 위해 쓰는 클래스로 이해된다.\
특징이 있다면 fair, nonFair 모드가 있다.

#### Fair

- 쓰레드가 경쟁 상태에서 특정 쓰레드가 점유를 못하는 상황을 안만들기 위해서 따로 계산해서 그런 상황을 안만드는 방법. 당연히 계산을 하는 비용때문에 성능은 nonFair 에 비해 좋지 않다.

```kotlin
private val lock = ReentrantLock(true)
```

#### NonFair

- 특정 쓰레드가 점유를 계속 못하는 상태가 발샐할 수 있지만, 따로 계산을 하지 않기 때문에 Fair 에 비해서 성능은 좋다

```kotlin
private val lock = ReentrantLock()
```

내가 궁금했던 핵심 부분만 살펴보자

lock 을 획득하는 과정과 release 하는 과정 2가지를 살펴보려 한다.
메서드로는 tryLock 과 release 2가지를 살펴보려 한다.

### tryLock

code

```kotlin
public boolean tryLock() {
    return sync.nonfairTryAcquire(1);
}

fun /*@@blobya@@*/nonfairTryAcquire(acquires:/*@@jgkkob@@*/Int): /*@@lawcdm@@*/Boolean {
    val current: /*@@kcalqc@@*/java.lang.Thread? = java.lang.Thread.currentThread()
    val c: /*@@jgkkob@@*/Int = getState()
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current)
            return true
        }
    } else if (current === getExclusiveOwnerThread()) {
        var nextc: /*@@jgkkob@@*/Int = c + acquires
        if (nextc < 0) // overflow
            throw java.lang.Error("Maximum lock count exceeded")
        setState(nextc)
        return true
    }
    return false
}
```

보면 현재 쓰레드와 상태를 가져와 0이면 진입하고 true 리턴 (락 획득 성공)\
만약 같은 쓰레드면 더 진입가능(락 획득 성공) 이 말은 같은 쓰레드에서는 .tryLock 을 여러개 할 수 있다로 해석된다.

여기서 핵심은 state 를 화긴해보면

```kotlin
import kotlin.concurrent.Volatile

protected final void setState (int newState) {
    state = newState;
}

@Volatile
private val state = 0
```

이런식으로 volatile 값을 셋팅하는 방식으로 되어있다.

### tryRelease

```kotlin
protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```

예상대로 setState 즉 state 값을 이용하여 lock 과 release 를 하는것을 알 수 있다.

우선 acquire 과 release 를 살펴보았고 추가로 더 필요한 내용이 있다면 업데이트 할 예정


