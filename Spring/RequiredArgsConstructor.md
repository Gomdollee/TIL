# @RequiredArgsConstructor 

> @RequiredArgsConstructor 어노테이션을 붙이는 이유 
> ->"의존성 주입(DI)을 안전하고, 불변하게, 코드 간결하게" 하기 위해서 

``` java
@RequiredArgsConstructor 
public class OrderService {
    private final OrderRepository orderRepository;
    private final PaymentClient paymentClient;
}
```
- final 이 붙은 필드만 파라미터로 받는 생성자를 자동 생성 

컴파일시 생기는 코드는 
 ``` java
 public OrderService(OrderRepository orderRepository,
                    PaymentClient paymentClient) {
    this.orderRepository = orderRepository;
    this.paymentClient = paymentClient;
}
 ```
