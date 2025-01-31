# Netty 로 서버를 띄워보자

생각보다 쉽다.\
Netty 서버는 논블로킹 방식으로 클라이언트와 통신한다

1. 의존성 추가
2. 실행

## 의존성 추가

implementation("io.projectreactor.netty:reactor-netty") 추가

## 실행

Netty started on port 8080 (http)\
실행시 이 문구가 나오면 Netty 로 뜬것이다.


### 추가
Mono 와 Flux 형태로 return 하는 controller 를 만들어 테스트 해보자

```kotlin
@GetMapping("/mono/{name}")
    fun getMono(@PathVariable name: String): Mono<String> {
        println("Start Mono request $name")
        return Mono.just<String>("Hello, $name!")
            .delayElement(Duration.ofSeconds(1)) // 1초 지연
    }

@GetMapping("/flux")
fun getFluxMany(@RequestParam id: Int): Flux<String> {
    logger.info("Request received for ID: $id")

    return Flux.just("A", "B", "C") // 응답 3개 생성
        .delayElements(Duration.ofMillis(2000)) // 2초 간격으로 전송
        .doOnNext { logger.info("Response sent for ID: $id -> $it") } // 서버에서 로그 출력
}
```

나는 서버쪽에 Mono 형태와 Flux 형태로 return 을 간단하게 만들고 테스트 진행\
테스트를 위한 [클라이언트](WebClient.md)는 여기를 참조