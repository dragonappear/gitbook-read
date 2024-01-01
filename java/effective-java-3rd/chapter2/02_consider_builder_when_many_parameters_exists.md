# 생성자에 매개변수가 많다면 빌더를 고려해라


## 생성자, 정적 팩토리 제약

선택적 매개변수가 많을 때 적절히 대응하기 어렵다.

선택적 변수가 많으면 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 선택 매개변수를 2개까지 받는 생성자, ... 형태로 선택 매개변수를 전부 다 받는 생성자까지 늘려가는 방식 즉,
점층적 생성자 패턴을 사용할 수 있다.

보통 이런 생성자는 설정하길 원치 않는 매개변수까지 포함해줘야 해서, 어쩔 수 없이 매개변수에도 값을 지정해줘야 하는 상황이 발생한다.

```Java
public NutritionFacts(int servingSize, int servigns){
    this(servingSize, servings, 0);
}

public NutritionFacts(int servingSize, int servigns, int calories){
    this(servingSize, servings, calories, 0);
}

public NutritionFacts(int servingSize, int servigns, int calories, int fat){
    this(servingSize, servings, calories, fat, 0);
}
```

`NutritionFacts cocaCola = new NutritionFacts(240,9,100,0,35,27);`

위 코드에서는 지방이라는 선택 매개변수에 0을 넘겼다.

**이렇게 어쩔 수 없이 넣어줘야 하는 수가 많아지면 클라이언트는 코드를 작성하거나 읽기 어려워진다.**

- 코드를 읽을때 각 값의 의미가 무엇인지 헷갈리고
- 매개변수가 몇 개지인지도 주의해야 한다
- 또한 타입이 같은 매개변수가 여러개 있으면 클라이언트 실수로 순서를 바꿔 건네줘도 컴퍼일러가 알아차리지 못한다

<br><br><br>

## 자바빈즈 패턴

```Java
class Foo {
    private int necessaryVar1;
    private int necessaryVar2;
    private int optionalVar1;
    private int optionalVar2;
    
    public Foo() {}
    
    public void setNecessaryVar1(int val){};
    public void setNecessaryVar2(int val){};
    public void setOptionalVar1(int val){};
    public void setOptionalVar2(int val){};
}
```

이렇게 선택 배개변수가 많을 때 자바빈즈 패턴을 대안으로 사용할 수 있다.

매개변수가 없는 생성자로 객체를 만든 후 세터 메서드로 원하는 매개변수의 값을 설정하는 방식이다.

세터 메서드로 값을 설정함으로써 위에서 언급한 문제점들없이 가독성을 챙길 수 있다.

하지만 **객체 하나를 만드는데 메서드를 여러개 호출해야 하고, 객체가 완전히 생성되기 전까지 일관성이 무너진 상태에 놓이게 된다.**

**이처럼 일관성이 무너지는 문제 때문에 자바빈즈 패턴에서는 클래스를 불변으로 만들 수 없다.**

<br><br><br>


## 빌더 패턴

**점층적 생성자의 안정성과 자바 빈즈 패턴의 가독성을 모두 챙길 수 있다.**

### 빌더 패턴 구현

클라이언트는 필요한 객체를 직접 만드는 대신 필요 매개변수만으로 빌더 생성자(or 정적 팩터리)를 호출해서 빌더 객체를 얻고, 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정한다.

마지막으로 매개변수가 없는 build() 메서드를 호출해 필요한 객체를 얻는다.

> 빌더는 생성할 클래스 안에 정적 멤버 클래스로 만들어두는게 보통이다.

```Java
public class Foo {

    private final int necessaryVar1;
    private final int necessaryVar2;
    private final int optionalVar1;
    private final int optionalVar2;

    public static class Builder {
        // 필수 매개변수
        private final int necessaryVar1;
        private final int necessaryVar2;

        // 선택 매개변수 - 기본값으로 초기화
        private int optionalVar1 = 0;
        private int optionalVar2 = 0;

        public Builder(int necessaryVar1, int necessaryVar2) {
            this.necessaryVar1 = necessaryVar1;
            this.necessaryVar2 = necessaryVar2;
        }

        public Builder optionalVar1(int val) {
            this.optionalVar1 = val;
            return this;
        }

        public Builder optionalVar2(int val) {
            this.optionalVar2 = val;
            return this;
        }

        public Foo build() {
            return new Foo(this);
        }
    }

    public Foo(Builder builder) {
        this.necessaryVar1 = builder.necessaryVar1;
        this.necessaryVar2 = builder.necessaryVar2;
        this.optionalVar1 = builder.optionalVar1;
        this.optionalVar2 = builder.optionalVar2;
    }
}
```

**빌더 패턴을 적용함으로써 클라이언트 코드는 쓰기 쉽고 읽기 쉬워진다.**

```Java
@Test
    void 빌더_패턴_구현_테스트(){
        Foo foo = new Builder(1, 2)
                .optionalVar1(3)
                .optionalVar2(4)
                .build();

        assertThat(foo.getNecessaryVar1()).isOne();
        assertThat(foo.getNecessaryVar2()).isEqualTo(2);
        assertThat(foo.getOptionalVar1()).isEqualTo(3);
        assertThat(foo.getOptionalVar2()).isEqualTo(4);
    }
```

잘못된 매개변수는 빌더의 생성자와 메서드에서 입력 매개변수를 검사하고, build 메서드가 호출하는 생성자에서 여러 매개변수에 걸친 불변식을 검사하자.

<br>

### 계층적 클래스

**빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.**

각 계층의 클래스에 관련된 빌더를 멤버로 정의한다.

추상 클래스는 추상 빌더를, 구체 클래스는 구체 빌더를 갖도록 한다.

```Java
public abstract class Pizza {
    public enum Topping {HAM, MUSHROOM, ONION, PEPEER, SAUSAGE}

    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // 하위 클래스는 이 메서드를 오버라이딩하여 "this"를 반환하도록 해야 한다
        protected abstract T self();
    }
    
    public Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }
}
```

구현체

```Java
public class NyPizza extends Pizza {

    public enum Size {SMALL, MEDIUM, LARGE}

    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override
        Pizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }

    }

    private NyPizza(Builder builder) {
        super(builder);
        this.size = builder.size;
    }
}
```

하위 클래스의 빌더가 정의한 build 메서드는 구체 하위 클래스를 반환하도록 선언한다.

```Java
@Test
    void 계층적_빌더_패턴_구현_테스트() {
        Pizza pizza = new Builder(LARGE)
                .addTopping(SAUSAGE)
                .addTopping(ONION)
                .build();

        assertThat(pizza.getToppings()).containsOnly(SAUSAGE, ONION);
    }
```

빌더 패턴은 상당히 유연하고, 빌더 하나로 여러 객체를 순회하면서 만들 수 있다.

빌더 패턴을 사용하여 객체를 만들려면 먼저 빌더부터 만들어야 하는데, 빌더 생성 비용이 크지는 않지만 민감한 상황에서는 문제가 될 수 도 있다.ㄴ

또한 점층적 생성자 패턴보다는 코드가 장황해서 매개 변수 4개 이상은 되어야 값어치를 한다.

> 생성자나 정적 팩토리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는게 더 낫다. 매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다. 

**`API`는 시간이 지날수록 매개변수가 많아지는 경향이 있으니, 애초에 빌더로 시작하는 편이 나을 때가 많다.**