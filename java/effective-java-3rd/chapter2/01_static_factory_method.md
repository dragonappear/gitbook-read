# 생성자 대신 정적 팩토리 메서드를 고려해라

### 정적 팩토리 메서드란

클라이언트에게 public 생성자 대신(혹은 함께) 정적 팩토리 메서드로 제공하여 객체를 생성할 수 있게 한다.

**정적 팩토리 메서드로 인스턴스를 생성하거나 반환하는 방식을 강제하는 효과가 있다.**

<br><br><br>

### 장점 1. 이름을 가질 수 있다

생성자에 넘기는 매개변수와 생성자 자체로는 객체의 특성을 제대로 이해하기 어렵다.
정적 팩토리 메서드는 이름만 잘 지으면 반환될 객체의 특성을 보다 쉽게 묘사할 수 있다.

```Java
new BigInteger(int,int,random);

BigInteger.probablePrime(int,int,random);
```

객체를 생성하는 두가지 방법(생성자, 정적 팩토리 메서드) 중 정적 팩토리 메서드가 '값이 소수인 BigInteger를 반환한다' 의미를 잘 설명할 수 있다.

**한 클래스에 시그니처가 같은 생성자가 여러개 필요할 것 같으면 생성자를 정적 팩토리 메서드로 바꾸고 각각의 차이를 드러내는 이름으로 지어주면 된다.**

<br>

### 장점 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다

인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 방식으로 불필요한 객체 생성을 피할 수 있다. 
객체 생성은 비용이 크기 때문에 같은 객체가 자주 요청되는 상황이라면 성능을 상당히 끌어올려 준다.

정적 펙토리 메서드는 인스턴스를 통제함으로써 싱글톤으로 만들거나, 인스턴스화 불가로 클래스를 만들거나, 불변값 클래스에서 동치인 인스턴스가 단 하나임을 보장할 수 있다.

<br>

### 장점 3. 반환 타입의 하위 타입 객체를 반환할 수 있다

구현 클래스를 공개하지 않고도 객체를 반환할 수 있어서 API를 유연하고, 작게 만들 수 있다.

![](https://github.com/dragonappear/read/assets/89398909/996ec364-39cb-4a17-9709-2360570624f2)

정적 팩토리 메서드를 사용하여 구현 클래스를 숨겼음을 확인할 수 있다.

**정적 팩토리 메서드를 사용하는 클라이언트는 반환된 객체를 인터페이스만으로 다루게 된다.**

<br>

### 장점 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다

클라이언트는 팩토리가 반환하는 객체가 어느 클래스의 인스턴스인지 알 필요 없이 인스턴스를 사용할 수 있다.

<br>

### 장점 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다

<br><br><br>


### 단점 1. 상속을 할 수 없다

생성자를 private 으로 제한하기 때문에 하위 클래스를 만들 수 없다

이 제약은 상속보다 컴포지션을 사용하도록 유도하고 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 장점으로 받아들일 수도 있다.

<br>

### 단점 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다

<br><br><br>

## 정적 팩토리 메서드 네이밍

### `from`

매개변수 하나를 받아서 해당 타입의 ㄷ인스턴스를 반환하는 형변환 메서드

`Date d = Date.from(instant);`

<br>

### `of`

여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드

`Set<Rank> cards = EnumSet.of(JACK,QUEEN,KING);`

<br>

### `valueOf`

`from`과 `of`의 더 자세한 버전

`BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);`

<br>

### `instance`, `getInstance`

(매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, **같은 인스턴스임을 보장하지는 않는다.**

`StackWalker luke = StackWalker.getInstance(options);`

<br>

### `create`, `newInstance`

instance or getInstance 와 같지만, **매번 새로운 인스턴스를 생성해 반환함을 보장한다.**

`Object newArray = Array.newInstance(classObject,arrayLen);`

<br>

### `getType`

`getInstance`와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메서드를 정의할 때 쓴다. `Type`은 팩토리 메서드가 반환할 객체의 타입이다

`FileStore fs = Files.getFileStore(path);`

<br>

### `newType`

`newInstance`와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메서드를 정의할 때 쓴다. `Type`은 팩토리 메서드가 반환할 객체의 타입이다

`BufferedReader br = Files.newBufferedReader(path);`

<br>

### `type`

`getType`와 `newType`의 간결한 버전

`List<Complaint litany = Collections.list(legacyLitany);` 