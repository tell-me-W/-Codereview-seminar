# 코드 품질 세미나 활동 요약

코드 품질을 주제로 진행한 5개 세미나의 활동 중 어려웠던 점과 각 세미나의 주요 내용을 정리한다.

| 세미나 Markdown | 활동 중 어려웠던 점 | 세미나 내용 요약 |
|---|---|---|
| `code-review.md` | 코드리뷰의 필요성을 개인의 취향이나 지적이 아닌 팀의 품질 활동으로 설명하는 것이 어려웠다. | 코드리뷰의 필요성과 우선 확인 항목을 살펴보고, PR 템플릿·체크리스트·Suggestion 등 GitHub를 활용한 리뷰 방법을 다룬다. |
| `junit-testcode-and-jacoco.md` | 테스트가 거의 없는 레거시 환경과 낮은 JDK 버전, static/private 의존성 때문에 테스트 작성이 어려웠다. | 테스트 코드의 필요성과 FIRST 원칙을 설명하고, JUnit·Mock·PowerMock의 역할과 JaCoCo 커버리지를 해석하는 방법을 다룬다. |
| `testable-code-solid-legacy-refactoring.md` | God Class와 강한 결합을 한 번에 변경하지 않고 안전하게 분리하는 방법을 전달하기 어려웠다. | 응집도와 결합도를 기준으로 SOLID 원칙을 이해하고, 테스트하기 어려운 레거시 코드를 작은 단계로 분리하는 리팩토링 방법을 다룬다. |
| `design-patterns-for-changeable-code.md` | tibrv 메시지와 긴 분기 및 처리 흐름에 적절한 패턴을 선택하면서 과도한 설계를 피해야 했다. | Builder로 입력 객체 생성을 정리하고, Strategy로 조건 분기를 분리하며, Pipeline·Chain·Facade·Adapter로 메시지 처리 흐름과 레거시 경계를 구조화하는 방법을 다룬다. |
| `code-smells-ko.md` | 코드 스멜을 무조건 수정해야 하는 문제로 오해하지 않도록 설명하는 것이 어려웠다. | 긴 메서드와 큰 클래스 등 주요 코드 스멜을 유형별로 살펴보고, 스멜의 원인과 변경 위험을 바탕으로 리팩토링 우선순위를 판단하는 방법을 다룬다. |
