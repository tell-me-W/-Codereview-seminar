# Refactoring.Guru 리팩토링 카탈로그 정리

## 정리 기준

- 출처: [Refactoring.Guru - Catalog of Refactoring](https://refactoring.guru/ko/refactoring/catalog)
- 원문 카탈로그의 분류와 항목명을 기준으로 정리한다.
- 세부 설명은 원문을 참고하고, 이 문서는 빠르게 항목을 훑기 위한 목록으로 사용한다.

---

## 1. Code Smells

Code Smells는 코드에서 리팩토링 필요성을 의심해볼 수 있는 신호들이다.

### 1.1 Bloaters

코드, 메서드, 클래스가 커져서 이해하고 변경하기 어려워지는 냄새.

- Long Method
- Large Class
- Primitive Obsession
- Long Parameter List
- Data Clumps

### 1.2 Object-Orientation Abusers

객체지향 원칙을 불완전하거나 부적절하게 적용했을 때 나타나는 냄새.

- Alternative Classes with Different Interfaces
- Refused Bequest
- Switch Statements
- Temporary Field

### 1.3 Change Preventers

한 곳을 바꾸려 할 때 여러 곳을 함께 수정하게 만드는 냄새.

- Divergent Change
- Parallel Inheritance Hierarchies
- Shotgun Surgery

### 1.4 Dispensables

없어져도 코드가 더 단순하고 이해하기 쉬워질 수 있는 불필요한 요소.

- Comments
- Duplicate Code
- Data Class
- Dead Code
- Lazy Class
- Speculative Generality

### 1.5 Couplers

클래스 간 결합이 과도하거나, 위임 구조가 불필요하게 복잡해졌을 때 나타나는 냄새.

- Feature Envy
- Inappropriate Intimacy
- Incomplete Library Class
- Message Chains
- Middle Man

---

## 2. Refactoring Techniques

Refactoring Techniques는 코드의 구조를 개선하기 위해 사용할 수 있는 대표적인 리팩토링 기법들이다.

### 2.1 Composing Methods

긴 메서드를 읽기 쉬운 단위로 나누고, 중복과 복잡도를 줄이는 기법.

- Extract Method
- Inline Method
- Extract Variable
- Inline Temp
- Replace Temp with Query
- Split Temporary Variable
- Remove Assignments to Parameters
- Replace Method with Method Object
- Substitute Algorithm

### 2.2 Moving Features between Objects

책임이 잘못 배치된 메서드나 필드를 옮기고, 클래스 경계를 재정리하는 기법.

- Move Method
- Move Field
- Extract Class
- Inline Class
- Hide Delegate
- Remove Middle Man
- Introduce Foreign Method
- Introduce Local Extension

### 2.3 Organizing Data

데이터 표현을 더 명확하게 만들고, primitive나 연관 관계를 다루기 쉽게 정리하는 기법.

- Change Value to Reference
- Change Reference to Value
- Duplicate Observed Data
- Self Encapsulate Field
- Replace Data Value with Object
- Replace Array with Object
- Change Unidirectional Association to Bidirectional
- Change Bidirectional Association to Unidirectional
- Encapsulate Field
- Encapsulate Collection
- Replace Magic Number with Symbolic Constant
- Replace Type Code with Class
- Replace Type Code with Subclasses
- Replace Type Code with State/Strategy
- Replace Subclass with Fields

### 2.4 Simplifying Conditional Expressions

조건식이 길어지고 중첩될 때 의도를 드러내고 분기를 단순화하는 기법.

- Consolidate Conditional Expression
- Consolidate Duplicate Conditional Fragments
- Decompose Conditional
- Replace Conditional with Polymorphism
- Remove Control Flag
- Replace Nested Conditional with Guard Clauses
- Introduce Null Object
- Introduce Assertion

### 2.5 Simplifying Method Calls

메서드 호출과 인터페이스를 더 명확하고 사용하기 쉽게 정리하는 기법.

- Add Parameter
- Remove Parameter
- Rename Method
- Separate Query from Modifier
- Parameterize Method
- Introduce Parameter Object
- Preserve Whole Object
- Remove Setting Method
- Replace Parameter with Explicit Methods
- Replace Parameter with Method Call
- Hide Method
- Replace Constructor with Factory Method
- Replace Error Code with Exception
- Replace Exception with Test

### 2.6 Dealing with Generalization

상속, 추상화, 위임 구조를 다루며 일반화 수준을 조정하는 기법.

- Pull Up Field
- Pull Up Method
- Pull Up Constructor Body
- Push Down Field
- Push Down Method
- Extract Subclass
- Extract Superclass
- Extract Interface
- Collapse Hierarchy
- Form Template Method
- Replace Inheritance with Delegation
- Replace Delegation with Inheritance

---

## 3. 실무에서 먼저 살펴볼 만한 항목

레거시 코드 개선을 시작할 때는 아래 항목들이 자주 연결된다.

### 긴 메서드와 God Class

- Long Method
- Large Class
- Extract Method
- Extract Class
- Move Method

### 긴 조건 분기

- Switch Statements
- Decompose Conditional
- Replace Nested Conditional with Guard Clauses
- Replace Conditional with Polymorphism
- Replace Type Code with State/Strategy

### 데이터와 파라미터 복잡도

- Primitive Obsession
- Long Parameter List
- Data Clumps
- Introduce Parameter Object
- Replace Data Value with Object

### 결합도와 위임 구조

- Feature Envy
- Message Chains
- Middle Man
- Hide Delegate
- Remove Middle Man
- Replace Inheritance with Delegation
