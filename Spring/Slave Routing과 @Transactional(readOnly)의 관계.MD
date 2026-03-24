# Spring Master/Slave Routing과 @Transactional(readOnly)의 관계

> 지난 회사에서 진행했던 프로젝트를 회고 및 리팩토링하면서 남기는 기록 

Spring boot 에서 Master/Slave 구조를 사용했고 
``` java
쓰기 트랜잭션 → Master DB
읽기 트랜잭션 → Slave DB
```
이를 구현하기 위해  `AbstractRoutingDataSource`를 사용하며 
내가 궁금햇던 점은 
``` java
@Transactional(readOnly = true)가 어떻게 Slave DB 선택에 영향을 주는가?
```
였다 

### 결론 부터 말하면 
``` java
readOnly는 DB를 직접 바꾸는 것이 아니라, 
Spring이 현재 트랜잭션 상태를 저장하고, 
RountingDataSource가 그 상태를 읽어 DataSource를 선택한다. 
```

도움 예시 
``` java
public class RoutingDataSource extends AbstractRoutingDataSource {

    /**
     * 현재 트랜잭션 기반으로 사용할 DataSource Key를 결정
     */
    @Override
    protected Object determineCurrentLookupKey() {

        boolean isReadOnly = TransactionSynchronizationManager.isCurrentTransactionReadOnly();

        if (isReadOnly) {
            log.info("[DataSource Routing] -> SLAVE");
            return DataSourceType.SLAVE;
        }
        log.info("[DataSource Routing] → MASTER");
        return DataSourceType.MASTER;
    }
}
```

### 전체 동작 흐름 
읽기 API 호출 시 내부 흐름을 보면 

``` java
1. Controller 
2. Service 호출 
3. @Transactional AOP 프록시 개입 
4. 트랜잭션 시작 
5. readOnly=true 상태를 ThreadLocal에 저장 
6. Repository 실행 (DB 커넥션 필요)
7. RoutingDataSource 호출 
8. 현재 트랜잭션 상태 확인 
9. SLAVE 선택 
10. 쿼리 실행 
``` 

### @Transactional의 본질 (AOP 관점)
`@Transactional`은 단순 애노테이션이 아니라 
Spring의 AOP기반 프록시 매커니즘으로 동작한다. 

``` java
메서드 호출 -> 프록시가 가로챔 -> 트랜잭션 시작 -> 실제 메서드 진행 
``` 

즉, 실제 흐름을 보면 
``` java
프록시 
 -> 트랜잭션 시작 
 -> readOnly 설정 
 -> 실제 비즈니스 로직 실행 
 -> 트랜잭션 종료 
``` 

따라서 `@Transactional`은 '선언' 이고 실제 동작은 프록시가 수행한다. 

### 트랜잭션 상태 저장 방식 
Spring은 트랜잭션 상태를 ThreadLocal 기반으로 관리한다. 
``` java
TransactionSynchronizationManager.isCurrentTransactionReadOnly()
``` 

여기서 
``` java
@Transactional(readOnly = true)
 → 트랜잭션 시작 시 상태 저장
 → 이후 모든 로직에서 동일 쓰레드에서 참조 가능
``` 

### RoutingDataSource의 역할
``` java
boolean isReadOnly =
    TransactionSynchronizationManager.isCurrentTransactionReadOnly();
``` 
``` java
true  → SLAVE
false → MASTER
``` 

여기서 중요한점은 
**RoutingDataSource는 @Transactional을 직접 보는 것이 아니라 
"현재 트랜잭션 상태" 를 보고 판단**하는 것이다. 

### 커넥션 획득 시점 
Routing이 제대로 동작하려면 순서가 중요한데 아래 같은 순서로 진행된다. 
``` java
1. 트랜잭션 시작 
2. readOnly 상태 설정 
3. 커넥션 획득 
4. Routing 결정 
``` 

**잘못된 경우**
``` java
1. 커넥션 먼저 획득
2. 이후 트랜잭션 시작
``` 
이 경우 routing이 깨진다.

### LazyConnectionDataSourceProxy의 존재 이유 
이 문제를 해결하기 위해 사용한다. 
``` java
new LazyConnectionDataSourceProxy(routingDataSource)
``` 

역할
``` java
커넥션 획득을 "실제 쿼리 실행 시점까지 지연"
``` 
효과 
``` java
트랜잭션 -> 상태 설정 -> 이후 커넥션 획득 -> Routing 정확하게 수행 가능 
``` 

### readOnly의 진짜의미 
``` java
readOnly = true → DB 쓰기 막는다 ❌
``` 

실제 의미 : 
``` java
1. 트랜잭션을 "읽기 전용"으로 표시 
2. Routing 판단 기준으로 사용 
3. 일부 JPA 최적화 힌트 제공 
``` 

### 간단하게 정리하면 
``` java
1. @Transactional은 AOP 기반으로 동작한다 
2. 트랜잭션 상태는 ThreadLocal에 저장된다 
3. readOnly는 상태값일 뿐 직접 DB를 바꾸지 않는다. 
4. RoutingDataSource는 커넥션 획득 시점에 상태를 읽는다. 
5. LazyConnectionDataSourceProxy가 있어야 타이밍이 맞는다
``` 
