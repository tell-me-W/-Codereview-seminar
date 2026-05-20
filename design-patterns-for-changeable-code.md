# 디자인 패턴: 변경에 강한 코드를 만드는 반복 해법

## 1. 왜 디자인 패턴이 필요한가?

디자인 패턴은 외워서 쓰는 공식이 아니다.

실무에서 반복해서 만나는 설계 문제에 이름을 붙이고, 그 문제를 해결하는 대표적인 구조를 정리한 것이다.

레거시 시스템에서는 같은 문제가 반복된다. 메시지 타입이 늘어날수록 if/else가 길어지고, 하나의 클래스가 수신, 검증, 변환, 처리, 응답까지 모두 담당하며, 객체 생성 규칙이 여기저기 흩어진다.

디자인 패턴은 이런 문제를 한 번에 마법처럼 해결하지 않는다. 대신 변경이 자주 일어나는 지점을 분리하고, 테스트 가능한 단위로 나누며, 코드를 이해할 수 있는 이름과 구조로 정리하는 데 도움을 준다.

---

## 2. 우리 레거시 시스템의 문제 상황

우리 시스템은 tibrv로 받은 메시지 객체를 여러 로직에서 계속 들고 다니며 처리하는 구조를 가지고 있다.

처음에는 이 방식이 단순해 보일 수 있다. 수신한 메시지 객체에서 필요한 값을 바로 꺼내 쓰면 별도 변환 과정이 필요 없기 때문이다.

하지만 시간이 지나면 다음 문제가 생긴다.

- 메시지 필드명을 여러 클래스가 직접 알고 있어야 한다.
- 같은 필드 파싱 로직이 여러 곳에 반복된다.
- 메시지 구조가 바뀌면 영향 범위가 넓어진다.
- 테스트에서 tibrv 메시지 객체를 만들거나 흉내 내야 한다.
- 비즈니스 로직이 메시지 포맷에 강하게 묶인다.

또 몇 개의 God Class가 여러 기능을 가지고 있고, 한 파일 안에 매우 긴 라인의 코드가 모여 있다.

God Class는 보통 다음 일을 동시에 한다.

- 메시지 수신
- 메시지 필드 검증
- 타입별 분기
- 도메인 객체 생성
- DB 조회
- 비즈니스 처리
- 외부 시스템 호출
- 응답 메시지 생성
- 로그 기록
- 예외 처리

이 구조에서는 작은 기능 하나를 수정해도 영향 범위를 판단하기 어렵다. 테스트를 작성하려고 해도 준비해야 할 조건이 많고, 특정 로직만 따로 검증하기 어렵다.

디자인 패턴은 이런 God Class를 한 번에 없애기 위한 도구가 아니다. 변경하는 부분부터 안전하게 분리하기 위한 리팩토링 도구로 보는 것이 좋다.

---

## 3. 패턴을 적용하기 전에 알아야 할 것

패턴을 많이 쓴다고 좋은 코드가 되는 것은 아니다.

오히려 문제에 맞지 않는 패턴을 억지로 적용하면 코드가 더 복잡해질 수 있다. 간단한 if문 하나면 충분한 곳에 Strategy를 만들고, 단순 생성자 하나면 충분한 곳에 Builder를 만들면 읽어야 할 코드만 늘어난다.

패턴을 적용해야 할 때는 보통 다음 신호가 있다.

- 같은 분기 조건이 여러 곳에 반복된다.
- 객체 생성 과정이 복잡하고 실수하기 쉽다.
- 하나의 메서드가 여러 처리 단계를 모두 담당한다.
- 외부 시스템 또는 레거시 API에 직접 의존해서 테스트가 어렵다.
- 새로운 기능을 추가할 때 기존 코드를 계속 수정해야 한다.

리팩토링의 목표는 구조를 멋있게 만드는 것이 아니다. 변경 범위를 줄이고, 테스트 가능성을 높이고, 코드 리뷰에서 의도를 파악하기 쉽게 만드는 것이다.

---

## 4. 빌더 패턴: 복잡한 객체 생성의 구원자

Builder Pattern은 복잡한 객체 생성 과정을 분리하는 패턴이다.

우리 시스템에서는 tibrv로 받은 메시지 객체를 계속 들고 다니는 구조를 줄이는 데 특히 유용하다.

### 4.1 메시지 객체를 계속 들고 다니는 문제

예를 들어 다음과 같은 코드가 있다고 가정해보자.

```java
public class TradeProcessor {
    public void process(TibrvMsg msg) {
        String accountNo = msg.getString("ACCOUNT_NO");
        String productCode = msg.getString("PRODUCT_CODE");
        int quantity = msg.getInt("QTY");
        BigDecimal price = new BigDecimal(msg.getString("PRICE"));
        String tradeType = msg.getString("TRADE_TYPE");

        if (accountNo == null || accountNo.isEmpty()) {
            throw new IllegalArgumentException("계좌번호가 없습니다.");
        }

        if (quantity <= 0) {
            throw new IllegalArgumentException("수량이 올바르지 않습니다.");
        }

        // 이후 비즈니스 로직
    }
}
```

처음에는 문제가 없어 보인다. 하지만 이런 코드가 여러 클래스에 퍼지면 메시지 필드명, 타입 변환, 기본값 처리, 검증 규칙이 중복된다.

또 비즈니스 로직이 `TibrvMsg`에 직접 의존한다. 테스트를 작성하려면 실제 tibrv 메시지 객체를 만들거나 복잡한 Mock을 준비해야 한다.

더 좋은 방향은 메시지 객체를 초기에 도메인 요청 객체로 변환하고, 이후 로직은 그 요청 객체를 사용하게 만드는 것이다.

```java
public class TradeRequest {
    private final String accountNo;
    private final String productCode;
    private final int quantity;
    private final BigDecimal price;
    private final TradeType tradeType;

    private TradeRequest(Builder builder) {
        this.accountNo = builder.accountNo;
        this.productCode = builder.productCode;
        this.quantity = builder.quantity;
        this.price = builder.price;
        this.tradeType = builder.tradeType;
    }

    public static Builder builder() {
        return new Builder();
    }

    public static class Builder {
        private String accountNo;
        private String productCode;
        private int quantity;
        private BigDecimal price;
        private TradeType tradeType;

        public Builder accountNo(String accountNo) {
            this.accountNo = accountNo;
            return this;
        }

        public Builder productCode(String productCode) {
            this.productCode = productCode;
            return this;
        }

        public Builder quantity(int quantity) {
            this.quantity = quantity;
            return this;
        }

        public Builder price(BigDecimal price) {
            this.price = price;
            return this;
        }

        public Builder tradeType(TradeType tradeType) {
            this.tradeType = tradeType;
            return this;
        }

        public TradeRequest build() {
            validate();
            return new TradeRequest(this);
        }

        private void validate() {
            if (accountNo == null || accountNo.isEmpty()) {
                throw new IllegalArgumentException("계좌번호가 없습니다.");
            }
            if (quantity <= 0) {
                throw new IllegalArgumentException("수량이 올바르지 않습니다.");
            }
            if (price == null) {
                throw new IllegalArgumentException("가격이 없습니다.");
            }
        }
    }
}
```

이제 tibrv 메시지에서 요청 객체를 만드는 책임을 별도 클래스로 둘 수 있다.

```java
public class TradeRequestBuilder {
    public TradeRequest from(TibrvMsg msg) {
        return TradeRequest.builder()
                .accountNo(msg.getString("ACCOUNT_NO"))
                .productCode(msg.getString("PRODUCT_CODE"))
                .quantity(msg.getInt("QTY"))
                .price(new BigDecimal(msg.getString("PRICE")))
                .tradeType(TradeType.from(msg.getString("TRADE_TYPE")))
                .build();
    }
}
```

비즈니스 로직은 이제 `TibrvMsg`가 아니라 `TradeRequest`를 받는다.

```java
public class TradeProcessor {
    public void process(TradeRequest request) {
        // 메시지 파싱이 아니라 거래 처리에 집중한다.
    }
}
```

### 4.2 Builder가 해결하는 문제

Builder를 사용하면 다음 문제가 줄어든다.

- 생성자 파라미터가 너무 길어지는 문제
- setter를 여러 번 호출하다가 필수값을 빠뜨리는 문제
- 메시지 필드 파싱 로직이 여러 곳에 흩어지는 문제
- 도메인 로직이 tibrv 메시지 구조에 직접 묶이는 문제
- 테스트에서 복잡한 메시지 객체를 준비해야 하는 문제

특히 필드가 많고 선택값이 많은 요청 객체에서는 생성자보다 Builder가 읽기 쉽다.

```java
TradeRequest request = TradeRequest.builder()
        .accountNo("123456")
        .productCode("ABC")
        .quantity(10)
        .price(new BigDecimal("1000"))
        .tradeType(TradeType.BUY)
        .build();
```

테스트 코드에서도 어떤 입력 조건을 만드는지 명확하게 보인다.

### 4.3 Builder를 사용할 때 주의할 점

Builder도 남용하면 안 된다.

필드가 2~3개뿐이고 생성 규칙이 단순한 객체라면 생성자나 정적 팩토리 메서드가 더 낫다.

Builder가 특히 유용한 경우는 다음과 같다.

- 생성해야 할 필드가 많다.
- 필수값과 선택값이 섞여 있다.
- 생성 과정에서 검증이 필요하다.
- 외부 메시지나 Map에서 값을 꺼내 도메인 객체로 변환해야 한다.
- 테스트에서 다양한 입력 조합을 자주 만들어야 한다.

우리 시스템에서는 tibrv 메시지를 직접 오래 들고 다니는 구조를 줄이기 위해 Builder를 사용할 수 있다. 메시지 수신 초기에 필요한 값을 도메인 요청 객체로 변환하고, 이후 로직은 명확한 타입을 가진 객체를 사용하게 만드는 것이 목표다.

---

## 5. 전략 패턴: 끝없는 if-else와의 전쟁

Strategy Pattern은 바뀌는 알고리즘이나 정책을 별도 클래스로 분리하는 패턴이다.

God Class 안에 메시지 타입별, 업무 구분별, 상태별 분기가 계속 늘어나는 경우 Strategy가 도움이 된다.

### 5.1 긴 if-else가 만드는 문제

예를 들어 하나의 클래스가 메시지 타입에 따라 여러 처리를 한다고 가정해보자.

```java
public class MessageGodClass {
    public void process(TibrvMsg msg) {
        String messageType = msg.getString("MSG_TYPE");

        if ("ORDER".equals(messageType)) {
            // 주문 처리
            validateOrder(msg);
            saveOrder(msg);
            sendOrderResponse(msg);
        } else if ("CANCEL".equals(messageType)) {
            // 취소 처리
            validateCancel(msg);
            cancelOrder(msg);
            sendCancelResponse(msg);
        } else if ("MODIFY".equals(messageType)) {
            // 정정 처리
            validateModify(msg);
            modifyOrder(msg);
            sendModifyResponse(msg);
        } else if ("QUERY".equals(messageType)) {
            // 조회 처리
            query(msg);
            sendQueryResponse(msg);
        } else {
            throw new IllegalArgumentException("지원하지 않는 메시지 타입입니다.");
        }
    }
}
```

이 구조는 메시지 타입이 늘어날수록 계속 커진다.

문제는 단순히 코드가 길다는 것이 아니다.

- 새로운 메시지 타입을 추가할 때 기존 God Class를 수정해야 한다.
- 한 타입의 변경이 다른 타입 로직에 영향을 줄 수 있다.
- 테스트할 때 모든 분기 조건을 고려해야 한다.
- 코드 리뷰에서 변경 범위를 파악하기 어렵다.
- 같은 타입 분기가 다른 메서드에도 반복될 수 있다.

### 5.2 Strategy로 메시지 처리 분리하기

먼저 메시지 처리 역할을 인터페이스로 정의한다.

```java
public interface MessageHandler {
    boolean supports(MessageType messageType);

    void handle(MessageContext context);
}
```

각 메시지 타입별 처리 로직은 별도 클래스로 분리한다.

```java
public class OrderMessageHandler implements MessageHandler {
    private final OrderService orderService;

    public OrderMessageHandler(OrderService orderService) {
        this.orderService = orderService;
    }

    @Override
    public boolean supports(MessageType messageType) {
        return messageType == MessageType.ORDER;
    }

    @Override
    public void handle(MessageContext context) {
        OrderRequest request = context.getOrderRequest();
        orderService.order(request);
    }
}
```

```java
public class CancelMessageHandler implements MessageHandler {
    private final CancelService cancelService;

    public CancelMessageHandler(CancelService cancelService) {
        this.cancelService = cancelService;
    }

    @Override
    public boolean supports(MessageType messageType) {
        return messageType == MessageType.CANCEL;
    }

    @Override
    public void handle(MessageContext context) {
        CancelRequest request = context.getCancelRequest();
        cancelService.cancel(request);
    }
}
```

God Class에 있던 분기 선택 책임은 Dispatcher로 옮긴다.

```java
public class MessageDispatcher {
    private final List<MessageHandler> handlers;

    public MessageDispatcher(List<MessageHandler> handlers) {
        this.handlers = handlers;
    }

    public void dispatch(MessageContext context) {
        MessageType messageType = context.getMessageType();

        MessageHandler handler = handlers.stream()
                .filter(candidate -> candidate.supports(messageType))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException("지원하지 않는 메시지 타입입니다."));

        handler.handle(context);
    }
}
```

이제 새로운 메시지 타입이 추가되면 기존 Dispatcher를 크게 수정하지 않고 새로운 `MessageHandler` 구현체를 추가할 수 있다.

### 5.3 Strategy 적용 후 좋아지는 점

Strategy를 적용하면 변경 단위가 작아진다.

주문 처리 변경은 `OrderMessageHandler`와 관련 서비스만 보면 된다. 취소 처리 변경은 `CancelMessageHandler` 쪽만 보면 된다.

테스트도 단순해진다.

```java
@Test
public void handle_orderMessage_callsOrderService() {
    OrderService orderService = mock(OrderService.class);
    OrderMessageHandler handler = new OrderMessageHandler(orderService);
    MessageContext context = MessageContext.order(orderRequest);

    handler.handle(context);

    verify(orderService).order(orderRequest);
}
```

God Class 전체를 준비하지 않고 특정 전략 하나만 테스트할 수 있다.

### 5.4 Strategy 적용 후보

우리 시스템에서 다음 신호가 보이면 Strategy 적용을 검토할 수 있다.

- 메시지 타입별 if/else가 길다.
- 업무 구분 코드에 따라 처리 로직이 나뉜다.
- 특정 조건마다 계산 방식이나 검증 방식이 다르다.
- 새로운 조건이 자주 추가된다.
- 분기마다 서로 다른 외부 시스템을 호출한다.

단, 모든 분기를 Strategy로 바꿀 필요는 없다. 조건이 적고 변경 가능성이 낮은 단순 분기는 그대로 두는 편이 더 읽기 좋다.

Strategy는 변경 가능성이 높은 분기, 계속 늘어나는 분기, 테스트를 분리하고 싶은 분기에 적용하는 것이 좋다.

---

## 6. 파이프라인/책임 연쇄 패턴: 메시지 처리 흐름의 구조화

Pipeline Pattern과 Chain of Responsibility Pattern은 긴 처리 흐름을 단계별로 나누는 데 사용한다.

두 패턴은 형태가 비슷하다. Pipeline은 정해진 단계를 순서대로 통과하는 느낌이 강하고, Chain of Responsibility는 각 Handler가 처리 여부를 판단하거나 다음 Handler로 넘기는 느낌이 강하다.

세미나에서는 둘을 메시지 처리 흐름을 구조화하는 방법으로 함께 이해하면 좋다.

### 6.1 하나의 메서드에 모든 흐름이 들어 있는 문제

레거시 God Class에는 다음과 같은 메서드가 흔하다.

```java
public void onMessage(TibrvMsg msg) {
    // 1. 로그 시작
    // 2. 공통 필드 검증
    // 3. 메시지 타입 확인
    // 4. 사용자 권한 확인
    // 5. 메시지를 요청 객체로 변환
    // 6. DB 조회
    // 7. 비즈니스 처리
    // 8. 응답 메시지 생성
    // 9. 응답 발송
    // 10. 로그 종료
    // 11. 예외 처리
}
```

이 구조에서는 특정 단계만 수정하기 어렵다. 검증 로직을 바꾸려 해도 전체 메서드를 읽어야 하고, 응답 생성 방식을 바꾸려 해도 처리 로직과 예외 처리까지 함께 얽혀 있다.

테스트도 어렵다. "권한 검증이 실패하면 비즈니스 처리를 하지 않는다" 같은 시나리오를 검증하려면 전체 메시지 처리 환경을 준비해야 할 수 있다.

### 6.2 MessageContext 만들기

파이프라인을 구성하려면 각 단계가 공유할 컨텍스트가 필요하다.

중요한 점은 `TibrvMsg`를 그대로 계속 들고 다니지 않는 것이다. 초기 단계에서 필요한 정보를 추출하고, 처리 단계에서는 점점 더 명확한 타입의 데이터를 사용하게 한다.

```java
public class MessageContext {
    private final TibrvMsg rawMessage;
    private MessageType messageType;
    private TradeRequest tradeRequest;
    private ResponseMessage responseMessage;

    public MessageContext(TibrvMsg rawMessage) {
        this.rawMessage = rawMessage;
    }

    public TibrvMsg getRawMessage() {
        return rawMessage;
    }

    public MessageType getMessageType() {
        return messageType;
    }

    public void setMessageType(MessageType messageType) {
        this.messageType = messageType;
    }

    public TradeRequest getTradeRequest() {
        return tradeRequest;
    }

    public void setTradeRequest(TradeRequest tradeRequest) {
        this.tradeRequest = tradeRequest;
    }

    public ResponseMessage getResponseMessage() {
        return responseMessage;
    }

    public void setResponseMessage(ResponseMessage responseMessage) {
        this.responseMessage = responseMessage;
    }
}
```

처음에는 현실적으로 `rawMessage`를 컨텍스트에 둘 수 있다. 하지만 목표는 모든 단계가 `rawMessage`에 직접 접근하는 것이 아니라, 앞 단계에서 변환한 명확한 객체를 뒤 단계가 사용하는 구조로 가는 것이다.

### 6.3 Pipeline 단계 나누기

각 처리 단계를 인터페이스로 정의한다.

```java
public interface MessageStep {
    void execute(MessageContext context);
}
```

공통 필드 검증 단계는 다음처럼 만들 수 있다.

```java
public class CommonValidationStep implements MessageStep {
    @Override
    public void execute(MessageContext context) {
        TibrvMsg msg = context.getRawMessage();

        if (msg.getString("MSG_TYPE") == null) {
            throw new IllegalArgumentException("메시지 타입이 없습니다.");
        }
    }
}
```

메시지 타입 파싱 단계는 다음처럼 분리한다.

```java
public class MessageTypeParsingStep implements MessageStep {
    @Override
    public void execute(MessageContext context) {
        String rawType = context.getRawMessage().getString("MSG_TYPE");
        context.setMessageType(MessageType.from(rawType));
    }
}
```

요청 객체 생성 단계에서는 Builder를 사용할 수 있다.

```java
public class TradeRequestBuildStep implements MessageStep {
    private final TradeRequestBuilder tradeRequestBuilder;

    public TradeRequestBuildStep(TradeRequestBuilder tradeRequestBuilder) {
        this.tradeRequestBuilder = tradeRequestBuilder;
    }

    @Override
    public void execute(MessageContext context) {
        TradeRequest request = tradeRequestBuilder.from(context.getRawMessage());
        context.setTradeRequest(request);
    }
}
```

비즈니스 처리 단계에서는 Strategy Dispatcher를 사용할 수 있다.

```java
public class DispatchStep implements MessageStep {
    private final MessageDispatcher dispatcher;

    public DispatchStep(MessageDispatcher dispatcher) {
        this.dispatcher = dispatcher;
    }

    @Override
    public void execute(MessageContext context) {
        dispatcher.dispatch(context);
    }
}
```

파이프라인 실행기는 단계들을 순서대로 실행한다.

```java
public class MessagePipeline {
    private final List<MessageStep> steps;

    public MessagePipeline(List<MessageStep> steps) {
        this.steps = steps;
    }

    public void execute(MessageContext context) {
        for (MessageStep step : steps) {
            step.execute(context);
        }
    }
}
```

이제 메시지 수신부는 전체 처리 흐름을 간단히 표현할 수 있다.

```java
public class MessageListener {
    private final MessagePipeline pipeline;

    public MessageListener(MessagePipeline pipeline) {
        this.pipeline = pipeline;
    }

    public void onMessage(TibrvMsg msg) {
        MessageContext context = new MessageContext(msg);
        pipeline.execute(context);
    }
}
```

### 6.4 Chain of Responsibility로 중단 가능한 흐름 만들기

Pipeline은 보통 모든 단계를 순서대로 실행한다. 하지만 어떤 단계에서 처리 완료 또는 중단을 판단해야 한다면 Chain of Responsibility가 더 적합할 수 있다.

예를 들어 중복 메시지는 더 이상 비즈니스 처리하지 않고 응답만 보내야 할 수 있다.

```java
public interface MessageHandlerChain {
    void handle(MessageContext context, Chain chain);

    interface Chain {
        void next(MessageContext context);
    }
}
```

```java
public class DuplicateMessageHandler implements MessageHandlerChain {
    private final DuplicateMessageChecker duplicateMessageChecker;

    public DuplicateMessageHandler(DuplicateMessageChecker duplicateMessageChecker) {
        this.duplicateMessageChecker = duplicateMessageChecker;
    }

    @Override
    public void handle(MessageContext context, Chain chain) {
        if (duplicateMessageChecker.isDuplicate(context)) {
            context.setResponseMessage(ResponseMessage.duplicate());
            return;
        }

        chain.next(context);
    }
}
```

이 방식은 각 Handler가 다음 단계로 넘길지, 현재 단계에서 멈출지 결정할 수 있다.

### 6.5 Pipeline과 Chain을 선택하는 기준

둘 중 무엇을 써야 하는지는 처리 흐름의 성격에 따라 결정한다.

Pipeline이 어울리는 경우:

- 대부분의 메시지가 같은 단계를 순서대로 거친다.
- 검증, 변환, 처리, 응답 생성 흐름이 명확하다.
- 각 단계가 독립적으로 테스트되면 좋다.

Chain of Responsibility가 어울리는 경우:

- 중간 Handler가 처리 여부를 판단한다.
- 조건에 따라 다음 단계로 넘기지 않을 수 있다.
- 예외, 중복, 권한 실패 같은 조기 종료 흐름이 중요하다.

우리 시스템에서는 기본 메시지 처리 흐름은 Pipeline으로 보고, 중간에 흐름을 멈추거나 대체 처리해야 하는 부분은 Chain of Responsibility 관점으로 볼 수 있다.

---

## 7. Facade Pattern: 복잡한 하위 기능 앞의 단순한 진입점

Facade Pattern은 복잡한 하위 시스템을 단순한 인터페이스 뒤로 숨기는 패턴이다.

God Class가 여러 하위 기능을 직접 호출하고 있다면 Facade가 도움이 될 수 있다.

예를 들어 메시지 처리 중 다음 작업이 여러 곳에서 반복된다고 가정해보자.

- 고객 정보 조회
- 계좌 상태 확인
- 상품 정보 조회
- 거래 가능 여부 확인
- 감사 로그 기록

God Class가 이 모든 서비스를 직접 호출하면 흐름이 복잡해진다.

```java
Customer customer = customerService.findCustomer(customerId);
Account account = accountService.findAccount(accountNo);
Product product = productService.findProduct(productCode);
riskService.validate(customer, account, product);
auditService.write(customer, account, product);
```

관련된 조회와 검증을 하나의 진입점으로 묶을 수 있다.

```java
public class TradeValidationFacade {
    private final CustomerService customerService;
    private final AccountService accountService;
    private final ProductService productService;
    private final RiskService riskService;

    public TradeValidationResult validate(TradeRequest request) {
        Customer customer = customerService.findCustomer(request.getCustomerId());
        Account account = accountService.findAccount(request.getAccountNo());
        Product product = productService.findProduct(request.getProductCode());

        riskService.validate(customer, account, product);

        return new TradeValidationResult(customer, account, product);
    }
}
```

Facade는 내부 복잡도를 없애는 것이 아니라, 사용하는 쪽에서 알아야 할 복잡도를 줄인다.

세미나에서는 Facade를 God Class를 더 큰 God Class로 만드는 도구로 쓰면 안 된다는 점을 강조해야 한다. Facade는 관련 있는 하위 기능을 묶는 단순한 진입점이어야 한다.

---

## 8. Adapter Pattern: tibrv와 레거시 API를 경계 밖으로 밀어내기

Adapter Pattern은 기존 인터페이스를 우리가 사용하기 좋은 인터페이스로 변환하는 패턴이다.

우리 시스템에서는 tibrv 메시지 객체, 레거시 static 유틸, 외부 API 클라이언트처럼 바꾸기 어려운 대상이 Adapter 후보가 된다.

예를 들어 여러 로직이 `TibrvMsg`에서 직접 값을 꺼내고 있다면, 메시지 접근을 Adapter 뒤로 감쌀 수 있다.

```java
public interface MessageReader {
    String getString(String fieldName);

    int getInt(String fieldName);
}
```

```java
public class TibrvMessageReader implements MessageReader {
    private final TibrvMsg msg;

    public TibrvMessageReader(TibrvMsg msg) {
        this.msg = msg;
    }

    @Override
    public String getString(String fieldName) {
        return msg.getString(fieldName);
    }

    @Override
    public int getInt(String fieldName) {
        return msg.getInt(fieldName);
    }
}
```

이제 Builder는 `TibrvMsg`가 아니라 `MessageReader`에 의존할 수 있다.

```java
public class TradeRequestBuilder {
    public TradeRequest from(MessageReader reader) {
        return TradeRequest.builder()
                .accountNo(reader.getString("ACCOUNT_NO"))
                .productCode(reader.getString("PRODUCT_CODE"))
                .quantity(reader.getInt("QTY"))
                .price(new BigDecimal(reader.getString("PRICE")))
                .tradeType(TradeType.from(reader.getString("TRADE_TYPE")))
                .build();
    }
}
```

테스트에서는 tibrv 없이 간단한 Fake Reader를 사용할 수 있다.

```java
public class FakeMessageReader implements MessageReader {
    private final Map<String, String> values;

    public FakeMessageReader(Map<String, String> values) {
        this.values = values;
    }

    @Override
    public String getString(String fieldName) {
        return values.get(fieldName);
    }

    @Override
    public int getInt(String fieldName) {
        return Integer.parseInt(values.get(fieldName));
    }
}
```

Adapter의 목표는 외부 기술을 없애는 것이 아니다. 외부 기술에 직접 의존하는 범위를 줄이는 것이다.

tibrv는 메시지 수신 경계에서만 알고, 비즈니스 로직은 `TradeRequest`, `MessageContext`, `MessageReader` 같은 내부 모델에 의존하게 만드는 것이 좋다.

---

## 9. God Class를 점진적으로 분해하는 순서

God Class는 한 번에 없애려고 하면 위험하다.

이미 많은 기능이 들어 있고 운영 중인 코드라면, 큰 리팩토링은 장애 가능성을 높인다. 따라서 변경이 필요한 부분부터 작은 단위로 분리해야 한다.

### 9.1 메시지 읽기 분리

먼저 tibrv 메시지 직접 접근을 줄인다.

기존 코드:

```java
String accountNo = msg.getString("ACCOUNT_NO");
int quantity = msg.getInt("QTY");
```

개선 방향:

```java
MessageReader reader = new TibrvMessageReader(msg);
String accountNo = reader.getString("ACCOUNT_NO");
int quantity = reader.getInt("QTY");
```

이 단계에서는 큰 구조를 바꾸지 않아도 된다. 메시지 접근 경계를 하나 만드는 것이 목표다.

### 9.2 요청 객체 생성 분리

다음으로 메시지 필드를 도메인 요청 객체로 변환한다.

```java
TradeRequest request = tradeRequestBuilder.from(reader);
```

이후 로직은 가능하면 `msg`나 `reader`가 아니라 `TradeRequest`를 사용한다.

이 단계에서 Builder Pattern을 적용할 수 있다.

### 9.3 분기 로직을 Strategy로 분리

God Class 내부의 메시지 타입별 분기를 Handler로 분리한다.

```java
messageDispatcher.dispatch(context);
```

새로운 메시지 타입이 추가될 때 기존 God Class의 if/else를 수정하는 대신 새로운 Handler를 추가하는 구조로 바꾼다.

이 단계에서 Strategy Pattern을 적용할 수 있다.

### 9.4 처리 흐름을 Pipeline 또는 Chain으로 분리

긴 메서드를 처리 단계로 나눈다.

```java
MessagePipeline pipeline = new MessagePipeline(Arrays.asList(
        new CommonValidationStep(),
        new MessageTypeParsingStep(),
        new TradeRequestBuildStep(tradeRequestBuilder),
        new DispatchStep(messageDispatcher),
        new ResponseSendStep(responseSender)
));
```

검증, 변환, 처리, 응답 생성 같은 흐름을 단계별로 나누면 각 단계의 책임이 명확해진다.

### 9.5 레거시 외부 의존성은 Adapter와 Facade로 감싼다

레거시 static 유틸, 외부 API, 복잡한 하위 서비스 호출은 Adapter 또는 Facade 뒤로 숨긴다.

```java
public interface LegacyRiskChecker {
    void check(TradeRequest request);
}
```

```java
public class LegacyRiskCheckerAdapter implements LegacyRiskChecker {
    @Override
    public void check(TradeRequest request) {
        LegacyRiskUtil.check(request.getAccountNo(), request.getProductCode());
    }
}
```

이렇게 하면 신규 로직은 레거시 구현 세부사항이 아니라 내부에서 정의한 인터페이스에 의존할 수 있다.

### 9.6 분해 순서 정리

God Class를 분해할 때 추천 순서는 다음과 같다.

1. 메시지 직접 접근을 Adapter로 감싼다.
2. 메시지 값을 도메인 요청 객체로 변환한다.
3. 객체 생성 규칙은 Builder로 모은다.
4. 타입별 분기는 Strategy로 나눈다.
5. 긴 처리 흐름은 Pipeline 또는 Chain으로 나눈다.
6. 복잡한 하위 시스템 호출은 Facade로 묶는다.
7. 레거시 static/API 의존은 Adapter 뒤로 숨긴다.

중요한 것은 한 PR에서 모든 단계를 끝내려 하지 않는 것이다. 변경 대상 주변부터 조금씩 분리하고, 분리한 단위에 테스트를 추가하는 방식이 안전하다.

---

## 10. 우리 팀에서 적용할 기준

우리 팀에서는 디자인 패턴을 다음 기준으로 적용한다.

### 10.1 신규 로직은 메시지 객체를 직접 오래 들고 다니지 않는다

tibrv 메시지는 수신 경계에서 내부 모델로 변환한다.

신규 비즈니스 로직은 가능하면 `TibrvMsg`가 아니라 요청 객체, 컨텍스트, 도메인 객체를 사용한다.

### 10.2 긴 if-else가 늘어나면 Strategy 후보로 본다

메시지 타입, 업무 구분, 거래 유형에 따른 분기가 계속 늘어나면 Strategy 적용을 검토한다.

특히 새로운 조건이 추가될 때마다 기존 God Class를 수정해야 한다면 분리할 시점이다.

### 10.3 처리 단계가 길어지면 Pipeline 또는 Chain 후보로 본다

검증, 변환, 조회, 처리, 응답 생성이 한 메서드에 계속 쌓이면 Pipeline으로 나눌 수 있다.

중간 단계에서 흐름을 멈추거나 다른 처리로 전환해야 한다면 Chain of Responsibility를 고려한다.

### 10.4 tibrv나 레거시 API는 Adapter 뒤로 숨긴다

외부 기술이나 레거시 API를 비즈니스 로직 곳곳에서 직접 사용하지 않는다.

Adapter를 사용하면 테스트에서 Fake나 Mock으로 대체하기 쉬워지고, 외부 API 변경의 영향 범위도 줄어든다.

### 10.5 God Class는 한 번에 없애지 않는다

God Class를 한 번에 갈아엎는 리팩토링은 위험하다.

다음 기준으로 작은 개선을 반복한다.

- 이번 변경에 필요한 메시지 필드 접근만 분리한다.
- 이번 변경에 필요한 요청 객체만 만든다.
- 이번 변경에 관련된 분기 하나만 Handler로 분리한다.
- 이번 변경에 관련된 처리 단계 하나만 Step으로 분리한다.
- 분리한 단위부터 테스트를 작성한다.

### 10.6 패턴 이름보다 문제 해결을 우선한다

리뷰에서 중요한 질문은 "어떤 패턴을 썼는가?"가 아니다.

다음 질문이 더 중요하다.

- 변경 범위가 줄었는가?
- 테스트하기 쉬워졌는가?
- 메시지 포맷 의존성이 줄었는가?
- God Class의 책임이 줄었는가?
- 새 기능 추가 시 기존 코드를 덜 수정하게 되었는가?

패턴은 목적이 아니라 수단이다.

---

## 11. 정리

디자인 패턴은 변경에 강한 코드를 만들기 위한 반복 해법이다.

우리 시스템처럼 tibrv 메시지 객체를 계속 들고 다니고, 몇 개의 God Class가 긴 로직을 담당하는 구조에서는 패턴을 다음처럼 바라볼 수 있다.

- Builder는 입력을 정리한다.
- Strategy는 분기를 정리한다.
- Pipeline/Chain은 흐름을 정리한다.
- Facade와 Adapter는 레거시 경계를 정리한다.

Builder를 사용하면 복잡한 메시지 기반 입력을 명확한 요청 객체로 바꿀 수 있다.

Strategy를 사용하면 끝없이 늘어나는 if/else를 메시지 타입별 처리 클래스로 나눌 수 있다.

Pipeline 또는 Chain of Responsibility를 사용하면 긴 메시지 처리 메서드를 검증, 변환, 처리, 응답 같은 단계로 분리할 수 있다.

Facade와 Adapter를 사용하면 복잡한 하위 시스템과 바꾸기 어려운 레거시 API를 내부 코드로부터 격리할 수 있다.

중요한 것은 패턴을 많이 적용하는 것이 아니다. 변경이 자주 발생하는 지점을 찾고, 그 지점을 더 작고 테스트 가능한 구조로 분리하는 것이다.

---

## 세미나 중 던져볼 질문

- 이 로직은 메시지 객체가 꼭 필요한가요, 아니면 요청 객체로 바꿀 수 있나요?
- 이 if/else는 앞으로 더 늘어날 가능성이 있나요?
- 새로운 메시지 타입이 추가되면 기존 God Class를 얼마나 수정해야 하나요?
- 이 처리 흐름은 검증, 변환, 처리, 응답 단계로 나눌 수 있나요?
- tibrv나 레거시 API를 직접 쓰지 않으려면 어떤 Adapter가 필요할까요?
- 이 패턴을 적용하면 테스트가 더 쉬워지나요?
