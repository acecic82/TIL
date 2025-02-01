# 테스트용 WebClient

netty 서버에서 Mono 와 Flux 를 return 하는 서버를 만들고 클라이언트를 간단하게 만들어 테스트를 해보자

크게 3 부분으로 되어있는데 난 2 부분만 사용하였다. 하지만, 참고를 위해 3개를 다 써두자

### Provider, HttpClient, WebClient

#### Provider

```kotlin
val provider = ConnectionProvider.builder("custom")
            .maxConnections(50) // 최대 Connection 수 조정
            .build()
```

여러 옵션을 설정할 수 있지만 maxConnection 만 설정하였다. timeout 등을 설정할 수 있다.

#### HttpClient

```kotlin
val httpClient = HttpClient.create()
            .doOnConnected { conn ->
                conn.addHandlerLast(object : ChannelInboundHandlerAdapter() {
                    override fun channelRead(ctx: ChannelHandlerContext, msg: Any) {
                        if (msg is HttpContent) {
                            val data = msg.content().toString(io.netty.util.CharsetUtil.UTF_8)
                            logger.info("Data received in buffer: $data")
                        }
                        ctx.fireChannelRead(msg) // 다음 핸들러로 전달
                    }
                })
            }
```

.doOnConnect 는 딱히 필요 없지만 나는 필요해서 추가하였다.\
flux 의 동작을 더 잘 이해하기 위해서 추가하였는데 평소엔 필요없을듯 하다.

#### WebClient
```kotlin
val webClient = WebClient.builder()
            .clientConnector(ReactorClientHttpConnector(httpClient))
            .codecs { configurer ->
                configurer.defaultCodecs().maxInMemorySize(10) // WebClient 의 버퍼 크기 조정
            }
            .baseUrl("http://localhost:8080")
            .build()
```

마찬가지로 maxInMemorySize 를 설정한 필요는 없지만 확인을 위해서 추가하였다.
Buffer 사이즈 테스트를 위해서 추가한 것으로 보통의 경우엔 필요가 없을것이다.

```kotlin
.doOnNext { response ->
                        logger.info("Response receive for $it, Response : $response Thread : ${Thread.currentThread()} time : ${System.currentTimeMillis()}")
                    }
```
doOnNext 를 통해서 Flux 의 경우 데이터를 줄 떄 마다 어떤 데이터가 오는지 확인하려 했다. 하지만, A,B,C 를 2초 간격으로 보내도 6초후에 ABC 가 한 번에 오는것을 확인했다.
\
Response receive for 7, Response : ABC

이런식으로 오는것을 확인하였다. 의도했던것과 달라 왜 그런지 조사를 해보니 버퍼가 있고 그 버퍼 크기가 다 차면 doOnNext 쪽으로 빠지는것이었다.

curl 로 날려보니 2초마다 응답이 오는것을 확인하고 서버는 문제가 없다고 판단. 클라이언트쪽을 수정하였다.

```kotlin
override fun channelRead(ctx: ChannelHandlerContext, msg: Any) {
                        if (msg is HttpContent) {
                            val data = msg.content().toString(io.netty.util.CharsetUtil.UTF_8)
                            logger.info("Data received in buffer: $data")
                        }
                        ctx.fireChannelRead(msg) // 다음 핸들러로 전달
                    }
```

그래서 추가한것이 httpClient 의 connect 부분에 channelRead 쪽에 메세지를 볼 수 있도록 추가했다. 결과는 예상한 대로 나왔다.\
Data received in buffer: A \
어딘가 이름이 channel 인것으로 봐선 pubsub 이 생각나는 부분이었다.

### 결론

WebClient 로 flux 를 수신할 시 doOnNext 에는 netty server 가 응답을 줄 때마다 동작하지 않을 수 있다. 버퍼 사이즈에 따라 달라보일 수 있지만, \
실제 들어오는 데이터는 서버에서 날려주는 데이터를 즉시 수신한다.
