# @Configuration 

## 개념 
`@Configuration`은 설정 클래스를 의미한다 
<br> 쉽게 말하면, Spring에게 다음과 말하는 것이다.
> "이 클래스 안에는 Spring이 관리해야 할 객체들을 등록하는 설정 코드가 들어 있다."

예를 들어 코드(내가 개인적으로 Kamis Project때 사용했던) 
`KamisPriceJobConfig`는 다음과 같은 역할을 한다. 
``` java
@Configuration
@RequiredArgsConstructor
public class KamisPriceJobConfig {

    private final JobRepository jobRepository;

    private final PlatformTransactionManager transactionManager;

    private final KamisItemReader reader;

    private final KamisItemProcessor processor;

    private final JdbcBatchItemWriter<PriceData> writer;

    @Bean
    public Job kamisPriceJob() {

        return new JobBuilder("kamisPriceJob", jobRepository)
                .start(kamisPriceStep())
                .build();
    }

    @Bean
    public Step kamisPriceStep() {

        return new StepBuilder("kamisPriceStep", jobRepository)
                .<ExpandedPriceRow, PriceData>chunk(100, transactionManager)

                .reader(reader)

                .processor(processor)

                .writer(writer)

                .build();
    }
}
 ```

- Job Bean 등록 
- Step Bean 등록 
- Reader / Processor / Writer를 묶어서 배치 흐름 정의

즉, `@Configuration` 클래스는 애플리케이션의 조립 설명서 같은 역할을 한다.

## 왜 필요한가 
배치에서는 Reader, Processor, Writer를 각각 따로 만들고 끝나는 것이 아니라, 
이 셋을 하나의 Step, 그리고 Step을 하나의 Job으로 연결해야 한다. 

그 연결을 담당하는 곳이 보통 `@Configuration` 클래스다. 


예를 들면
``` java
@Configuration
public class KamisPriceJobConfig {

    @Bean
    public Job kamisPriceJob() { ... }

    @Bean
    public Step kamisPriceStep() { ... }
}
``` 
이렇게 작성하면 Spring은 이 클래스를 읽고 
- `kamisPriceJob이라는 Bean`
- `kamisPriceStep이라는 Bean`

을 등록한다.

## 정리 
`@Configuration`은 다음 상황에서 상요한다. 
 - Bean을 조립해야 할 때 
 - 프레임워크 설정을 정의할 때 
 - 배치 Job/Step 같은 구조를 구성할 때 

즉, `@Configuration`은 **로직 클래스라기보다 설정/조립 클래스**에 붙는다고 이해하면 편하다. 
