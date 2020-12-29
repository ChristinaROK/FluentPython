# Part3: Function as object (객체로서의 함수)
> 현재 Fluent Python 을 공부하고, 파트별로 핵심 내용을 정리하고 있다.   
> 오늘은 세번째 파트인 (first-class function)일급 객체로서의 함수에 대해 알아본다. 

## Chapter7: Function Decorator and Closure (함수 데커레이터와 클로져)


### decorator (데커레이터)
* 정의 : 다른 함수를 인수로 받는 콜러블(=decorated function)
* 역할 : 데커레이트된 함수(=decorated function)에 특정한 처리를 수행하고 함수를 반환하거나 함수를 다른 함수나 콜러블 객체로 대체함
* 예시 : decorate이란 이름의 데커레이터가 있다고 가정할 때, method1과 method2는 동일한 함수

```python
# method1
@decorate
def target():
    print(f"running target()!")

#method2
decorate(target)
```
* 실행 시간
    * 데커레이터(ex. decorate())가 실행되는 시간 : **임포트타임**
    * 데커레이트된 함수(decorated function ex. target())가 실행되는 시간 : **런타임**에서 명시적으로 함수가 호출될 때 실행


```python
def deco(func):
    def inner():
        print('running inner()')
    return inner # (1)

@deco
def target():
    print('running target()')
```

* (1) deco 함수는 inner() 함수의 객체를 반환함. 따라서 데커레이트 된 target() 함수를 호출하면 inner() 함수를 호출함   
  즉, 데커레이터는 함수를 다른 함수로 대체할 수 있음


```python
print(target()) 

target # target 함수는 inner()을 가리키고 있음
```

    running inner()
    None





    <function __main__.deco.<locals>.inner()>



### Improve strategy pattern with decorater (전략 패턴을 데커레이터로 개선하기)


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
```


```python
promos = []

def promotion(promo_func):
    promos.append(promo_func)
    return promo_func # (1)

@promotion # (2)
def fidelity_promo(order): # Specific Strategy 1
    """충성도 포인트가 1000점 이상인 고객에게 전체 5% 할인 적용"""
    return order.total() *.05 if order.customer.fidelity >= 1000 else 0

@promotion
def bulk_item_promo(order): # Specific Strategy 2
    """20개 이상의 동일 상품을 구입하면 해당 상품에 대해 10% 할인 적용"""
    discount = 0
    for item in order.cart:
        if item.quantity >= 20:
            discount += item.total() * .1
    return discount

@promotion
def large_order_promo(order): # Specific Strategy 3
    """서로 다른 상품을 10종류 이상 주문하면 전체 주문에 대해 7% 할인 적용"""
    unique_item = set(item.product for item in order.cart)
    if len(unique_item) >= 10:
        return order.total() * .07
    return 0

def best_promo(order):
    """최대로 할인받을 금액을 반환"""
    return max(promo(order) for promo in promos)
```

* (1) 등록 데커레이터 : 인수로 들어온 콜백 함수를 다시 반환 함
* (2) promotion 데커레이터를 이용하면, *_promo 함수가 새로 생성될때마다 `promos` 리스트에 수동으로 등록해야됐던 한계를 극복   
  임포트타임에 promotion 데커레이터를 갖는 모든 함수가 `promos` 리스트에 입력됨 
* 이전 방법들 (ex.`globals()`사용 , `inspector.getmembers()`사용 / `Part3-Chap6` 참고)과 비교했을 때 장점:
    * 함수 이름이 자유로움. 굳이 *_promo로 끝나지 않아도 됨
    * 함수가 다른 모듈에서 정의되도 됨. @promotion 데커레이터만 붙이면 됨

### 변수 범위 규칙

* 함수 외부
    * 모든 변수는 전역 변수 취급
    * 단, 함수 내부에 같은 변수명이 사용될 시, 함수 내부에서는 지역 변수로 취급


```python
# b <- global variable
def f1(a):
    print(a)
    print(b)
    
b = 6
f1(3)  
```

    3
    6



```python
# b <- local variable
def f1(a):
    print(a)
    print(b)
    b = 9 # 파이썬 인터프리터가 b를 지역 변수로 취급
    
b = 6
f1(3) 
```

    3



    ---------------------------------------------------------------------------

    UnboundLocalError                         Traceback (most recent call last)

    <ipython-input-16-dceb2dd95705> in <module>
          5 
          6 b = 6
    ----> 7 f1(3)
    

    <ipython-input-16-dceb2dd95705> in f1(a)
          1 def f1(a):
          2     print(a)
    ----> 3     print(b)
          4     b = 9
          5 


    UnboundLocalError: local variable 'b' referenced before assignment


* 함수 내부
    * 파이썬은 함수 내의 변수는 지역 변수로 여김. 전역 변수로 사용하기위해서는 전역 변수로 선언(`global`)해야 함


```python
b = 6
def f(a):
    global b
    print(a)
    print(b)
    b = 9
f(3)
f(3)
```

    3
    6
    3
    9


### closure (클로저)
* 정의 : 어떤 함수가 다른 함수 내부에 선언됐을 때, 외부 함수의 변수는 **자유 변수**, 즉 전역 변수처럼 다룰 수 있게 됨  
  (이를 제외하고는 함수 내부의 변수는 모두 지역 변수 취급)
* 용도 : 데커레이터 함수에서 변수를 클로저에 저장해 데커레이터의 내부 함수에서도 그 변수를 사용할 수 있도록 만듦 


```python
class Averager: # (1)
    
    def __init__(self):
        self.series = []
    
    
    def __call__(self, new_value):
        self.series.append(new_value)
        total = sum(self.series)
        return total / len(self.series)

    
def make_averager(): # (2)
    series = [] # (3)
    ##### closure #####
    def averager(new_value):
        series.append(new_value)
        total = sum(series)
        return total / len(series)
    
    return averager
```

* (1) 이동 평균을 계산하는 __클래스__ 
* (2) 이동 평균(일정하게 변하는 값의 평균)을 계산하는 __고위 함수__
* (3) 자유 변수 (free variable) : 지역 범위에 바인딩 되어 있지 않은 변수. 
    * 함수 내부의 변수(series)를 전역 변수처럼 사용할 수 있는 것이 특징.


```python
# 같은 결과 값을 냄
avg = Averager()
print(avg(10))
print(avg(11))
print(avg(12))
print()

avg = make_averager()
print(avg(10))
print(avg(11))
print(avg(12))
```

    10.0
    10.5
    11.0
    
    10.0
    10.5
    11.0



```python
print(avg.__code__.co_varnames) # 지역 변수
print(avg.__code__.co_freevars) # 자유 변수
```

    ('new_value', 'total')
    ('series',)


* 자유 변수의 값을 `__closure__`속성에서 찾기


```python
avg.__closure__[0].cell_contents
```




    [10, 11, 12]



### nonlocal 
* 정의 : nonlocal로 선언된 변수는 함수 안에서 값을 재할당 받더라도 자유 변수의 특징을 유지함
* 용도 : 불변형의 변수가 함수 내에서 재할당될 때 자유 변수의 속성을 유지하기위해 이 변수 앞에 nonlocal을 붙임



```python
def make_average():
    count = 0
    total = 0
    
    def averager(new_value):
        count+=1 # (1)
        total+=new_value # (1)
        return total / count
    return averager
```


```python
avg = make_average()
avg(1)
```


    ---------------------------------------------------------------------------

    UnboundLocalError                         Traceback (most recent call last)

    <ipython-input-53-be3152f359be> in <module>
          1 avg = make_average()
    ----> 2 avg(1)
    

    <ipython-input-52-447248c572ed> in averager(new_value)
          4 
          5     def averager(new_value):
    ----> 6         count+=1 # (1)
          7         total+=new_value # (1)
          8         return total / count


    UnboundLocalError: local variable 'count' referenced before assignment


* make_average 함수가 new_value의 값을 저장하는 것은 비효율적이기 때문에 "누적합"과 "개수"를 저장하도록 수정
* (1) 잘못된 함수 : count변수는 불변형인데, 값을 **재할당**하는 연산 때문에 이 변수는 지역 변수 취급 됨. 즉 count 변수가 클로저에 저장되지 않음


```python
def make_average():
    count = 0
    total = 0
    
    def averager(new_value):
        nonlocal count, total # (1)
        count+=1 
        total+=new_value 
        return total / count
    return averager
```


```python
avg = make_average()
print(avg(1))
print(avg(2))
```

    1.0
    1.5


* (1) nonlocal을 이용해 자유 변수 특징을 유지하기

### 데커레이터 구현 (클로저를 이용)
* 전형적인 데커레이터 작동 방식
    * 데커레이트된 함수(ex. snooze, factorial)를 동일한 인수를 받는 다른 함수(clocked)로 교체
    * 추가적인 처리를 수행 (ex. 연산 소요 시간 측정)
    * 데커레이트된 함수가 반환해야하는 값을 반환 (ex. `return result`)


```python
import time

def clock(func):
    def clocked(*args):
        t0 = time.perf_counter()
        result = func(*args) # (1)
        elapsed = time.perf_counter() - t0
        name = func.__name__
        arg_str = ",".join(repr(arg) for arg in args)
        print(f"[{elapsed:0.8f}] {name}({arg_str}) -> {result}")
        return result
    return clocked #(2)
```

* (1) `func`는 자유 변수
* (2) 데커레이트된 함수 (아래 함수)의 반환값을 내부 함수로 대체함


```python
@clock
def snooze(seconds):
    time.sleep(seconds)

@clock
def factorial(n):
    return 1 if n<2 else n * factorial(n-1)

snooze(.123)
print("*"*40)
factorial(6) # (1)
```

    [0.12320990] snooze(0.123) -> None
    ****************************************
    [0.00000113] factorial(1) -> 1
    [0.00006259] factorial(2) -> 2
    [0.00010673] factorial(3) -> 6
    [0.00015059] factorial(4) -> 24
    [0.00019386] factorial(5) -> 120
    [0.00023738] factorial(6) -> 720





    720



* (1) factorial 함수가   

    ```python
    def factorial(n):
        return 1 if n<2 else n*f(n-1)
    
    ```
    
    일 때, clock 데커레이트된 factorial함수는 factorial = clock(factorial) 로 표현 가능함.   
    clock 데커레이터 함수는 clocked 함수를 반환하기 때문에   
    factorial == clock(factorial) == clocked 임.      
    즉, **factorial == clocked**


```python
# (1)의 증명
print(snooze.__name__)
print(factorial.__name__)
```

    clocked
    clocked



```python
import time
import functools

def clock(func):
    @functools.wraps(func) # (1)
    def clocked(*args, **kwargs): # (2)
        t0 = time.perf_counter()
        result = func(*args, **kwargs) 
        elapsed = time.perf_counter() - t0
        name = func.__name__
        arg_list = []
        if args:
            arg_list.append(",".join(repr(arg) for arg in args))
        if kwargs:
            pairs = [f"{k}={w}" for k,w in sorted(kwargs.items())]
            arg_list.append(",".join(pairs))
        arg_str=",".join(arg_list)
        print(f"[{elapsed:0.8f}] {name}({arg_str}) -> {result}")
        return result
    return clocked 
```

* 함수 리펙토링
    * (1) built-in 라이브러리 데코레이터를 사용해 데커레이트된 함수 (ex. snooze)의 `__name__`, `__doc__` 속성을 clocked에 복사
    * (2) keyword argument도 처리 가능


```python
# (1)
@clock
def snooze(seconds, multiply=1):
    time.sleep(seconds * multiply)

snooze(.123, multiply = 5)
```

    [0.61569073] snooze(0.123,multiply=5) -> None



```python
# (2)
print(snooze.__name__)
```

    snooze


### 표준 라이브버리에서 제공하는 데커레이터
* 종류 
    * classmethod()
    * staticmethod()
    * property()
    * functools.wrap()
    * functools.lru_cache()
    * functools.singledispatch()

#### functools.lru_cache()
* 명칭 : Least Recently Used. 즉, 오랫동안 사용하지 않은 항목을 버림
* 용도 : memoization(메모이제이션)을 구현. 즉, **이전에 실행한 값비싼 함수의 결과를 저장**해 이전에 사용된 인수에 대해 다시 계산하지 않게 하여 캐시를 효율적으로 사용. 웹에서 정보를 가져올 때도 유용


```python
@clock
def fibonacci(n):
    if n<2:
        return n
    return fibonacci(n-2) + fibonacci(n-1)

print(fibonacci(8))
```

    [0.00000117] fibonacci(0) -> 0
    [0.00000128] fibonacci(1) -> 1
    [0.00024098] fibonacci(2) -> 1
    [0.00000098] fibonacci(1) -> 1
    [0.00000098] fibonacci(0) -> 0
    [0.00000089] fibonacci(1) -> 1
    [0.00008757] fibonacci(2) -> 1
    [0.00017466] fibonacci(3) -> 2
    [0.00050404] fibonacci(4) -> 3
    [0.00000088] fibonacci(1) -> 1
    [0.00000088] fibonacci(0) -> 0
    [0.00000084] fibonacci(1) -> 1
    [0.00009000] fibonacci(2) -> 1
    [0.00017411] fibonacci(3) -> 2
    [0.00000075] fibonacci(0) -> 0
    [0.00000082] fibonacci(1) -> 1
    [0.00008516] fibonacci(2) -> 1
    [0.00000084] fibonacci(1) -> 1
    [0.00000098] fibonacci(0) -> 0
    [0.00000082] fibonacci(1) -> 1
    [0.00008604] fibonacci(2) -> 1
    [0.00016938] fibonacci(3) -> 2
    [0.00033643] fibonacci(4) -> 3
    [0.00059457] fibonacci(5) -> 5
    [0.00122175] fibonacci(6) -> 8
    [0.00000077] fibonacci(1) -> 1
    [0.00000082] fibonacci(0) -> 0
    [0.00000081] fibonacci(1) -> 1
    [0.00008427] fibonacci(2) -> 1
    [0.00016892] fibonacci(3) -> 2
    [0.00000076] fibonacci(0) -> 0
    [0.00000082] fibonacci(1) -> 1
    [0.00008376] fibonacci(2) -> 1
    [0.00000079] fibonacci(1) -> 1
    [0.00000083] fibonacci(0) -> 0
    [0.00000080] fibonacci(1) -> 1
    [0.00008746] fibonacci(2) -> 1
    [0.00021359] fibonacci(3) -> 2
    [0.00038202] fibonacci(4) -> 3
    [0.00063648] fibonacci(5) -> 5
    [0.00000080] fibonacci(0) -> 0
    [0.00000082] fibonacci(1) -> 1
    [0.00008319] fibonacci(2) -> 1
    [0.00000083] fibonacci(1) -> 1
    [0.00000094] fibonacci(0) -> 0
    [0.00000087] fibonacci(1) -> 1
    [0.00008406] fibonacci(2) -> 1
    [0.00021413] fibonacci(3) -> 2
    [0.00038152] fibonacci(4) -> 3
    [0.00000072] fibonacci(1) -> 1
    [0.00000089] fibonacci(0) -> 0
    [0.00000095] fibonacci(1) -> 1
    [0.00008349] fibonacci(2) -> 1
    [0.00016719] fibonacci(3) -> 2
    [0.00000079] fibonacci(0) -> 0
    [0.00000083] fibonacci(1) -> 1
    [0.00008379] fibonacci(2) -> 1
    [0.00000075] fibonacci(1) -> 1
    [0.00000130] fibonacci(0) -> 0
    [0.00000085] fibonacci(1) -> 1
    [0.00008747] fibonacci(2) -> 1
    [0.00017136] fibonacci(3) -> 2
    [0.00033728] fibonacci(4) -> 3
    [0.00058641] fibonacci(5) -> 5
    [0.00104886] fibonacci(6) -> 8
    [0.00176642] fibonacci(7) -> 13
    [0.00307235] fibonacci(8) -> 21
    21



```python
import functools

@functools.lru_cache() # (1)
@clock
def fibonacci(n):
    if n<2:
        return n
    return fibonacci(n-2) + fibonacci(n-1)

print(fibonacci(8))
```

    [0.00000124] fibonacci(0) -> 0
    [0.00000117] fibonacci(1) -> 1
    [0.00024840] fibonacci(2) -> 1
    [0.00000257] fibonacci(3) -> 2
    [0.00035040] fibonacci(4) -> 3
    [0.00000121] fibonacci(5) -> 5
    [0.00041351] fibonacci(6) -> 8
    [0.00000111] fibonacci(7) -> 13
    [0.00047443] fibonacci(8) -> 21
    21


* (1) 데커레이터에 인수 존재   
    ```functools.lru_cache(maxsize=128, type=False)```
    * maxsize: 얼마나 많은 호출을 할 것인가 (2의 제곱수)
    * type: 인수의 자료형이 다르면 결과도 다르게 저장할 것인가
* 주의 : functools.lru_cache() 데커레이트 된 함수의 인수는 반드시 **해시가능** 해야 함 

#### functools.singledispatch()
* 용도: 파이썬은 오버로딩을 지원하지 않아, 인수의 종류에 따라 서로 다른 시그니처를 가진 함수를 만들 수 없음      
    하지만 functools.singledispatch()로 데커레이트 된 범용 함수(generic function)는 첫 번째 인수의 자료형에 따라 서로 다른 알고리즘을 수행


```python
from functools import singledispatch
from collections import abc
import numbers
import html

@singledispatch
def htmlize(obj):
    content=html.escape(repr(obj))
    return f"<pre>{content}</pre>"

@htmlize.register(str) # (1)
def _(text): # (2)
    content=html.escape(text).replace('\n','<br>\n')
    return f"<p>{content}</p>"

@htmlize.register(numbers.Integral) # (3)
def _(n):
    return f"<pre>{n} ({hex(n)})</pre>"

@htmlize.register(tuple)
@htmlize.register(abc.MutableSequence) # (4)
def _(seq):
    inner = "</li>\n<li>".join(htmlize(item) for item in seq)
    return "<ul>\n<li>" + inner + "</li>\n</ul>"
```

* (1) : dispatch 함수에서 특화된 함수는 `@<dispatch함수명>.register(<객체형>)`으로 데커레이트 됨
* (2) : 특화된 함수는 고유의 이름을 가질 필요가 없기 때문에 언더바(_)로 함수명을 지정함
* (3) : numbers.Integral은 int의 가상(=추상) 슈퍼클래스(=부모클래스)
* (4) : 동일한 함수로 여러 자료형 (ex. tuple, mutable object(list, dictionary))을 지원하기위해 데커레이터를 여러개 쌓아 올림


```python
print(htmlize('hi \n there'))
print(htmlize(42))
print(htmlize(['alpha', 66, {3,2,1}]))
```

    <p>hi <br>
     there</p>
    <pre>42 (0x2a)</pre>
    <ul>
    <li><p>alpha</p></li>
    <li><pre>66 (0x42)</pre></li>
    <li><pre>{1, 2, 3}</pre></li>
    </ul>


### decorator factory (매개변수화된 데커레이터)
* 정의 : 데커레이터의 인수로 함수 말고 다른 값을 받도록 구현한 데커레이터


```python
registry = set()

def register(active = True): # (1)
    def decorate(func): # (2)
        print(f'running register(active={active}) -> decorate({func})')
        if active:
            registry.add(func)
        else:
            registry.discard(func)
            
        return func
    return decorate

@register(active=False) # (3)
def f1():
    print('running f1()')

@register()
def f2():
    print('running f2()')
    
    
print('running main()')
print(f'registry -> {registry}')
```

    running register(active=False) -> decorate(<function f1 at 0x7f8cfdb3bd90>)
    running register(active=True) -> decorate(<function f2 at 0x7f8cfd95f6a8>)
    running main()
    registry -> {<function f2 at 0x7f8cfd95f6a8>}


* (1): register()는 키워드 인수를 받는 데커레이터 팩토리.
* (2): decorate() 내부 함수가 실제 데커레이터. `f1 = register(active=True)(f1)` 이기 때문에 `f1 = decorate(f1)`이 된다. 따라서 **decorate()의 인수로 funciton이 들어가야한다는 점**에 유의할 것
* (3): @register 팩토리는 원하는 매개변수와 함께 **함수**로 호출해야 함


```python
import time

DEFULT_FMT = '[{elapsed:0.8f}s] {name}({args}) -> {result}'

def clock(fmt=DEFULT_FMT): 
    def decorate(func):
        def clocked(*_args):
            t0 = time.time()
            _result = func(*_args)
            elapsed = time.time() - t0
            name = func.__name__
            args = ', '.join(repr(arg) for arg in _args)
            result = repr(_result)
            print(fmt.format(**locals())) # (1)
            return _result
        return clocked
    return decorate


if __name__=="__main__":
    
    @clock()
    def snooze(seconds):
        time.sleep(seconds)
    
    @clock('{name}: {elapsed}s')
    def simple_snooze(seconds):
        time.sleep(seconds)
    
    for i in range(3):
        snooze(.123)
        
    for i in range(3):
        simple_snooze(.123)
```

    [0.12322688s] snooze(0.123) -> None
    [0.12321329s] snooze(0.123) -> None
    [0.12315106s] snooze(0.123) -> None
    simple_snooze: 0.12320089340209961s
    simple_snooze: 0.12319827079772949s
    simple_snooze: 0.1231529712677002s


* (1) : `locals()`는 이 함수가 호출된 메서드에서 정의된 모든 지역 변수를 딕셔너리 객체로 반환함. 따라서 format의 인수에 `**locals()`를 사용하면 지역 변수를 참조할 수 있음

##### etc : words definition
* signature(시그니처) : 함수나 메소드의 입력값(parameter)과 값의 자료형 또는 출력값과 값의 자료형
* abstract(추상화) : 공통적으로 사용하는 기능과 속성을 클래스나 함수로 묶어 이름을 붙임. 추상화된 클래스나 함수를 객체로 만들 수 없는데, 그 이유는 추상 클래스 또는 함수의 기능이 객체가 되기에는 너무 추상적으로 구현되어 있기 때문. 따라서 추상 클래스를 상속 받거나 추상 함수로 데커레이트 한 함수를 생성해 기능을 구체화한 후 객체를 생성해야 함

##### advanced contents 
* [how you implemented your python decorator is wrong](https://github.com/GrahamDumpleton/wrapt/blob/develop/blog/01-how-you-implemented-your-python-decorator-is-wrong.md)
