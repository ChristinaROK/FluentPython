# Part3: Function as object (객체로서의 함수)
> 현재 Fluent Python 을 공부하고, 파트별로 핵심 내용을 정리하고 있다.   
> 오늘은 세번째 파트인 (first-class function)일급 객체로서의 함수에 대해 알아본다. 

## Chapter6: Design pattern of first-class function (일급 함수 디자인 패턴)

> 함수 객체를 이용한 리펙토링으로 패턴을 단순화

### 전략 패턴 (Strategy Pattern)

* 정의: 공유 객체(전략 객체)를 생성해 여러 콘텍스트에 동시에 사용할 수 있도록 함 
* 장점
    * 객체를 공유하기때문에 반복 작업에 소요되는 비용을 줄일 수 있음
    * 알고리즘을 사용하는 클라이언트와 알고리즘을 독립적으로 변경 가능
    
* UML(Unified Modeling Language) class diagram
    * 다이어그램 기본 설명 : [참고 url](https://morm.tistory.com/88)
    * 콘텍스트 (context) : 콘텍스트의 클라이언트에 따라 하나의 전략이 선택됨
    * 전략 (strategy) : 여러 알고리즘이 구현하는 컴포넌트의 공통 인터페이스 (추상 클래스)
    * 구체적인 전략 : 전략의 서브 클래스 중 하나
    
![img](./chap6-uml.png)


```python
from abc import ABC, abstractmethod
from collections import namedtuple

Customer = namedtuple('Customer', 'name fidelity') # client

class LineItem:
    
    def __init__(self, product, quantity, price):
        self.product = product
        self.quantity = quantity
        self.price = price
    
    
    def total(self):
        return self.price * self.quantity

    
class Order: # Context
    
    def __init__(self, customer, cart, promotion=None):
        self.customer = customer
        self.cart = list(cart)
        self.promotion = promotion
    
    
    def total(self):
        if not hasattr(self, '_total'):
            self._total = sum(item.total() for item in self.cart)
        return self._total
    
    
    def due(self):
        if self.promotion is None:
            discount = 0
        else:
            discount = self.promotion.discount(self) # Promotion의 개별 자식 클래스의 메소드(인수로 self를 갖는)를 호출
        return self.total() - discount
    
    
    def __repr__(self):
        return f'<Order total : {self.total():,.0f} due : {self.due():,.0f}>'

    
class Promotion(ABC): # Strategy : Abstract base class 
    
    @abstractmethod
    def discount(self, order):
        """
        할인액을 구체적이 숫자로 반환한다.
        """

        
class FidelityPromo(Promotion): # Specific Strategy 1
    """충성도 포인트가 1000점 이상인 고객에게 전체 5% 할인 적용"""
    
    
    def discount(self, order):
        return order.total() * .05 if order.customer.fidelity >= 1000 else 0

    
class BulkItemPromo(Promotion): # Specific Strategy 2
    """20개 이상의 동일 상품을 구입하면 해당 상품에 대해 10% 할인 적용"""
    
    
    def discount(self, order):
        discount = 0
        for item in order.cart:
            if item.quantity>= 20:
                discount += item.total() * .1
        return discount

    
class LargeOrderPromo(Promotion): # Specific Strategy 3
    """서로 다른 상품을 10종류 이상 주문하면 전체 주문에 대해 7% 할인 적용"""
    
    
    def discount(self, order):
        unique_item = {item.product for item in order.cart}
        return order.total() * .07 if len(unique_item)>=10 else 0
```


```python
# client
chris = Customer('Christina', 0)
wang = Customer('Daewang', 1100)
cart = [LineItem('banana', 4, 500),LineItem('wine', 2, 15000), LineItem('puppygum', 12, 800)]

print(Order(wang, cart, FidelityPromo()))
print(Order(chris, cart, FidelityPromo()))

```

    <Order total : 41,600 due : 39,520>
    <Order total : 41,600 due : 41,600>


* 함수 지향 전략 
    * 추상 클래스로 구현됐던 Strategy 클래스를 제거
    * Specific Strategy를 개별 함수로 구현


```python
from collections import namedtuple

Customer = namedtuple('Customer', 'name fidelity') # client

class LineItem:
    
    def __init__(self, product, quantity, price):
        self.product = product
        self.quantity = quantity
        self.price = price
    
    
    def total(self):
        return self.price * self.quantity


class Order: # context
    
    def __init__(self, customer, cart, promotion=None):
        self.customer = customer
        self.cart = list(cart) ## 
        self.promotion = promotion
    
    def total(self):
        if not hasattr(self, '_total'):
            self._total = sum(item.total() for item in self.cart)
        return self._total
    
    
    def due(self):
        if self.promotion is None:
            discount = 0
        else:
            discount = self.promotion(self) # 개별 함수를 호출(인수로 self를 갖는)
        return self.total() - discount
    
    
    def __repr__(self):
        return f'<Order total : {self.total():,.0f} due : {self.due():,.0f}>'

    
def fidelity_promo(order): # Specific Strategy 1
    """충성도 포인트가 1000점 이상인 고객에게 전체 5% 할인 적용"""
    return order.total() *.05 if order.customer.fidelity >= 1000 else 0

def bulk_item_promo(order): # Specific Strategy 2
    """20개 이상의 동일 상품을 구입하면 해당 상품에 대해 10% 할인 적용"""
    discount = 0
    for item in order.cart:
        if item.quantity >= 20:
            discount += item.total() * .1
    return discount

def large_order_promo(order): # Specific Strategy 3
    """서로 다른 상품을 10종류 이상 주문하면 전체 주문에 대해 7% 할인 적용"""
    unique_item = set(item.product for item in order.cart)
    if len(unique_item) >= 10:
        return order.total() * .07
    return 0
```


```python
# client
chris = Customer('Christina', 0)
wang = Customer('Daewang', 1100)
cart = [LineItem('banana', 4, 500),LineItem('wine', 2, 15000), LineItem('puppygum', 12, 800)]

print(Order(wang, cart, fidelity_promo))
print(Order(chris, cart, fidelity_promo))

puppy_cart = [LineItem('puppygum', 30, 800), LineItem('banana', 25, 500)]
print(Order(wang, puppy_cart, bulk_item_promo))

long_order = [LineItem(str(item_code), 1, 2000) for item_code in range(10)]
print(Order(chris, long_order, large_order_promo))
```

    <Order total : 41,600 due : 39,520>
    <Order total : 41,600 due : 41,600>
    <Order total : 36,500 due : 32,850>
    <Order total : 20,000 due : 18,600>


* 최선의 전략으로 리펙토링하기


```python
promos = [fidelity_promo, bulk_item_promo, large_order_promo] # (1)
def best_promo(order):
    """최대로 할인받을 금액을 반환"""
    return max([promo(order) for promo in promos])
```


```python
print(Order(wang, puppy_cart, best_promo))
print(Order(chris, long_order, best_promo))
```

    <Order total : 36,500 due : 32,850>
    <Order total : 20,000 due : 18,600>


* (1) list of functions : 새로운 promotion함수가 정의될때마다 수동으로 함수를 추가해야한다는 문제가 발생

* 해결1 (하나의 모듈에 모든 코드를 모아 놓음) : `globals()` 내장 함수
    * __현재 모듈__에 대한 내용 테이블을 나타내는 딕셔너리 객체 (__현재 모듈__이란, globals()가 호출된 위치를 기준으로 한다. 예를 들어, 메서드 안에서 호출되면 메서드를 호출한 모듈이 아니라 메서드가 __정의된__ 모듈을 의미한다.)  


```python
promos = [globals()[name] for name in globals() 
                if name.endswith('_promo') and name != 'best_promo']
def best_promo(order):
    """최대로 할인받을 금액을 반환"""
    return max([promo(order) for promo in promos])
```

* 해결2 (모듈을 분리): `inspect` 모듈의 `getmembers` 매소드
    * 함수만을 모아 놓은 모듈을 임포트해와 함수인 객체만 사용하는 방법


```python
import inspect
import promos # *_promo 함수만 모아 놓은 모듈
    
promos = [func for name, func in inspect.getmembers(promotions, inspect.isfunction)]
def best_promo(order):
    """최대로 할인받을 금액을 반환"""
    return max([promo(order) for promo in promos])
```

### 명령 패턴 (Command Pattern)

* 정의: 연산을 실행하는 객체(=호출자(invoker))와 연산을 구현하는 객체(=수신자(receiver))를 분리하여 구현 
* 장점
     * 호출자는 수신자의 인터페이스를 알 필요가 없음
     * 호출자는 서브클래스를 통해 새로운 수신자를 추가할 수 있음
* UML(Unified Modeling Language) class diagram
