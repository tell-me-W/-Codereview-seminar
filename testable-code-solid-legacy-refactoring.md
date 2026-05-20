# 테스트하기 쉬운 코드: SOLID와 레거시 리팩토링

## 1. 왜 테스트하기 쉬운 코드가 중요한가?

테스트 코드 작성이 어려운 이유는 테스트 도구를 잘 몰라서가 아닐 때가 많다. 실제로는 코드 구조가 테스트하기 어렵게 되어 있기 때문에 테스트 작성이 어려워진다.

테스트하기 쉬운 코드는 보통 변경하기도 쉽다. 반대로 테스트하기 어려운 코드는 의존성이 복잡하고, 책임이 섞여 있고, 작은 수정도 예상하지 못한 영향을 만들 가능성이 높다.

레거시 시스템에서 테스트 가능한 구조를 만드는 것은 단순히 테스트 코드를 늘리는 일이 아니라, 앞으로의 변경 비용을 줄이는 일이다.

---

## 2. 테스트하기 어려운 코드의 특징

테스트하기 어려운 코드는 대체로 비슷한 특징을 가진다.

- 하나의 클래스나 메서드가 너무 많은 일을 한다.
- static 메서드에 강하게 의존한다.
- private 메서드 안에 중요한 비즈니스 로직이 숨어 있다.
- 메서드 내부에서 직접 객체를 생성한다.
- DB, 외부 API, 파일, 현재 시간 같은 외부 요소에 직접 의존한다.
- if/else 또는 switch 분기가 계속 늘어난다.
- 테스트하려는 로직보다 준비해야 할 주변 조건이 더 많다.

이런 구조에서는 JUnit, Mockito, PowerMock 같은 도구를 알아도 테스트 작성이 계속 어려워진다.

---

## 3. 좋은 코드 구조의 기준: 응집도와 결합도

응집도는 하나의 모듈이 얼마나 관련 있는 책임들로 구성되어 있는지를 의미한다. 응집도가 높으면 코드의 목적이 명확하고 테스트 대상도 분명해진다.

결합도는 모듈 간 의존성이 얼마나 강한지를 의미한다. 결합도가 높으면 하나를 테스트하기 위해 너무 많은 주변 객체와 환경이 필요해진다.

좋은 구조는 보통 높은 응집도와 낮은 결합도를 가진다. 테스트 코드 관점에서는 "이 로직만 따로 실행할 수 있는가?", "외부 의존성을 쉽게 대체할 수 있는가?"가 중요한 기준이 된다.

---

## 4. SOLID란 무엇인가?

SOLID는 객체지향 설계를 위한 다섯 가지 원칙이다.

- SRP: 단일 책임 원칙
- OCP: 개방-폐쇄 원칙
- LSP: 리스코프 치환 원칙
- ISP: 인터페이스 분리 원칙
- DIP: 의존성 역전 원칙

SOLID는 모든 코드를 복잡하게 추상화하자는 의미가 아니다. 변경에 강하고 테스트하기 쉬운 구조를 만들기 위한 판단 기준에 가깝다.

레거시 코드에서는 SOLID를 처음부터 완벽하게 적용하기 어렵다. 중요한 것은 현재 코드에서 변경이 자주 일어나는 부분, 테스트가 어려운 부분, 장애 위험이 큰 부분부터 점진적으로 개선하는 것이다.

---

## 5. SRP: 단일 책임 원칙

SRP는 Single Responsibility Principle의 약자이며, 하나의 클래스는 하나의 책임을 가져야 한다는 원칙이다.

여기서 책임은 단순히 "기능 하나"를 의미하지 않는다. 책임은 변경되는 이유라고 볼 수 있다. 하나의 클래스가 여러 이유로 변경된다면 그 클래스는 여러 책임을 가지고 있을 가능성이 높다.

예를 들어 주문 처리 서비스가 다음 일을 모두 하고 있다고 가정해보자.

- 주문 금액 계산
- 할인 정책 적용
- 재고 확인
- 주문 저장
- 결제 API 호출
- 알림 발송
- 로그 기록

이런 클래스는 기능이 추가될수록 계속 커진다. 할인 정책을 바꿀 때도 수정되고, 결제 방식을 바꿀 때도 수정되고, 알림 정책을 바꿀 때도 수정된다. 테스트를 작성하려고 해도 주문 금액 계산 하나를 검증하기 위해 DB, 결제 API, 알림 발송 객체까지 준비해야 할 수 있다.

SRP를 적용하면 책임을 분리한다.

```java
public class OrderService {
    private final PriceCalculator priceCalculator;
    private final StockService stockService;
    private final PaymentClient paymentClient;
    private final OrderRepository orderRepository;
    private final NotificationService notificationService;

    public void order(OrderRequest request) {
        int price = priceCalculator.calculate(request);
        stockService.validate(request);
        paymentClient.pay(price);
        orderRepository.save(request, price);
        notificationService.send(request);
    }
}
```

위 코드에서 `OrderService`는 전체 주문 흐름을 조율한다. 금액 계산 자체는 `PriceCalculator`가 담당하고, 재고 확인은 `StockService`가 담당한다. 각 객체는 더 작은 책임을 가지기 때문에 개별 테스트가 쉬워진다.

특히 테스트 코드 관점에서는 SRP가 중요하다. 하나의 클래스가 한 가지 책임만 가지면 테스트도 한 가지 이유로 실패한다. 반대로 책임이 섞여 있으면 테스트가 실패했을 때 원인을 찾기 어렵다.

레거시 코드에서 SRP를 적용할 때는 큰 클래스를 한 번에 전부 나누려고 하면 위험하다. 먼저 테스트하고 싶은 로직을 찾아서 별도 클래스로 분리하는 방식이 좋다.

예를 들어 다음과 같은 기준으로 시작할 수 있다.

- 계산 로직을 별도 클래스로 분리한다.
- 조건 판단 로직을 별도 클래스로 분리한다.
- 외부 API 호출을 별도 Client 클래스로 분리한다.
- DB 접근 로직을 Repository로 분리한다.
- 메시지 생성 로직을 별도 Formatter 또는 Builder로 분리한다.

SRP의 목표는 클래스를 무조건 작게 만드는 것이 아니다. 변경 이유를 분리하고, 테스트 대상의 범위를 명확하게 만드는 것이다.

---

## 6. OCP: 개방-폐쇄 원칙

OCP는 Open-Closed Principle의 약자이며, 확장에는 열려 있고 변경에는 닫혀 있어야 한다는 원칙이다.

처음 들으면 모순처럼 느껴질 수 있다. 새로운 기능을 추가하려면 코드를 수정해야 하는데, 변경에는 닫혀 있어야 한다는 말이 이상하게 들릴 수 있다.

OCP의 핵심은 "새로운 요구사항이 생겼을 때 기존 코드를 계속 수정하는 구조를 줄이자"는 것이다.

예를 들어 할인 정책이 다음처럼 작성되어 있다고 가정해보자.

```java
public int calculateDiscount(String memberType, int price) {
    if ("BASIC".equals(memberType)) {
        return price;
    }

    if ("SILVER".equals(memberType)) {
        return price - 1000;
    }

    if ("GOLD".equals(memberType)) {
        return price - 3000;
    }

    return price;
}
```

이 코드는 새로운 등급이 추가될 때마다 기존 메서드를 수정해야 한다. 조건이 늘어나면 테스트 케이스도 복잡해진다. 기존 등급 로직을 건드리다가 이미 잘 동작하던 할인 정책을 깨뜨릴 수도 있다.

OCP를 고려하면 할인 정책을 역할로 분리할 수 있다.

```java
public interface DiscountPolicy {
    boolean supports(MemberType memberType);

    int discount(int price);
}
```

```java
public class GoldDiscountPolicy implements DiscountPolicy {
    @Override
    public boolean supports(MemberType memberType) {
        return memberType == MemberType.GOLD;
    }

    @Override
    public int discount(int price) {
        return price - 3000;
    }
}
```

```java
public class DiscountCalculator {
    private final List<DiscountPolicy> policies;

    public int calculate(MemberType memberType, int price) {
        return policies.stream()
                .filter(policy -> policy.supports(memberType))
                .findFirst()
                .map(policy -> policy.discount(price))
                .orElse(price);
    }
}
```

이 구조에서는 새로운 회원 등급이 생겼을 때 기존 `DiscountCalculator`를 수정하지 않고 새로운 `DiscountPolicy` 구현체를 추가할 수 있다. 기존 정책의 테스트를 크게 건드리지 않고, 새 정책에 대한 테스트만 추가하면 된다.

물론 모든 if/else를 무조건 없애야 하는 것은 아니다. 단순한 조건문은 오히려 명확할 수 있다. OCP를 고민해야 하는 순간은 다음과 같다.

- 같은 조건문이 여러 곳에 반복된다.
- 새로운 조건이 자주 추가된다.
- 조건 하나를 추가할 때 기존 로직을 많이 수정해야 한다.
- 조건 분기 때문에 테스트 케이스가 급격히 늘어난다.
- 리뷰할 때 변경 영향 범위를 파악하기 어렵다.

OCP는 디자인 패턴과도 연결된다. Strategy 패턴은 OCP를 적용하는 대표적인 방법이다. 그러나 중요한 것은 패턴 이름이 아니라, 변경이 자주 발생하는 지점을 독립적으로 확장할 수 있게 만드는 것이다.

---

## 7. LSP: 리스코프 치환 원칙

LSP는 Liskov Substitution Principle의 약자이며, 하위 타입은 상위 타입을 대체할 수 있어야 한다는 원칙이다.

쉽게 말하면 부모 타입으로 기대한 동작이 자식 타입에서도 깨지면 안 된다는 뜻이다.

예를 들어 다음과 같은 상속 구조를 생각해볼 수 있다.

```java
public class FileStorage {
    public void save(File file) {
        // 파일 저장
    }
}
```

```java
public class ReadOnlyFileStorage extends FileStorage {
    @Override
    public void save(File file) {
        throw new UnsupportedOperationException("읽기 전용 저장소입니다.");
    }
}
```

겉으로 보면 `ReadOnlyFileStorage`는 `FileStorage`의 하위 타입이다. 하지만 `FileStorage`를 사용하는 코드는 `save()`가 가능하다고 기대한다. 그런데 하위 타입이 예외를 던지면 기존 기대가 깨진다.

이런 구조에서는 테스트도 혼란스러워진다. 상위 타입 기준으로 작성한 테스트는 특정 하위 타입에서 실패할 수 있고, 사용하는 쪽에서는 실제 구현체가 무엇인지 계속 신경 써야 한다.

LSP 위반은 보통 잘못된 상속에서 많이 발생한다.

- 부모 메서드를 자식이 사용할 수 없어 예외를 던진다.
- 자식 클래스가 부모 클래스의 의미를 바꾼다.
- 부모 타입으로 다룰 때는 정상인데 특정 자식 타입에서만 특별한 조건이 필요하다.
- 상속 관계가 "is-a"가 아니라 단순 코드 재사용을 위해 만들어졌다.

레거시 코드에서는 상속을 재사용 수단으로 사용하는 경우가 많다. 하지만 상속은 부모의 계약까지 함께 물려받는다. 단순히 공통 메서드를 쓰고 싶다는 이유만으로 상속하면 LSP를 위반하기 쉽다.

대안으로 조합을 고려할 수 있다.

```java
public class ReportService {
    private final FileWriter fileWriter;

    public void createReport(Report report) {
        String content = report.format();
        fileWriter.write(content);
    }
}
```

상속 대신 필요한 객체를 필드로 가지고 협력하게 만들면, 역할을 더 명확하게 나눌 수 있다. 테스트할 때도 `FileWriter`를 Mock으로 대체하기 쉽다.

LSP는 다른 원칙보다 추상적으로 느껴질 수 있다. 실무에서는 다음 질문으로 점검하면 좋다.

- 이 자식 클래스는 부모 클래스가 약속한 동작을 그대로 지키는가?
- 부모 타입으로 바꿔 사용해도 사용하는 쪽 코드를 수정하지 않아도 되는가?
- 자식 클래스 때문에 사용하는 쪽에 예외 조건이 늘어나고 있지는 않은가?
- 단순 코드 재사용 때문에 상속을 사용하고 있지는 않은가?

테스트 관점에서 LSP를 지키면 구현체를 바꿔도 테스트 구조가 크게 흔들리지 않는다. 반대로 LSP가 깨지면 구현체마다 별도 예외 테스트가 필요해지고, 코드 사용 규칙이 암묵적으로 늘어난다.

---

## 8. ISP: 인터페이스 분리 원칙

ISP는 Interface Segregation Principle의 약자이며, 클라이언트는 자신이 사용하지 않는 메서드에 의존하지 않아야 한다는 원칙이다.

인터페이스는 작고 구체적인 역할 단위로 나뉘어야 한다. 하나의 큰 인터페이스에 너무 많은 메서드가 들어 있으면 구현체도 불필요한 메서드를 구현해야 하고, 테스트용 Mock도 불필요하게 복잡해진다.

예를 들어 다음과 같은 인터페이스가 있다고 가정해보자.

```java
public interface UserManager {
    User findUser(long userId);

    void createUser(User user);

    void updateUser(User user);

    void deleteUser(long userId);

    void sendWelcomeEmail(User user);

    void writeAuditLog(User user);
}
```

이 인터페이스는 사용자 조회, 생성, 수정, 삭제, 메일 발송, 감사 로그 기록까지 모두 포함한다. 어떤 클래스는 사용자 조회만 필요할 수 있는데, 이 큰 인터페이스 전체에 의존하게 된다.

테스트할 때도 문제가 생긴다. 조회 기능만 테스트하고 싶은데 `UserManager` Mock을 만들면 필요 없는 메서드까지 같은 인터페이스에 묶여 있다. 시간이 지나면 하나의 인터페이스 변경이 여러 테스트에 영향을 줄 수 있다.

ISP를 적용하면 역할별로 인터페이스를 나눌 수 있다.

```java
public interface UserReader {
    User findUser(long userId);
}
```

```java
public interface UserWriter {
    void createUser(User user);

    void updateUser(User user);

    void deleteUser(long userId);
}
```

```java
public interface WelcomeEmailSender {
    void sendWelcomeEmail(User user);
}
```

이렇게 나누면 사용하는 쪽은 자신에게 필요한 역할에만 의존한다.

```java
public class UserProfileService {
    private final UserReader userReader;

    public UserProfile getProfile(long userId) {
        User user = userReader.findUser(userId);
        return UserProfile.from(user);
    }
}
```

이제 `UserProfileService` 테스트에서는 `UserReader`만 Mock으로 만들면 된다. 메일 발송, 사용자 삭제, 감사 로그 같은 관심 없는 기능을 신경 쓸 필요가 없다.

ISP는 특히 레거시 시스템의 비대한 Service 인터페이스를 다룰 때 유용하다. 오래된 시스템에서는 "Manager", "Service", "Util" 같은 이름의 클래스나 인터페이스가 많은 책임을 가지는 경우가 많다.

다음 신호가 보이면 ISP 적용을 고민할 수 있다.

- 인터페이스 이름이 너무 넓은 의미를 가진다.
- 구현체에서 사용하지 않는 메서드를 빈 구현으로 둔다.
- 어떤 메서드는 특정 구현체에서 항상 예외를 던진다.
- 테스트 Mock 설정이 테스트 대상보다 더 길다.
- 인터페이스 메서드 하나를 추가했는데 영향받는 클래스가 너무 많다.

인터페이스를 나누면 파일 수가 늘어날 수 있다. 하지만 역할이 명확해지고 테스트 의존성이 줄어든다면 그 비용은 충분히 감수할 만하다.

---

## 9. DIP: 의존성 역전 원칙

DIP는 Dependency Inversion Principle의 약자이며, 고수준 모듈이 저수준 모듈에 직접 의존하지 말고 둘 다 추상화에 의존해야 한다는 원칙이다.

말이 어렵지만 테스트 코드 관점에서는 이렇게 이해할 수 있다.

"테스트 대상 코드가 직접 구체 객체를 만들거나 static 메서드를 호출하면 대체하기 어렵다. 그러니 바꿔 끼울 수 있는 역할에 의존하게 만들자."

예를 들어 다음 코드는 테스트하기 어렵다.

```java
public class OrderService {
    public void order(OrderRequest request) {
        PaymentClient paymentClient = new PaymentClient();
        paymentClient.pay(request.getAmount());
    }
}
```

`OrderService`가 직접 `PaymentClient`를 생성하고 있다. 테스트에서는 실제 결제 API를 호출하고 싶지 않지만, 이미 메서드 내부에서 구체 객체를 생성하기 때문에 대체하기 어렵다.

DIP를 적용하면 의존성을 외부에서 주입받는다.

```java
public interface PaymentProcessor {
    void pay(int amount);
}
```

```java
public class OrderService {
    private final PaymentProcessor paymentProcessor;

    public OrderService(PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }

    public void order(OrderRequest request) {
        paymentProcessor.pay(request.getAmount());
    }
}
```

이제 운영 환경에서는 실제 결제 구현체를 넣고, 테스트에서는 Mock 또는 Fake 구현체를 넣을 수 있다.

```java
PaymentProcessor paymentProcessor = mock(PaymentProcessor.class);
OrderService orderService = new OrderService(paymentProcessor);

orderService.order(request);

verify(paymentProcessor).pay(10000);
```

DIP는 Mockito와 매우 잘 연결된다. Mock을 사용하려면 의존성을 바꿔 끼울 수 있어야 한다. 의존성이 메서드 내부에서 직접 생성되거나 static으로 고정되어 있으면 일반 Mock으로는 제어하기 어렵고, PowerMock 같은 강한 도구가 필요해진다.

따라서 PowerMock이 필요하다는 것은 종종 DIP가 적용되어 있지 않다는 신호다.

레거시 코드에서 DIP를 바로 적용하기 어렵다면 Wrapper를 만들 수 있다.

```java
public interface CurrentTimeProvider {
    LocalDateTime now();
}
```

```java
public class SystemCurrentTimeProvider implements CurrentTimeProvider {
    @Override
    public LocalDateTime now() {
        return LocalDateTime.now();
    }
}
```

기존 코드가 `LocalDateTime.now()`를 직접 호출하고 있었다면, 이를 `CurrentTimeProvider` 뒤로 숨길 수 있다. 테스트에서는 원하는 시간을 반환하는 Fake 구현체를 넣으면 된다.

```java
public class FixedCurrentTimeProvider implements CurrentTimeProvider {
    @Override
    public LocalDateTime now() {
        return LocalDateTime.of(2026, 5, 21, 10, 0);
    }
}
```

DIP를 적용할 때 주의할 점도 있다. 모든 클래스에 무조건 인터페이스를 만들 필요는 없다. 변경 가능성이 낮고 외부 의존성이 없는 단순 값 객체나 계산 객체는 구체 클래스에 직접 의존해도 괜찮다.

DIP를 우선 적용하면 좋은 대상은 다음과 같다.

- DB 접근
- 외부 API 호출
- 파일 시스템 접근
- 현재 시간
- 랜덤값
- 메일, SMS, 알림 발송
- static 유틸에 감춰진 외부 의존성
- 테스트에서 결과를 통제해야 하는 객체

테스트하기 쉬운 코드를 만들고 싶다면 DIP는 가장 먼저 체감되는 원칙이다. 의존성을 바깥에서 넣을 수 있게 만들면 테스트 준비가 단순해지고, Mock을 이용한 검증도 쉬워진다.

---

## 10. 레거시 코드에서 SOLID를 적용하는 현실적인 방법

레거시 시스템에 SOLID를 적용할 때 가장 위험한 접근은 "전체 구조를 한 번에 고치자"는 생각이다. 이미 운영 중인 시스템에서는 큰 리팩토링이 오히려 장애 위험을 높일 수 있다.

현실적인 접근은 작은 단위로 개선하는 것이다.

### 10.1 변경하는 코드 주변부터 개선한다

모든 코드를 한 번에 깨끗하게 만들 필요는 없다. 기능 추가나 버그 수정으로 실제로 손대는 코드 주변부터 개선한다.

예를 들어 특정 할인 로직을 수정해야 한다면 전체 주문 시스템을 리팩토링하지 않는다. 먼저 할인 계산 로직을 분리하고, 해당 부분에 테스트를 추가한다.

이 방식은 변경 범위를 작게 유지하면서도 테스트 가능한 영역을 조금씩 늘릴 수 있다.

### 10.2 테스트가 필요한 로직을 먼저 분리한다

레거시 코드에는 한 메서드 안에 여러 작업이 섞여 있는 경우가 많다.

```java
public void process(Order order) {
    // DB 조회
    // 할인 계산
    // 재고 확인
    // 결제 요청
    // 주문 저장
    // 알림 발송
}
```

이런 메서드를 한 번에 테스트하려고 하면 많은 의존성을 준비해야 한다. 먼저 순수한 계산 로직이나 조건 판단 로직을 별도 클래스로 분리하면 테스트가 쉬워진다.

```java
public class DiscountCalculator {
    public int calculate(Order order) {
        // 할인 계산만 담당
    }
}
```

계산 로직은 DB나 API 없이 테스트할 수 있기 때문에 첫 번째 분리 대상으로 좋다.

### 10.3 static 호출은 Wrapper로 감싼다

레거시 코드에서 static 유틸은 흔하다.

```java
String encrypted = EncryptUtil.encrypt(password);
```

static 호출은 일반적인 Mock으로 대체하기 어렵다. 당장 코드를 전부 바꾸기 어렵다면 Wrapper를 만들 수 있다.

```java
public interface PasswordEncryptor {
    String encrypt(String password);
}
```

```java
public class StaticPasswordEncryptor implements PasswordEncryptor {
    @Override
    public String encrypt(String password) {
        return EncryptUtil.encrypt(password);
    }
}
```

운영 코드에서는 기존 static 유틸을 감싼 구현체를 사용하고, 테스트에서는 원하는 값을 반환하는 Mock을 사용한다.

### 10.4 private 메서드 테스트보다 책임 분리를 먼저 본다

private 메서드를 직접 테스트하고 싶어지는 경우가 있다. 하지만 private 메서드는 원래 클래스 내부 구현이다. private 메서드가 너무 복잡해서 직접 테스트하고 싶다면, 그 로직은 별도 클래스로 분리할 수 있는 책임일 가능성이 높다.

예를 들어 `OrderService` 안의 private 메서드가 복잡한 할인 계산을 하고 있다면 `DiscountCalculator`로 분리하는 것이 좋다.

PowerMock으로 private 메서드를 테스트할 수도 있지만, 장기적으로는 구조 개선이 더 좋은 방향이다.

### 10.5 외부 의존성은 경계 밖으로 밀어낸다

테스트를 어렵게 만드는 대표적인 요소는 외부 의존성이다.

- DB
- 외부 API
- 파일
- 현재 시간
- 랜덤값
- 네트워크
- 환경 변수

이런 요소는 비즈니스 로직 안에 직접 들어오지 않도록 경계를 만든다.

예를 들어 현재 시간을 직접 가져오는 대신 `CurrentTimeProvider`를 사용하고, 외부 API 호출은 `PaymentClient`, `MessageClient` 같은 객체 뒤로 숨긴다.

그 결과 비즈니스 로직은 "무엇을 할지"에 집중하고, 외부 의존성은 "어떻게 가져올지"를 담당한다.

### 10.6 신규 코드는 기존 레거시 스타일을 그대로 따라가지 않는다

레거시 시스템에서 자주 생기는 고민은 "기존 코드가 이렇게 되어 있으니 새 코드도 똑같이 작성해야 하나?"이다.

일관성은 중요하지만, 테스트하기 어려운 구조까지 그대로 반복할 필요는 없다. 신규 코드는 가능한 한 테스트 가능한 구조로 작성해야 한다.

예를 들어 기존 코드가 static 유틸과 큰 Service 클래스 중심으로 되어 있더라도, 새로 추가하는 기능은 다음 기준을 적용할 수 있다.

- 생성자 또는 DI를 통해 의존성을 주입받는다.
- 계산 로직은 별도 클래스로 분리한다.
- 외부 API 호출은 Client 클래스로 감싼다.
- 조건 분기가 커질 가능성이 있으면 정책 객체로 분리한다.
- 테스트하기 어려운 static 호출은 새 코드에 추가하지 않는다.

레거시 개선은 기존 코드를 모두 갈아엎는 것이 아니라, 새로 작성하는 코드부터 더 나은 방향으로 쌓아가는 것이다.

---

## 12. 예제로 보는 개선 흐름

다음 예시는 테스트하기 어려운 레거시 코드가 어떻게 테스트 가능한 구조로 바뀔 수 있는지 보여준다.

### 12.1 개선 전 코드

```java
public class OrderService {
    public int order(OrderRequest request) {
        User user = UserDao.findById(request.getUserId());

        int price = request.getPrice();

        if ("GOLD".equals(user.getGrade())) {
            price = price - 3000;
        } else if ("SILVER".equals(user.getGrade())) {
            price = price - 1000;
        }

        if (LocalDateTime.now().getHour() < 9) {
            throw new IllegalStateException("주문 가능 시간이 아닙니다.");
        }

        PaymentApi paymentApi = new PaymentApi();
        paymentApi.pay(price);

        OrderDao.save(request, price);

        return price;
    }
}
```

이 코드는 한눈에 보기에는 단순해 보일 수 있지만 테스트하기 어렵다.

문제점은 다음과 같다.

- `UserDao.findById()`와 `OrderDao.save()`가 static으로 호출된다.
- 할인 정책이 `OrderService` 안에 직접 들어 있다.
- 현재 시간을 `LocalDateTime.now()`로 직접 가져온다.
- `PaymentApi`를 메서드 내부에서 직접 생성한다.
- 주문 처리, 할인 계산, 주문 가능 시간 검증, 결제, 저장 책임이 하나의 메서드에 섞여 있다.

이 코드를 테스트하려면 DB, 현재 시간, 결제 API, static DAO 호출을 모두 제어해야 한다. 일반 Mockito만으로는 제어하기 어려워 PowerMock이 필요할 수 있다.

### 12.2 1단계: 할인 계산 분리

먼저 외부 의존성이 없는 계산 로직을 분리한다.

```java
public class DiscountCalculator {
    public int calculate(String grade, int price) {
        if ("GOLD".equals(grade)) {
            return price - 3000;
        }

        if ("SILVER".equals(grade)) {
            return price - 1000;
        }

        return price;
    }
}
```

이 클래스는 DB도, API도, 현재 시간도 필요 없다. 따라서 JUnit만으로 쉽게 테스트할 수 있다.

```java
@Test
public void calculate_goldUser_returnsDiscountedPrice() {
    DiscountCalculator calculator = new DiscountCalculator();

    int result = calculator.calculate("GOLD", 10000);

    assertEquals(7000, result);
}
```

이 단계만으로도 핵심 비즈니스 로직 일부를 보호할 수 있다.

### 12.3 2단계: 현재 시간 의존성 분리

현재 시간을 직접 호출하면 테스트 실행 시점에 따라 결과가 달라질 수 있다. 시간 의존성을 분리한다.

```java
public interface CurrentTimeProvider {
    LocalDateTime now();
}
```

```java
public class SystemCurrentTimeProvider implements CurrentTimeProvider {
    @Override
    public LocalDateTime now() {
        return LocalDateTime.now();
    }
}
```

주문 가능 시간 검증도 별도 클래스로 분리할 수 있다.

```java
public class OrderTimeValidator {
    private final CurrentTimeProvider currentTimeProvider;

    public OrderTimeValidator(CurrentTimeProvider currentTimeProvider) {
        this.currentTimeProvider = currentTimeProvider;
    }

    public void validate() {
        if (currentTimeProvider.now().getHour() < 9) {
            throw new IllegalStateException("주문 가능 시간이 아닙니다.");
        }
    }
}
```

테스트에서는 고정된 시간을 반환하는 Fake 객체를 사용할 수 있다.

```java
public class FixedCurrentTimeProvider implements CurrentTimeProvider {
    @Override
    public LocalDateTime now() {
        return LocalDateTime.of(2026, 5, 21, 8, 0);
    }
}
```

이제 시간에 따라 흔들리지 않는 테스트를 작성할 수 있다.

### 12.4 3단계: 외부 의존성을 인터페이스 뒤로 숨기기

DB와 결제 API 호출도 역할로 분리한다.

```java
public interface UserRepository {
    User findById(long userId);
}
```

```java
public interface OrderRepository {
    void save(OrderRequest request, int price);
}
```

```java
public interface PaymentProcessor {
    void pay(int price);
}
```

기존 static DAO나 외부 API는 구현체 안에서 감싼다.

```java
public class StaticUserRepository implements UserRepository {
    @Override
    public User findById(long userId) {
        return UserDao.findById(userId);
    }
}
```

이렇게 하면 운영 코드는 기존 방식을 계속 사용할 수 있고, 테스트 코드는 Mock으로 대체할 수 있다.

### 12.5 개선 후 코드

```java
public class OrderService {
    private final UserRepository userRepository;
    private final OrderRepository orderRepository;
    private final PaymentProcessor paymentProcessor;
    private final DiscountCalculator discountCalculator;
    private final OrderTimeValidator orderTimeValidator;

    public OrderService(
            UserRepository userRepository,
            OrderRepository orderRepository,
            PaymentProcessor paymentProcessor,
            DiscountCalculator discountCalculator,
            OrderTimeValidator orderTimeValidator
    ) {
        this.userRepository = userRepository;
        this.orderRepository = orderRepository;
        this.paymentProcessor = paymentProcessor;
        this.discountCalculator = discountCalculator;
        this.orderTimeValidator = orderTimeValidator;
    }

    public int order(OrderRequest request) {
        orderTimeValidator.validate();

        User user = userRepository.findById(request.getUserId());
        int price = discountCalculator.calculate(user.getGrade(), request.getPrice());

        paymentProcessor.pay(price);
        orderRepository.save(request, price);

        return price;
    }
}
```

개선 후 `OrderService`는 직접 DB를 호출하지 않고, 직접 결제 API를 생성하지 않고, 직접 현재 시간을 가져오지 않는다. 필요한 역할을 외부에서 주입받는다.

테스트는 다음처럼 작성할 수 있다.

```java
@Test
public void order_goldUser_paysDiscountedPrice() {
    UserRepository userRepository = mock(UserRepository.class);
    OrderRepository orderRepository = mock(OrderRepository.class);
    PaymentProcessor paymentProcessor = mock(PaymentProcessor.class);
    DiscountCalculator discountCalculator = new DiscountCalculator();
    OrderTimeValidator orderTimeValidator = mock(OrderTimeValidator.class);

    OrderRequest request = new OrderRequest(1L, 10000);
    User user = new User(1L, "GOLD");

    when(userRepository.findById(1L)).thenReturn(user);

    OrderService orderService = new OrderService(
            userRepository,
            orderRepository,
            paymentProcessor,
            discountCalculator,
            orderTimeValidator
    );

    int result = orderService.order(request);

    assertEquals(7000, result);
    verify(paymentProcessor).pay(7000);
    verify(orderRepository).save(request, 7000);
}
```

이 테스트는 실제 DB나 결제 API를 사용하지 않는다. 현재 시간에도 영향을 받지 않는다. 테스트 대상이 어떤 의존성을 사용하는지 명확하고, 실패했을 때 원인도 비교적 쉽게 찾을 수 있다.

### 12.6 개선 흐름 정리

이 예제의 핵심은 한 번에 완벽한 구조로 바꾸는 것이 아니다.

개선 흐름은 다음과 같다.

1. 테스트하고 싶은 핵심 로직을 찾는다.
2. 외부 의존성이 없는 로직부터 분리한다.
3. 시간, DB, API 같은 제어하기 어려운 요소를 인터페이스 뒤로 숨긴다.
4. 직접 생성하는 객체를 주입받도록 변경한다.
5. 기존 static 호출은 Wrapper로 감싸서 점진적으로 줄인다.
6. 분리한 단위부터 테스트를 작성한다.

이 방식은 레거시 시스템에서 안전하게 적용하기 좋다. 기존 동작을 크게 바꾸지 않으면서도 테스트 가능한 영역을 늘릴 수 있기 때문이다.

---

## 13. 우리 팀에서 적용할 기준

우리 팀의 목표는 모든 레거시 코드를 한 번에 이상적인 구조로 바꾸는 것이 아니다. 현실적인 목표는 변경하는 코드부터 테스트 가능성을 높이고, 중요한 비즈니스 로직을 보호하는 것이다.

### 13.1 신규 코드는 테스트 가능한 구조로 작성한다

신규 코드는 가능한 한 다음 기준을 적용한다.

- 메서드 내부에서 무거운 의존성을 직접 생성하지 않는다.
- static 유틸 호출을 새로 추가하는 것을 신중하게 판단한다.
- 외부 API, DB, 파일, 시간 의존성은 별도 객체 뒤로 분리한다.
- 핵심 계산 로직은 순수 Java 코드로 테스트 가능하게 만든다.
- 조건 분기가 커질 가능성이 있으면 정책 객체 분리를 고려한다.

신규 코드까지 기존 레거시 스타일을 반복하면 테스트 부채가 계속 늘어난다. 새 코드는 앞으로의 기준을 보여주는 출발점이 되어야 한다.

### 13.2 버그 수정 시 재발 방지 테스트를 추가한다

버그가 발생했다는 것은 해당 동작을 자동으로 검증하는 테스트가 없었다는 뜻일 수 있다.

버그 수정 시에는 가능하면 다음 순서로 진행한다.

1. 버그를 재현하는 테스트를 작성한다.
2. 테스트가 실패하는 것을 확인한다.
3. 코드를 수정한다.
4. 테스트가 성공하는 것을 확인한다.

이렇게 하면 같은 버그가 다시 발생했을 때 테스트가 먼저 알려준다.

### 13.3 PowerMock은 레거시 대응 수단으로 제한한다

PowerMock은 static, private, constructor 호출 등을 제어할 수 있기 때문에 레거시 시스템에서 유용하다.

하지만 PowerMock을 많이 사용한다는 것은 그만큼 코드가 테스트하기 어려운 구조라는 신호이기도 하다.

우리 팀에서는 PowerMock을 다음 기준으로 사용하는 것이 좋다.

- 당장 구조 변경이 어렵고 테스트가 꼭 필요한 경우 사용한다.
- static/private 의존을 테스트하기 위한 임시 수단으로 본다.
- 신규 코드에서는 PowerMock이 필요한 구조를 만들지 않는다.
- 가능하면 Wrapper, DI, 역할 분리를 통해 일반 Mockito로 테스트 가능한 구조로 개선한다.

PowerMock을 완전히 금지할 필요는 없다. 다만 PowerMock이 필요한 이유를 코드 구조 관점에서 함께 봐야 한다.

### 13.4 리뷰 시 테스트 가능성을 함께 본다

코드 리뷰에서는 동작이 맞는지만 볼 것이 아니라 테스트 가능한 구조인지도 함께 봐야 한다.

리뷰할 때 다음 질문을 던질 수 있다.

- 이 코드는 단위 테스트를 작성하기 쉬운가?
- 외부 의존성을 Mock으로 대체할 수 있는가?
- 메서드 내부에서 직접 생성하는 객체가 너무 많지 않은가?
- static 호출이 테스트를 어렵게 만들고 있지 않은가?
- 하나의 클래스가 너무 많은 책임을 가지고 있지 않은가?
- 새로운 조건이 추가될 때 기존 코드를 많이 수정해야 하는가?

테스트 가능성은 코드 품질의 중요한 신호다. 테스트하기 어려운 코드는 대체로 변경하기도 어렵다.

### 13.5 커버리지보다 중요한 것은 위험 영역 보호다

JaCoCo 커버리지는 유용한 지표지만 숫자 자체가 목표가 되어서는 안 된다.

우리 팀이 우선 보호해야 할 영역은 다음과 같다.

- 장애가 나면 영향이 큰 핵심 비즈니스 로직
- 자주 변경되는 로직
- 조건 분기가 복잡한 로직
- 과거에 버그가 발생했던 로직
- 배포 전 수동 확인에 시간이 많이 드는 로직

이런 영역부터 테스트를 추가하면 커버리지 숫자보다 더 실질적인 품질 개선을 얻을 수 있다.

### 13.6 작은 개선을 계속 쌓는다

레거시 리팩토링은 큰 이벤트가 아니라 일상적인 개선에 가깝다.

하나의 PR에서 모든 문제를 해결하려고 하면 리뷰도 어렵고 위험도 커진다. 대신 변경하는 코드 주변에서 작은 개선을 반복한다.

- 긴 메서드에서 계산 로직 하나를 분리한다.
- static 호출 하나를 Wrapper로 감싼다.
- 직접 생성하던 객체 하나를 주입받도록 바꾼다.
- 복잡한 조건문 하나를 정책 객체로 분리한다.
- 버그 수정 테스트 하나를 추가한다.

이런 작은 개선이 쌓이면 시스템은 점점 테스트 가능한 구조로 바뀐다.

---

## 14. 정리

테스트하기 쉬운 코드는 변경하기 쉬운 코드다.

SOLID는 이론을 위한 이론이 아니라, 변경 비용을 낮추고 테스트 가능한 구조를 만들기 위한 기준이다. 특히 레거시 시스템에서는 테스트 도구만으로 해결할 수 없는 구조적 문제가 많기 때문에 SOLID 관점이 필요하다.

중요한 것은 완벽한 설계를 한 번에 만드는 것이 아니다. 변경하는 코드부터 책임을 나누고, 의존성을 분리하고, 테스트 가능한 단위를 늘려가는 것이다.

우리 팀은 신규 코드부터 테스트 가능한 구조를 적용하고, 레거시 코드는 위험도가 높은 영역부터 점진적으로 개선한다.

---

## 세미나 중 던져볼 질문

- 이 클래스는 몇 가지 일을 하고 있나요?
- 이 코드를 테스트하려면 어떤 의존성을 제어해야 하나요?
- PowerMock이 필요한 이유가 테스트 도구 문제일까요, 코드 구조 문제일까요?
- 새로운 조건이 추가될 때 기존 코드를 얼마나 수정해야 하나요?
- 이 로직을 Mock 없이 테스트할 수 있으려면 어떤 구조가 필요할까요?
