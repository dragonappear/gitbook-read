# 생성자 대신 정적 팩토리 메서드를 고려해라

### 정적 팩토리 메서드란

클라이언트에게 public 생성자 대신(혹은 함께) 정적 팩토리 메서드로 제공하여 객체를 생성할 수 있게 한다.

<br>

### 장점 1. 이름을 가질 수 있다

생성자에 넘기는 매개변수와 생성자 자체로는 객체의 특성을 제대로 이해하기 어렵다. 

정적 팩토리 메서드는 이름만 잘 지으면 반환될 객체의 특성을 보다 쉽게 묘사할 수 있다.

```Java
new BigInteger(int,int,random); 

BigInteger.probablePrime(int,int,random); // '값이 소수인 `BigInteger`를 반환한다' 의미를 잘 설명한다
```

위 예시에서 객체를 생성하는 두가지 방법(생성자, 정적 팩토리 메서드) 중 정적 팩토리 메서드가 '값이 소수인 `BigInteger`를 반환한다' 의미를 더 잘 설명한다.
이처럼 `new` 보다 메서드 이름을 통한 의미 전달이 더 좋을 때 정적 팩토리 메서드를 사용하면 된다.

<br>

```Java
public Class Employee {
  private String name;
  private String address;
  
    private Employee(String name) {
        this.name = name;
    }
    
    private Employee(String address) {
        this.address = address;
    }
    
    public static Employee createWithName(String name) {
        return new Employee(name);
    }
    
    public static Employee createWithAddress(String address) {
        return new Employee(address);
    }
}
```

**한 클래스에 시그니처가 같은 생성자가 여러개 필요할 것 같으면 생성자를 정적 팩토리 메서드로 바꾸고 각각의 차이를 드러내는 이름으로 지어주면 된다.**

<br>

### 장점 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다

인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 방식으로 **불필요한 객체 생성을 피할 수 있다.** 
객체 생성은 비용이 크기 때문에 같은 객체가 자주 요청되는 상황이라면 성능을 상당히 끌어올려 준다.

정적 펙토리 메서드는 인스턴스를 통제함으로써 **싱글톤**으로 만들거나, **인스턴스화 불가로** 클래스를 만들거나, **불변값 클래스에서 동치인 인스턴스가 단 하나임을 보장**할 수 있다.

```Java
public final class Boolean {
    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);

    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
```

`valueOf()`는 객체를 새로 생성하지 않고 미리 만들어둔 `Boolean` 객체를 반환하여 인스턴스화 불가 클래스를 만든다.


```Java
class Color {
    private final int red;
    private final int green;
    private final int blue;

    private Color(int red, int green, int blue) {
        this.red = red;
        this.green = green;
        this.blue = blue;
    }

    // 정적 팩토리 메서드
    public static Color valueOf(int red, int green, int blue) {
        // 캐싱된 인스턴스가 있는지 확인
        return Cache.getColor(red, green, blue);
    }

    @Override
    public boolean equals(Object obj) {
        if (obj instanceof Color) {
            Color other = (Color) obj;
            return this.red == other.red && this.green == other.green && this.blue == other.blue;
        }
        return false;
    }

    @Override
    public int hashCode() {
        return Objects.hash(red, green, blue);
    }

    // 색상 객체를 캐시하는 내부 클래스
    private static class Cache {
        private static final Map<List<Integer>, Color> colorCache = new HashMap<>();

        static Color getColor(int red, int green, int blue) {
            List<Integer> key = Arrays.asList(red, green, blue);
            return colorCache.computeIfAbsent(key, k -> new Color(red, green, blue));
        }
    }
}

    @Test
    void 동치성_테스트() {
        Color color1 = Color.valueOf(255, 0, 0);
        Color color2 = Color.valueOf(255, 0, 0);

        assertThat(color1).isEqualTo(color2);
    }

```

`valueOf()` 메서드는 새로운 객체를 생성하지 않고 캐싱된 객체를 반환하여 동치성을 보장한다.

<br>

#### Flyweight 패턴

객체의 효율적인 공유를 통해 메모리 사용을 최적화하는 디자인 패턴

- Flyweight: 재사용될 수 있는 가벼운 객체
- Flyweight Factory: Flyweight 객체를 관리하고 저장하는 팩토리

```Java
// Flyweight 인터페이스
public interface Flyweight {
    void doOperation(String extrinsicState);
}

// 구체적인 Flyweight 클래스
public class ConcreteFlyweight implements Flyweight {
    private String intrinsicState;

    public ConcreteFlyweight(String intrinsicState) {
        this.intrinsicState = intrinsicState;
    }

    @Override
    public void doOperation(String extrinsicState) {
        System.out.println("Intrinsic State = " + intrinsicState + ", Extrinsic State = " + extrinsicState);
    }
}

// Flyweight Factory 클래스
public class FlyweightFactory {
    private static Map<String, Flyweight> flyweights = new HashMap<>();

    public static Flyweight getFlyweight(String key) {
        if (!flyweights.containsKey(key)) {
            flyweights.put(key, new ConcreteFlyweight(key));
        }
        return flyweights.get(key);
    }
}

class FlyweightPatternTest {

    @Test
    void 캐싱_테스트() {
        FlyweightFactory factory = new FlyweightFactory();

        Flyweight flyweightA = factory.getFlyweight("A");
        Flyweight flyweightB = factory.getFlyweight("B");

        Flyweight flyweightA2 = factory.getFlyweight("A"); 

        assertThat(flyweightA).isEqualTo(flyweightA2);

        flyweightA.doOperation("Operation 1");
        flyweightB.doOperation("Operation 2");
        flyweightA2.doOperation("Operation 3");
    }
}
```

<br>

### 장점 3. 반환 타입의 하위 타입 객체를 반환할 수 있다

구현 클래스를 공개하지 않고도 객체를 반환할 수 있어서 API를 유연하고, 작게 만들 수 있다.

![](https://github.com/dragonappear/read/assets/89398909/996ec364-39cb-4a17-9709-2360570624f2)

`Collections`의 `of()` 메서드를 보면 정적 팩토리 메서드를 사용하여 구현 클래스를 숨겼음을 확인할 수 있다.

정적 팩토리 메서드에서 하위 클래스를 반홤으로써, **클라이언트는 반환된 객체를 인터페이스만으로 다루게 하고, 구체 클래스에 대한 의존성을 감출 수 있다.**

<br>

### 장점 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다

클라이언트는 팩토리가 반환하는 객체가 어느 클래스의 인스턴스인지 알 필요 없이 인스턴스를 사용할 수 있다.

```Java
// Grade 인터페이스
interface Grade {
    String getGrade();
}

// A 등급을 나타내는 클래스
class GradeA implements Grade {
    @Override
    public String getGrade() {
        return "A";
    }
}

// B 등급을 나타내는 클래스
class GradeB implements Grade {
    @Override
    public String getGrade() {
        return "B";
    }
}

// Grade 객체를 생성하는 팩토리 클래스
class GradeFactory {

    public static Grade getGrade(int score) {
        if (score >= 90) {
            return new GradeA();
        }

        return new GradeB();
    }
}

    @Test
    void 분기_테스트() {
        assertThat(GradeFactory.getGrade(90).getGrade()).isEqualTo("A");
        assertThat(GradeFactory.getGrade(89).getGrade()).isEqualTo("B");
    }
```

<br>

### 장점 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다

서비스 제공자 프레임워크는 이 장점을 활용한 대표적인 예이다. 

**클라이언트가 구현체에 대해 알 필요없이 서비스를 사용할 수 있게 해준다.**

서비스 제공자 프레임워크는 주로 4가지 컴포넌트로 구성된다.

- 서비스 인터페이스: 클라이언트가 사용할 서비스의 인터페이스
- 제공자 등록 API: 제공자가 구현체를 등록할 때 사용한다.
- 서비스 접근 API: 클라이언트가 서비스의 인스턴스를 얻을 때 사용한다.
- 서비스 제공자 인터페이스: 서비스 인스턴스를 생성하는 팩토리 객체에 대한 인터페이스.


```Java
import java.util.HashMap;
import java.util.Map;

// 1. 서비스 인터페이스: 클라이언트가 사용할 서비스의 인터페이스
interface Service {
    void execute();
}

// 2. 서비스 제공자 인터페이스: 서비스 인스턴스를 생성하는 팩토리 객체에 대한 인터페이스
interface ServiceProvider {
    Service newService();
}

// 3. 서비스 제공자 등록과 접근을 관리하는 클래스
class ServiceManager {
    private static final Map<String, ServiceProvider> providers = new HashMap<>();
    public static final String DEFAULT_PROVIDER_NAME = "<def>";

    // 제공자 등록 API:  제공자가 구현체를 등록할 때 사용
    public static void registerDefaultProvider(ServiceProvider p) {
        registerProvider(DEFAULT_PROVIDER_NAME, p);
    }

    public static void registerProvider(String name, ServiceProvider p) {
        providers.put(name, p);
    }

    // 서비스 접근 API: 클라이언트가 서비스의 인스턴스를 얻을 때 사용
    public static Service getService(String name) {
        ServiceProvider p = providers.get(name);
        if (p == null)
            throw new IllegalArgumentException("No provider registered with name: " + name);
        return p.newService();
    }
}

// 구체적인 서비스와 제공자 구현
class ConcreteService1 implements Service {
    @Override
    public void execute() {
        System.out.println("Executing service 1");
    }

    // 서비스 제공자
    public static ServiceProvider provider() {
        return new ServiceProvider() {
            @Override
            public Service newService() {
                return new ConcreteService1();
            }
        };
    }
}

// 메인 클래스
public class ServiceProviderFrameworkDemo {
    public static void main(String[] args) {
        // 서비스 제공자 등록
        ServiceManager.registerProvider("service1", ConcreteService1.provider());

        // 서비스 사용
        Service service1 = ServiceManager.getService("service1");
        service1.execute(); // 출력: Executing service 1
    }
}
```

<br>

#### 브릿지 패턴

구현부에서 추상층을 분리하여 각자 독립적으로 변형할 수 있게 하는 패턴

```Java
// 1. 추상층
abstract class Shape {
    protected Color color; // 구현부

    public Shape(Color color) {
        this.color = color;
    }

    abstract void draw();
}

// 2. 구현부
interface Color {
    void applyColor();
}

// 3. 구현부 구현체
class RedColor implements Color {
    @Override
    public void applyColor() {
        System.out.println("red.");
    }
}

class GreenColor implements Color {
    @Override
    public void applyColor() {
        System.out.println("green.");
    }
}

// 4. 추상층 구현체
class Triangle extends Shape {
    public Triangle(Color color) {
        super(color);
    }

    @Override
    void draw() {
        System.out.print("Triangle filled with color ");
        color.applyColor(); // 구현부에 위임
    }
}

class Pentagon extends Shape {
    public Pentagon(Color color) {
        super(color);
    }

    @Override
    void draw() {
        System.out.print("Pentagon filled with color ");
        color.applyColor();
    }
}

// 5. 메인 클래스
public class BridgePatternDemo {
    public static void main(String[] args) {
        Shape triangle = new Triangle(new RedColor());
        triangle.draw(); // 출력: Triangle filled with color red.

        Shape pentagon = new Pentagon(new GreenColor());
        pentagon.draw(); // 출력: Pentagon filled with color green.
    }
}
```

<br><br><br>


### 단점 1. 상속을 할 수 없다

생성자를 `private` 으로 제한하기 때문에 하위 클래스를 만들 수 없다.

불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 장점으로 받아들일 수도 있다.

만약 상속을 통해 하위 클래스가 생성되면, 상위 클래스의 메서드가 하위 클래스에서 오버라이드되어 동작을 변경할 수 있다.


```Java
// 불변 클래스의 예시
public final class ImmutableClass {
    private final String value;

    private ImmutableClass(String value) {
        this.value = value;
    }

    public static ImmutableClass valueOf(String value) {
        return new ImmutableClass(value);
    }

    public String getValue() {
        return value;
    }
}

// 이 클래스는 ImmutableClass를 상속할 수 없음
// public class ExtendedClass extends ImmutableClass { ... } // 오류 발생

@Test
    void 불변_테스트() {
        ImmutableClass obj = ImmutableClass.valueOf("Hello");

        assertThat(obj.getValue()).isEqualTo("Hello");
    }
```

<br>

#### `컴포지션`

상속을 할 수 없기 때문에 컴포지션 사용(한 클래스가 다른 클래스의 기능을 확장하는 대신 그 클래스의 인스턴스를 자신의 필드로 포함시키는)을 유도한다

```Java
// 기본 기능을 제공하는 클래스
class Engine {
    public void start() {
        System.out.println("Engine is starting.");
    }
}

// 다른 클래스가 Engine 클래스를 포함하여 기능 확장
class Car {
    private Engine engine;

    public Car() {
        this.engine = new Engine(); // 컴포지션: Engine 객체를 Car의 필드로 포함
    }

    public void startCar() {
        engine.start(); // 포함된 Engine 객체의 기능을 사용
        System.out.println("Car is ready to go!");
    }
}

public class CompositionExample {
    public static void main(String[] args) {
        Car myCar = new Car();
        myCar.startCar();
    }
}
```

이렇게 `Comoosition`을 사용하면

- 상속을 하지 않아도 된다
- 코드 재사용성이 높아진다
- 런타임에 동적으로 기능을 확장할 수 있다
- 코드가 유연해진다
- 캡슐화를 통해 객체 간의 결합도가 낮아진다


<br>

### 단점 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다

생성자처럼 API 설명에 명확히 드러나지 않기 때문에 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화할 방법을 알아내야 한다.

<br><br><br>

### 정적 팩토리 메서드 네이밍 컨벤션

- `from`
  - 매개변수 하나를 받아서 해당 타입의 ㄷ인스턴스를 반환하는 형변환 메서드
  - `Date d = Date.from(instant);`

- `of`
  - 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
  - `Set<Rank> cards = EnumSet.of(JACK,QUEEN,KING);`


- `valueOf`
  - `from`과 `of`의 더 자세한 버전
  - `BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);`

- `instance`, `getInstance`
  - (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, **같은 인스턴스임을 보장하지는 않는다.**
  - `StackWalker luke = StackWalker.getInstance(options);`

- `create`, `newInstance`
  - instance or getInstance 와 같지만, **매번 새로운 인스턴스를 생성해 반환함을 보장한다.**
  - `Object newArray = Array.newInstance(classObject,arrayLen);`

- `getType`
  - `getInstance`와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메서드를 정의할 때 쓴다. `Type`은 팩토리 메서드가 반환할 객체의 타입이다
  - `FileStore fs = Files.getFileStore(path);`


- `newType`
  - `newInstance`와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메서드를 정의할 때 쓴다. `Type`은 팩토리 메서드가 반환할 객체의 타입이다
  - `BufferedReader br = Files.newBufferedReader(path);`


- `type`
  - `getType`와 `newType`의 간결한 버전
  - `List<Complaint litany = Collections.list(legacyLitany);` 