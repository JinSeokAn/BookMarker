# 1.1 마이크로서비스와 분산 데이터 관리 문제

단일체 애플리케이션에서는 관계형 데이터베이스를 사용하며, 이로 인해 ACID 트랜잭션의 이점을 누릴 수 있습니다. 이는 여러 중요한 작업 특성을 제공합니다：

1. 원자성(Atomicity) - 모든 변경 사항이 원자적입니다.
2. 일관성(Consistency) - 데이터베이스 상태가 항상 일관성 있습니다.
3. 격리성(Isolation) - 트랜잭션이 동시에 실행되더라도 순차적으로 실행되는 것처럼 보입니다.
4. 지속성(Durability) - 트랜잭션이 커밋되면 롤백할 수 없습니다.

이러한 특성 덕분에 애플리케이션은 트랜잭션을 시작하고, 많은 행을 변경(삽입, 삭제, 업데이트)한 후, 이러한 트랜잭션을 커밋하는 것으로 간단화될 수 있습니다.

관계형 데이터베이스를 사용하는 또 다른 장점은 SQL(강력하고 선언적이며, 테이블 변환 쿼리 언어) 지원을 제공한다는 것입니다. 사용자는 쿼리를 통해 여러 테이블의 데이터를 쉽게 결합할 수 있으며, RDBMS 쿼리 스케줄러가 최적의 실행 방법을 결정합니다. 모든 애플리케이션 데이터가 하나의 데이터베이스에 있기 때문에 쿼리가 용이합니다.

그러나, 마이크로서비스 아키텍처에서는 데이터 접근이 매우 복잡해집니다. 이는 데이터가 마이크로서비스의 사적인 것이며, 유일한 접근 방법은 API를 통하는 것입니다. 이러한 데이터 접근 방식으로 인해 마이크로서비스 간에는 느슨한 결합이 이루어지며 서로 독립적입니다. 여러 서비스가 동일한 데이터에 접근하는 경우, 스키마는 접근 시간을 업데이트하고 모든 서비스 간에 조정을 진행합니다.

더욱이, 다양한 마이크로서비스는 종종 다른 데이터베이스를 사용합니다. 애플리케이션은 다양한 데이터를 생성하며, 관계형 데이터베이스가 항상 최선의 선택은 아닙니다. 일부 시나리오에서는 NoSQL 데이터베이스가 더 편리한 데이터 모델을 제공하며, 더 나은 성능과 확장성을 제공할 수 있습니다. 예를 들어, 문자열 생성 및 조회 애플리케이션은 Elasticsearch와 같은 문자 검색 엔진을 사용할 수 있습니다. 따라서 마이크로서비스 기반 애플리케이션은 일반적으로 SQL과 NoSQL을 결합한 데이터베이스, 즉 폴리글랏 지속성(polyglot persistence) 방법을 사용합니다.

분할된, 폴리글랏 지속성 아키텍처는 데이터 저장에 많은 이점을 제공하지만, 분산 데이터 관리에 따른 도전 과제도 수반됩니다.

첫 번째 도전 과제는 여러 서비스 간에 데이터 일관성을 유지하면서 트랜잭션을 완료하는 방법입니다. 온라인 B2B 상점을 예로 들어보겠습니다. 고객 서비스는 고객의 다양한 정보(예: 신용 한도)를 유지 관리하고, 주문 서비스는 주문을 관리하며, 새 주문이 고객의 신용 한도와 충돌하지 않는지 확인해야 합니다. 단일체 애플리케이션에서는 주문 서비스가 ACID 트랜잭션을 사용하여 사용 가능한 신용을 확인하고 주문을 생성하기만 하면 됩니다.

반면에, 마이크로서비스 아키텍처에서는 주문과 고객 테이블이 각각 해당 서비스의 사적인 테이블이며, 아래 그림과 같습니다.

![service table](./images/Private-table-of-the-corresponding-service.png)

주문 서비스는 고객 테이블에 직접 접근할 수 없으며, 고객 서비스가 제공하는 API를 통해서만 접근할 수 있습니다. 주문 서비스는 분산 트랜잭션, 즉 잘 알려진 2단계 커밋(2PC)을 사용할 수도 있습니다. 그러나, 2PC는 현대의 애플리케이션에서는 선택사항이 아닙니다. CAP 이론에 따르면, 가용성(availability)과 ACID 일관성(consistency) 사이에서 선택해야 하며, 가용성이 일반적으로 더 좋은 선택입니다. 그러나 많은 현대 기술, 예를 들어 많은 NoSQL 데이터베이스는 2PC를 지원하지 않습니다. 서비스와 데이터베이스 간의 데이터 일관성을 유지하는 것은 매우 근본적인 요구사항이므로, 다른 방안을 찾아야 합니다.

두 번째 도전 과제는 여러 서비스로부터 데이터를 검색하는 방법입니다. 예를 들어, 애플리케이션이 고객과 그의 주문을 표시해야 한다고 가정해 보겠습니다. 주문 서비스가 사용자 주문 정보를 수락하는 API를 제공한다면, 사용자는 애플리케이션 형식의 join 연산을 사용하여 데이터를 수신할 수 있습니다. 애플리케이션은 사용자 서비스로부터 사용자 정보를 받고, 주문 서비스로부터 해당 사용자의 주문을 받습니다. 주문 서비스가 주요 키를 기반으로 주문을 조회하는 것만 지원한다고 가정할 때(예를 들어, 주요 키 기반 조회만 지원하는 NoSQL 데이터베이스를 사용하는 경우), 필요한 데이터를 받는 적절한 방법이 없습니다.

# 1.2 이벤트 기반 아키텍처

많은 애플리케이션에 대한 해결책은 이벤트 기반 아키텍처(event-driven architecture)를 사용하는 것입니다. 이 아키텍처에서는 중요한 일이 발생할 때마다, 예를 들어 비즈니스 엔티티를 업데이트할 때, 마이크로서비스가 이벤트를 발행합니다. 이 이벤트를 구독하는 마이크로서비스가 이 이벤트를 수신하면, 자신의 비즈니스 엔티티를 업데이트하고 추가 이벤트를 발행할 수 있습니다.

이벤트를 사용하여 여러 서비스에 걸친 비즈니스 트랜잭션을 구현할 수 있습니다. 트랜잭션은 일련의 단계로 구성되며, 각 단계는 비즈니스 엔티티를 업데이트하는 마이크로서비스와 다음 단계를 활성화하는 이벤트를 발행하는 것으로 구성됩니다. 아래 그림은 주문 생성시 신용 가능성을 확인하는 방법을 이벤트 기반 방식으로 보여줍니다. 마이크로서비스는 메시지 브로커를 통해 이벤트를 교환합니다.

1. 주문 서비스는 NEW 상태의 Order(주문)을 생성하고 "Order Created Event(주문 생성 이벤트)"를 발행합니다.

![Order-Created-Event](./images/Order-Created-Event.png)

2. 고객 서비스는 Order Created Event를 소비하여 이 주문에 대한 신용을 예약하고 "Credit Reserved Event(신용 예약 이벤트)"를 발행합니다.

![Credit-Reserved-Event](./images/Credit-Reserved-Event.png)

3. 주문 서비스는 Credit Reserved Event를 소비하여 주문 상태를 OPEN으로 변경합니다.

![Status-is-OPEN](./images/Status-is-OPEN.png)

더 복잡한 시나리오는 추가 단계를 도입할 수 있습니다. 예를 들어, 사용자 신용을 확인하는 동안 재고를 예약하는 것과 같습니다.

(a) 각 서비스가 데이터베이스를 원자적으로 업데이트하고 이벤트를 발행하고, (b) 메시지 브로커가 이벤트를 최소 한 번 전달함을 보장한다면, 여러 서비스에 걸친 비즈니스 트랜잭션(이 트랜잭션은 ACID 트랜잭션이 아님)을 완료할 수 있습니다. 이러한 패턴은 약한 결정성, 예를 들어 최종 일관성 eventual consistency을 제공합니다. 이러한 트랜잭션 유형은 BASE 모델이라고 합니다.

이벤트를 사용하여 서로 다른 마이크로서비스가 소유한 데이터의 사전 조인(pre-join) 구현 뷰를 유지 관리할 수도 있습니다. 이 뷰를 유지 관리하는 서비스는 관련 이벤트를 구독하고 뷰를 업데이트합니다. 예를 들어, 고객 주문 뷰 업데이트 서비스(고객 주문 뷰를 유지 관리함)는 고객 서비스와 주문 서비스가 발행하는 이벤트를 구독합니다.

![pre-join](./images/pre-join.png)

고객 주문 뷰 업데이트 서비스가 고객 또는 주문 이벤트를 수신하면 고객 주문 뷰 데이터 세트를 업데이트합니다. 고객 주문 뷰는 문서 데이터베이스(예: MongoDB)를 사용하여 구현할 수 있으며, 각 사용자에 대한 문서를 저장합니다. 고객 주문 뷰 조회 서비스는 고객 및 최근 주문(고객 주문 뷰 데이터 세트를 조회하여)에 대한 조회를 처리합니다.

이벤트 기반 아키텍처는 장단점이 있습니다. 이 아키텍처는 트랜잭션이 여러 서비스에 걸쳐 있으면서 최종 일관성을 제공할 수 있으며, 애플리케이션이 최종 뷰를 유지 관리할 수 있게 합니다. 그러나 프로그래밍 모델이 ACID 트랜잭션 모델보다 복잡합니다. 애플리케이션 수준의 실패에서 복구하기 위해 보상 트랜잭션(예: 신용 검사가 실패하면 주문을 취소해야 함)을 완료해야 합니다. 또한, 애플리케이션은 데이터 불일치에 대처해야 합니다. 이는 비행 중(in-flight) 트랜잭션으로 인한 변경 사항이 보이기 때문입니다. 또한, 애플리케이션이 업데이트되지 않은 최종 뷰를 읽을 때 데이터 불일치 문제에 직면할 수 있습니다. 또 다른 단점은 구독자가 중복 이벤트를 감지하고 무시해야 한다는 것입니다.

# 1.3 원자성 달성

이벤트 기반 아키텍처에서 데이터베이스 업데이트와 이벤트 발행의 원자성 문제도 발생합니다. 예를 듭니다. 주문 서비스는 ORDER 테이블에 행을 삽입한 후 Order Created 이벤트를 발행해야 합니다. 이 두 작업은 원자적이어야 합니다. 데이터베이스를 업데이트한 후 서비스가 실패하여 이벤트가 발행되지 않으면 시스템이 일관성 없는 상태가 됩니다. 원자적 작업을 보장하는 표준 방법은 데이터베이스와 메시지 브로커를 포함하는 분산 트랜잭션을 사용하는 것입니다. 그러나 앞서 설명한 CAP 이론을 바탕으로, 이는 우리가 원하는 것이 아닙니다.

## 1.3.1 로컬 트랜잭션을 사용한 이벤트 발행

원자성을 달성하는 한 가지 방법은 로컬 트랜잭션만을 포함하는 다단계 프로세스를 이벤트 발행에 적용하는 것입니다. 핵심은 비즈니스 엔티티 데이터베이스 내에 메시지 목록 역할을 하는 EVENT 테이블입니다. 애플리케이션은 (로컬) 데이터베이스 트랜잭션을 시작하고, 비즈니스 엔티티 상태를 업데이트하며, EVENT 테이블에 이벤트를 삽입한 후 이 트랜잭션을 커밋합니다. 독립적인 애플리케이션 프로세스나 스레드가 EVENT 테이블을 쿼리하고, 메시지 브로커로 이벤트를 발행한 다음, 로컬 트랜잭션을 사용하여 이 이벤트를 발행된 것으로 표시합니다.

![multi-step process](./images/multi-step-process.png)

주문 서비스는 ORDER 테이블에 행을 삽입하고, EVENT 테이블에 Order Created 이벤트를 삽입합니다. 이벤트 발행 스레드나 프로세스가 EVENT 테이블을 쿼리하고, 발행되지 않은 이벤트를 요청하고, 이를 발행한 다음, EVENT 테이블을 업데이트하여 이 이벤트를 발행된 것으로 표시합니다.

이 방법은 장단점이 있습니다. 장점은 이벤트 발행이 2PC에 의존하지 않고 보장된다는 것이며, 애플리케이션이 비즈니스 수준 이벤트를 발행할 수 있으며, 발생한 사항을 추론할 필요가 없다는 것입니다. 단점은 개발자가 이벤트를 발행해야 한다는 것을 기억해야 하므로 오류가 발생할 수 있다는 것입니다. 또한, NoSQL 데이터베이스를 사용하는 일부 애플리케이션에 대한 도전 과제가 될 수 있습니다. NoSQL은 본질적으로 트랜잭션과 쿼리 능력이 제한됩니다.

이 방법은 애플리케이션이 로컬 트랜잭션을 사용하여 상태를 업데이트하고 이벤트를 발행하므로 2PC가 필요하지 않습니다. 이제 다른 방법을 살펴보겠습니다. 상태를 단순히 업데이트하여 원자성을 달성하는 방법입니다.

## 1.3.2 데이터베이스 트랜잭션 로그 마이닝

2PC 없이도 스레드나 프로세스가 이벤트를 원자적으로 발행할 수 있는 또 다른 방법은 데이터베이스 트랜잭션 또는 커밋 로그를 마이닝하는 것입니다. 애플리케이션은 데이터베이스를 업데이트하고, 데이터베이스 트랜잭션 로그에서 변경 사항이 발생하며, 트랜잭션 로그 마이닝 프로세스나 스레드가 이러한 트랜잭션 로그를 읽고, 이를 메시지 브로커로 발행합니다.

![No-2PC-required](./images/No-2PC-required.png)

이 방법의 예로는 LinkedIn의 Databus 프로젝트가 있습니다. Databus는 Oracle 트랜잭션 로그를 마이닝하여 변경 사항에 따라 이벤트를 발행합니다. LinkedIn은 Databus를 사용하여 시스템 내의 다양한 기록 간 일관성을 보장합니다.

또 다른 예로는 AWS의 DynamoDB 스트림 메커니즘이 있습니다. DynamoDB는 관리되는 NoSQL 데이터베이스이며, DynamoDB 스트림은 데이터베이스 테이블에 대한 최근 24시간 동안의 시간순 변경 사항(생성, 업데이트, 삭제 작업)을 나타냅니다. 애플리케이션은 이러한 변경 사항을 스트림에서 읽고, 이를 이벤트로 발행할 수 있습니다.

트랜잭션 로그 마이닝은 장단점이 있습니다. 장점은 이벤트를 발행하는 모든 업데이트가 2PC에 의존하지 않고 보장된다는 것입니다. 트랜잭션 로그 마이닝은 이벤트 발행과 애플리케이션 비즈니스 로직을 분리함으로써 단순화될 수 있습니다. 주요 단점은 트랜잭션 로그가 데이터베이스마다 다른 형식을 가지고 있으며, 심지어 같은 데이터베이스의 다른 버전에서도 형식이 다를 수 있다는 것입니다. 또한, 저수준 트랜잭션 로그 업데이트에서 고수준 비즈니스 이벤트로 변환하는 것이 어려울 수 있습니다.

트랜잭션 로그 마이닝 방법은 애플리케이션이 2PC 없이 직접 데이터베이스를 업데이트함으로써 이루어집니다. 이제 완전히 다른 접근 방식을 살펴보겠습니다. 업데이트 없이 이벤트에만 의존하는 방법입니다.

## 1.3.3 이벤트 소싱 사용

이벤트 소싱(Event Sourcing)은 2PC 없이 원자성을 달성하는 근본적으로 다른, 이벤트 중심의 접근 방식을 사용하여 비즈니스 엔티티의 일관성을 보장합니다. 이 접근 방식에서 애플리케이션은 비즈니스 엔티티의 현재 상태를 저장하는 대신, 엔티티의 상태 변경 이벤트 시리즈를 저장합니다. 애플리케이션은 이벤트를 재생하여 엔티티의 현재 상태를 재구성할 수 있습니다. 비즈니스 엔티티가 변경될 때마다 새 이벤트가 타임라인에 추가됩니다. 이벤트 저장은 단일 작업이므로 반드시 원자적입니다.

이벤트 소싱 작업 방식을 이해하기 위해 주문 엔티티를 예로 들어보겠습니다. 전통적인 방식에서는 각 주문이 ORDER 테이블의 한 행으로 매핑되고, 예를 들어 ORDER_LINE_ITEM 테이블에서 주문 항목이 매핑됩니다. 그러나 이벤트 소싱 방식에서는 주문 서비스가 주문의 상태 변경을 이벤트로 저장합니다: 생성됨, 승인됨, 발송됨, 취소됨; 각 이벤트는 주문 상태를 재구성하기에 충분한 데이터를 포함합니다.

![Event-sourcing](./images/Event-sourcing.png)

이벤트는 이벤트 데이터베이스에 장기간 저장되며, 엔티티 이벤트를 추가하고 가져오는 API를 제공합니다. 이벤트 저장소는 앞서 설명한 메시지 브로커와 유사하게 작동하며, 이벤트를 구독하는 API를 제공합니다. 이벤트 저장소는 모든 관심 있는 구독자에게 이벤트를 전달합니다. 이벤트 저장소는 이벤트 기반 마이크로서비스 아키텍처의 핵심입니다.

이벤트 소싱 방법은 많은 장점을 가지고 있습니다. 이는 이벤트 기반 아키텍처의 핵심 문제를 해결하며, 상태 변경이 있을 때마다 이벤트를 신뢰할 수 있게 발행할 수 있으며, 이는 마이크로서비스 아키텍처에서 데이터 일관성 문제를 해결합니다. 또한, 객체를 저장하는 대신 이벤트를 지속적으로 저장함으로써 객체 관계 임피던스 불일치 문제를 피할 수 있습니다.

이벤트 소싱 방법은 비즈니스 엔티티 변경의 100% 신뢰할 수 있는 모니터링 로그를 제공하며, 어떤 시점에서든 엔티티 상태를 얻을 수 있게 합니다. 또한, 이벤트 소싱 방법은 비즈니스 로직을 이벤트 교환에 의해 느슨하게 결합된 비즈니스 엔티티로 구성할 수 있게 해줍니다. 이러한 이점은 단일체 애플리케이션을 마이크로서비스 아키텍처로 마이그레이션하는 것을 상대적으로 쉽게 만듭니다.

이벤트 소싱 방법은 단점도 가지고 있습니다. 새로운 또는 익숙하지 않은 프로그래밍 패턴을 사용하기 때문에, 학습 곡선이 가파를 수 있습니다. 이벤트 저장소는 주요 키를 기반으로 엔티티를 조회하는 것만 지원하므로, Command Query Responsibility Segregation (CQRS)를 사용하여 조회 작업을 수행해야 합니다. 따라서, 애플리케이션은 최종적으로 일관된 데이터를 처리해야 합니다.

# 1.4 결론

마이크로서비스 아키텍처에서, 각 마이크로서비스는 자체 사적인 데이터 세트를 가집니다. 다양한 마이크로서비스는 다른 SQL 또는 NoSQL 데이터베이스를 사용할 수 있습니다. 데이터베이스 아키텍처는 강력한 이점을 제공하지만, 분산 데이터 관리의 도전과제도 있습니다. 첫 번째 도전과제는 여러 서비스 간에 비즈니스 트랜잭션의 일관성을 유지하는 방법이며, 두 번째 도전과제는 여러 서비스 환경에서 일관된 데이터를 얻는 방법입니다.

최적의 해결책은 이벤트 기반 아키텍처를 사용하는 것입니다. 여기에는 상태를 원자적으로 업데이트하고 이벤트를 발행하는 방법에 대한 도전과제가 포함됩니다. 이 문제를 해결하기 위한 몇 가지 방법이 있으며, 이에는 데이터베이스를 메시지 큐로 간주하고, 트랜잭션 로그 마이닝, 이벤트 소싱 등이 포함됩니다.
