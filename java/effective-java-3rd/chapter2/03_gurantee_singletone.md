# private 생성자나 열거 타입으로 싱글턴임을 보증하라

클래스를 싱글톤으로 만들면 테스트하기 어려워질 수 있다.

타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글톤이 아니라면 싱글톤 인스턴스를 목 객체로 대체할 수 없기 때문이다.


### 1. public static final 필드 방식의 싱글턴

```Java
public class Foo {
    public static final Foo INSTANCE = new Foo();

    private Foo() {
        // ...
    }
    
    public void doSomething() {
        // ...
    }
}
```

생성자를 private 으로 감춰두고 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 마련하는 방법

private 생성자는 public static final 필드인 INSTANCE 를 초기화할 때 딱 한번만 호출된다.

**해당 클래스가 싱글턴임이 API 에 명백히 드러난다.**

<br><br>

### 2. 정적 팩터리 방식의 싱글턴

```Java
public class Foo {
    private static final Foo INSTANCE = new Foo();

    private Foo() {
        // ...
    }

    public static Foo getInstance() {
        return INSTANCE;
    }
    
    public void doSomething() {
        // ...
    }
}
```

- 마음이 바뀌면 API를 바꾸지 않고도 싱글톤이 아니게 변경할 수 있다.
- 정적 팩토리를 제네릭 싱글톤 팩토리로 만들 수 있다.
- 정적 팩토리의 메서드 참조를 공급자(supplier)로 사용할 수 있다.

#### 직렬화
 
싱글톤 클래스를 직렬화하려면 단순히 Serializable 을 구현한다고 선언하는 것만으로는 부족하다.
모든 인스턴스 필드를 일시적(transient)이라고 선언하고 readResolve 메서드를 제공해야 한다.
그렇지 않으면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다.

```Java
// 싱글턴임을 보장해주는 readResolve 메서드
private Object readResolve() {
    // 진짜 Foo 를 반환하고, 가짜 Foo 는 GC 에게 맡긴다.
    return INSTANCE;
}
```

<br><br>

### 3. 열거 타입 방식의 싱글턴

```Java
public enum Foo {
    INSTANCE;
    
    public void doSomething() {
        // ...
    }
}
```

- public 필드 방식과 비슷하지만, 더 간결하고, 추가 노력 없이 직렬화할 수 있다.
  - 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽히 막아준다.
- **대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.**
- 단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.


