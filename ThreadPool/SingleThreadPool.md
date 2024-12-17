# SingleThreadPool

FixedThreadPool 과 매우 유사하여 생성자만 봐도 될것 같다.

```kotlin
new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>())
```

core 와 maximum 이 1인것이 특징이고 나머지는 fixed 와 똑같다.
기호에 따라 사용하면 된다.

특징
- 단일 스레드로 작업을 처리.
- 작업이 FIFO(First In, First Out) 순서로 실행됨.
- 단일 스레드 환경에서 작업의 순서가 중요할 때 적합.