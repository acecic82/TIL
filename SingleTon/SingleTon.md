# SingleTon Java & Kotlin

클래스의 인스턴스를 하나만 생성하고 전역적으로 접근하도록 하는 디자인 패턴

### Java 종류
- Eager Initialization
- Lazy Initialization
- Thread-Safe Singleton
- Double-Checked Locking
- Bill Pugh Singleton
- Enum Singleton

### Kotlin 종류
- Object Declaration
- Lazy Initialization
- Thread-Safe
- Double-Checked
- Static Inner Class(Bill pugh 방식 변형)
- Enum Singleton

### Eager Initialization
클래스 로드 시점에 인스턴스 생성. 사용되지 않을 경우에도 인스턴스를 생성하기 때문에 자원낭비 가능성이 있다.

```Java
public class Singleton {
    private static final Singleton instance = new Singleton();

    private Singleton() { }

    public static Singleton getInstance() {
        return instance;
    }
}
```
<br/>

### Lazy Initialization
필요할때까지 인스턴스를 생성하지 않는다. 멀티 스레드 환경에서는 동기화 처리가 없으면 문제가 발생할 수 있다.
```java
public class Singleton {
    private static Singleton instance;

    private Singleton() { }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
<br/>

### Thread-Safe Singleton
멀티 스레드에서도 안전하도록 syncronized 키워드 사용. 하지만 성능상 문제가 있음
```java
public class Singleton {
    private static Singleton instance;

    private Singleton() { }

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
<br/>

### Double-checked Locking
동기화 블록을 줄여 성능 문제를 개선한 방식 volatile 키워드를 사용해 인스턴스가 제대로 초기화되지 않는 문제를 방지
하지만, 문제가 발생할 수 있다. 얼핏보기엔 발생안할것 같지만 그리고 이럴 가능성은 매우 낮지만
객체가 할당되는 과정을 생각하면
1) 메모리 주소 공간을 할당한다
2) 그 주소에 객체를 할당한다.

1번과 2번 사이에 해당 로직이 돌다가 객체가 할당되지 전에 주소를 리턴하고 그걸 사용하면
문제가 생길 수 있지만 거의 그런 타이밍은 잘 나오지 않는다.
```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() { }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

```
<br/>

### Bill Pugh Singleton
정적 내부 클래스를 사용해 싱글톤 구현. 클래스 로딩 시점과 인스턴스 생성시점 분리. 성능문제와 멀티스레드 안정성 확보
```java
public class Singleton {
    private Singleton() { }

    private static class SingletonHelper {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
}

```
<br/>

### Enum Singleton
자바에서 enum은 자동으로 싱글톤 보장하며 serialize deserialize 에도 안전
```java
public enum Singleton {
    INSTANCE;

    public void doSomething() {
        // 메서드 구현
    }
}

```

Enum 은 왜 안전하지? 궁금해서 찾아봄
enum은 클래스의 특별한 형태로, JVM이 인스턴스를 단 하나만 생성하도록 보장한다고 한다.\
이를 통해 싱글톤 패턴의 구현이 간단하고 안전해지며, 추가적으로 직렬화와 역직렬화 및 리플렉션 공격에서도 안전한다.\
```java
public enum Singleton {
    INSTANCE;

    public void doSomething() {
        System.out.println("Doing something...");
    }
}

```
```java
public final class Singleton extends Enum<Singleton> {
    public static final Singleton INSTANCE = new Singleton();

    private Singleton() {
        super("INSTANCE", 0); // Enum의 이름과 ordinal 값을 전달
    }

    public static Singleton[] values() {
        return new Singleton[]{INSTANCE};
    }

    public static Singleton valueOf(String name) {
        if (name.equals("INSTANCE")) {
            return INSTANCE;
        }
        throw new IllegalArgumentException("No enum constant " + name);
    }
}

```
이렇게 바뀐다고 한다.

#### JVM 동작 과정
클래스 로드\
Singleton 클래스가 로드될 때 INSTANCE가 초기화됩니다. 이 초기화 과정은 JVM에 의해 관리되며 멀티스레드 환경에서도 안전합니다.

싱글톤 보장

JVM은 Singleton.INSTANCE를 단 한 번만 생성하고, 이를 모든 호출에서 동일하게 참조합니다.

직렬화/역직렬화 처리\
enum은 직렬화 시 객체의 이름(예: "INSTANCE")만 기록하고, 역직렬화 시 이름을 기반으로 기존 인스턴스를 반환합니다. 이 과정은 java.lang.Enum에 의해 자동 처리됩니다.

### 내용은 다른건 쓰고 비슷한건 코드만

### Object Declaration
kotlin 은 object 키워드를 사용해서 간단하게 싱글톤 구현 가능

```kotlin
object Singleton {
    fun doSomething() {
        println("This is a Singleton!")
    }
}
```
object 는 어떻게 돼있길래 이게 가능하지?\
클래스 로딩시 한 번만 초기화됨
jvm 클래스 로딩과정에서 클래스의 정적 멤버를 스레드 안전하게 초기화해서 object 인스턴스 생성도 안전이 보장
object는 내부적으로 정적 필드를 사용해서 단일 인스턴스를 관리

### Lazy Initialization
```kotlin
class Singleton private constructor() {
    companion object {
        val instance: Singleton by lazy { Singleton() }
    }
}
```

### Thread-Safe Initialization
```kotlin
class Singleton private constructor() {
    companion object {
        private var instance: Singleton? = null

        @Synchronized
        fun getInstance(): Singleton {
            if (instance == null) {
                instance = Singleton()
            }
            return instance!!
        }
    }
}
```

### Double-Checked Locking
```kotlin
class Singleton private constructor() {
    companion object {
        @Volatile
        private var instance: Singleton? = null

        fun getInstance(): Singleton {
            return instance ?: synchronized(this) {
                instance ?: Singleton().also { instance = it }
            }
        }
    }
}
```

### Static Inner Class (Java의 Bill Pugh 방식 변형)
```kotlin
class Singleton private constructor() {
    companion object {
        val instance by lazy { Holder.INSTANCE }
    }

    private object Holder {
        val INSTANCE = Singleton()
    }
}
```

### Enum Singleton
```kotlin
enum class Singleton {
    INSTANCE;

    fun doSomething() {
        println("This is an Enum Singleton!")
    }
}
```