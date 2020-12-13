# Part3: Function as object (객체로서의 함수)
> 현재 Fluent Python 을 공부하고, 파트별로 핵심 내용을 정리하고 있다.   
> 오늘은 세번째 파트인 (first-class function)일급 객체로서의 함수에 대해 알아본다. 

## Chapter5: First-class function (일급 함수)

> 파이썬 함수는 일급 객체의 특징을 갖고 있다. 일급 객체의 특징으로는 변수에 할당 가능하며 함수의 인자로 전달이 가능하며 함수의 결과로 반환 가능하다는 점이 있다. 이번 장에서는 함수의 콜러블(callable) 특징, 함수의 속성(atttribute), 함수의 매개 변수 (parameters), 함수 애너테이션 (annotation), 그리고 내장 함수를 이용해 함수형 프로그래밍을 하는 방법을 알아본다. 


### function is first-class object(일급 객체)
* 정의 
    1. runtime에 생성 가능 
    2. 변수나 요소에 할당 가능
    3. 함수의 parameter (인자)로 전달 가능
    4. 함수의 결과로 반환 가능
* 종류 : int, str, dictionary, **function**, ...


```python
def factorial(n): # (1) runtime에 생성 가능
    """
    factorial function
    """
    return 1 if n<2 else n*factorial(n-1)
```


```python
# 1. 함수는 function이라는 class의 객체(instance)이다.
print(type(factorial))
```

    <class 'function'>



```python
# 2. 함수도 attribute(속성) (__doc__)을 갖고 있다. 
print(factorial.__doc__)
help(factorial)
```

    
        factorial function
        
    Help on function factorial in module __main__:
    
    factorial(n)
        factorial function
    



```python
fact = factorial # (2) 변수에 할당 가능
fact
```




    <function __main__.factorial(n)>




```python
list(map(fact, range(1,10))) # (3) 함수의 인자로 전달 가능
```




    [1, 2, 6, 24, 120, 720, 5040, 40320, 362880]



### higher-order function (고위 함수)
* 정의 : 함수를 인자로 받거나 결과로 함수를 반환하는 함수
* 종류 : map(lambda x : x+1, [1,2,3]), sorted([1,2,3],key = len), filter(), reduce(), apply(*python3 지원하지않음*)
* 대안 : list comprehension with lambda 


```python
from functools import reduce

print("map function: ", list(map(fact, range(10))))
print("alternative: ",[fact(i) for i in range(10)])

print("filter function: ", list(map(fact, filter(lambda x : x % 2, range(6)))))
print("alternative: ", [fact(i) for i in range(6) if i % 2])

print("reduce function: ", reduce(lambda x,y : x+y, range(100)))
print("alternative: ", sum(range(100)))
```

    map function:  [1, 1, 2, 6, 24, 120, 720, 5040, 40320, 362880]
    alternative:  [1, 1, 2, 6, 24, 120, 720, 5040, 40320, 362880]
    filter function:  [1, 6, 120]
    alternative:  [1, 6, 120]
    reduce function:  4950
    alternative:  4950



```python
print(all([])) # iterable의 요소 하나라도 false로 판명되지 않았기 때문에 true (all is innocent until one is proven to be guilty)
any([]) # iterable의 요소가 하나라도 true로 판명되지 않았기 때문에 false (all is guilty until one is proven to be innocent)
```

    True





    False



### callable object (호출 연산자  `()`로 호출 가능한 객체)
* 종류 : 
    * 함수
        * 사용자 정의 함수(ex. def, lambda를 이용한 함수)
        * 내장 함수(ex. len())
    * 매서드
        * 일반 매서드 (ex. 클래스 본체에 정의된 함수)
        * 내장 매서드 (ex. dict.get())
    * 클래스 및 클래스 객체 : 클래스내에 `__call__` 메서드가 구현되어 있으면 클래스를 함수로 호출 가능
    * 제너레이터 함수 : 제너레이터 함수 (ex. yield 키워드를 포함한 함수)를 호출하면 제너레이터 객체를 반환 함. 

#### bingocall


```python
import random


class BingoCage:
    
    def __init__(self, items):
        self._item = list(items) #(1) items가 mutable object이기 때문에 사본(copy)을 만들어 인수로 전단된 items의 변형을 방지
        random.shuffle(self._item)
        
    
    def pick(self):
        try:
            return self._item.pop() #(2)
        except IndexError:
            raise LookupError("pick from empty BingoCage")
        
    
    def __call__(self): #(3)
        return self.pick()

bingo = BingoCage(range(3))
```

* (3)클래스가 콜러블형(callable)인지 확인 


```python
print(callable(bingo))
```

    True


* (2) 객체가 호출될 때마다 항목이 하나씩 없어진 상태를 기억. **decorator** 함수도 호출된 이 후의 상태를 기억하는 기능을 함. 


```python
print(bingo())
print(bingo())
print(bingo())
print(bingo())
```


    ---------------------------------------------------------------------------

    IndexError                                Traceback (most recent call last)

    <ipython-input-47-5d5197a3247e> in pick(self)
         12         try:
    ---> 13             return self._item.pop() #(2)
         14         except IndexError:


    IndexError: pop from empty list

    
    During handling of the above exception, another exception occurred:


    LookupError                               Traceback (most recent call last)

    <ipython-input-50-4ad15f0b82f3> in <module>
    ----> 1 print(bingo())
          2 print(bingo())
          3 print(bingo())
          4 print(bingo())


    <ipython-input-47-5d5197a3247e> in __call__(self)
         17 
         18     def __call__(self):
    ---> 19         return self.pick()
         20 
         21 bingo = BingoCage(range(3))


    <ipython-input-47-5d5197a3247e> in pick(self)
         13             return self._item.pop() #(2)
         14         except IndexError:
    ---> 15             raise LookupError("pick from empty BingoCage")
         16 
         17 


    LookupError: pick from empty BingoCage


### function introspection (함수 인트로스펙션)


```python
# factorial 함수의 속성 (attribute) 확인
dir(factorial)
```




    ['__annotations__',
     '__call__',
     '__class__',
     '__closure__',
     '__code__',
     '__defaults__',
     '__delattr__',
     '__dict__',
     '__dir__',
     '__doc__',
     '__eq__',
     '__format__',
     '__ge__',
     '__get__',
     '__getattribute__',
     '__globals__',
     '__gt__',
     '__hash__',
     '__init__',
     '__init_subclass__',
     '__kwdefaults__',
     '__le__',
     '__lt__',
     '__module__',
     '__name__',
     '__ne__',
     '__new__',
     '__qualname__',
     '__reduce__',
     '__reduce_ex__',
     '__repr__',
     '__setattr__',
     '__sizeof__',
     '__str__',
     '__subclasshook__']



* `__dict__`
    * 객체에 할당된 속성(attribute)를 보관


```python
def upper_case_name(obj):
    return ("%s %s"%((obj.first_name, obj.last_name).upper()))
upper_case_name.short_descrption = 'Customer name'

upper_case_name.__dict__
```




    {'short_descrption': 'Customer name'}



* 함수에만 존재하는 속성 리스트 (다른 객체에는 ex. 클래스 기본적으로 존재하지 않음)


```python
class C: pass
obj = C()
def func(): pass
sorted(set(dir(func)) - set(dir(obj)))
```




    ['__annotations__',
     '__call__',
     '__closure__',
     '__code__',
     '__defaults__',
     '__get__',
     '__globals__',
     '__kwdefaults__',
     '__name__',
     '__qualname__']



### function arguments 
* keyword-only argument (키워드 전용 인수) : 


```python
def tag(name, *content, cls=None, **attrs):
    """
    하나 이상의 HTML 태그를 생성
    """
    if cls is not None: # (3)
        attrs["class"] = cls
    
    if attrs: # (1)
        attr_str = ''.join(f' {attr}="{value}"' for attr, value in sorted(attrs.items()))
    else:
        attr_str = ""
    
    if content: # (2)
        return '\n'.join(f"<{name}{attr_str}>{c}</{name}>" for c in content)
    else:
        return f"<{name}{attr_str} />"
```

* positional argument (위치 인수) : 인수의 위치 또는 타입 (tuple -> `*` , dictionary -> `**`)으로 구분 


```python
tag('br')
```




    '<br />'



* (2) `*` -> tuple argument
    * 매개 변수에 `*`을 붙이면 해당 위치 이후의 모든 인수들을 모두 해당 매개 변수에 튜플로 전달함


```python
print(tag('p','hello'))
print(tag('p','hello','word'))
```

    <p>hello</p>
    <p>hello</p>
    <p>word</p>


* (1)`**`-> dictionary argument
    * 매개 변수에 `**`을 붙이면 명시적으로 이름이 지정되지 않은 키워드 인수들을 딕셔너리로 받음


```python
print(tag('p','hello', id = 33))
```

    <p id="33">hello</p>


* (3)keyword argument (키워드 인수) : 인수의 key값으로 구분, **반드시 위치 인수 이후에 나와야 함**


```python
print(tag('p','hello', 'word', cls = 'sidebar'))
```

    <p class="sidebar">hello</p>
    <p class="sidebar">word</p>


* 인수 앞에 `**`을 붙이면 딕셔너리 안의 모든 항목을 인수로 보냄


```python
my_tag = {'name' : 'img', 'title' : 'Sunset', 'src' : 'sunset.jpg', 'cls' : 'framed'}
print(tag(**my_tag))
```

    <img class="framed" src="sunset.jpg" title="Sunset" />


* inspect 모듈을 이용해 함수의 매개 변수 정보 얻기
    * kind 속성의 종류
        * POSITIONAL_OR_KEYWORD : 위치 인수나 키워드 인수 (대부분이 여기에 속함)
        * VAR_POSITIONAL : 위치 매개 변수인 튜플
        * VAR_KEYWORD : 키워드 매개 변수인 딕셔너리 
        * KEYWORD_ONLY : 키워드 전용 매개 변수 
    * bind 메서드로 함수의 인수를 검증할 수 있음
        * 실제 인터프리터가 함수 호출 시 인수를 매개 변수에 바인딩하는 방식


```python
from inspect import signature

sig = signature(tag) 

print(str(sig)) # 함수의 매개 변수를 출력

for name, param in sig.parameters.items():
    print(param.kind," : ",name, "=", param.default)
```

    (name, *content, cls=None, **attrs)
    POSITIONAL_OR_KEYWORD  :  name = <class 'inspect._empty'>
    VAR_POSITIONAL  :  content = <class 'inspect._empty'>
    KEYWORD_ONLY  :  cls = None
    VAR_KEYWORD  :  attrs = <class 'inspect._empty'>



```python
bound_args = sig.bind(**my_tag)
bound_args # my_tag를 tag 함수의 인수로 바인딩함 
```




    <BoundArguments (name='img', cls='framed', attrs={'title': 'Sunset', 'src': 'sunset.jpg'})>



### function argument annotaition (함수 애너테이션)
* python3는 매개 변수와 반환값에 메타 데이터 (annotation expression)를 추가할 수있는 구문을 제공


```python
def clip(text: str, max_len:'int > 0'=80) -> str: # (1)
    """
    max_len 앞이나 뒤의 마지막 공백에서 잘라낸 텍스트를 반환한다. 
    """
    end = None
    if len(text) > max_len:
        space_before = text.rfind(' ', 0, max_len) # returns start index of the last occurence of the value
        if space_before >= 0: # max_len 앞에서 공백을 찾음 
            end = space_before
        else: # max_len 앞에서 공백을 찾지 못함 
            space_after = text.rfind(' ', max_len)
            if space_after >=0: # max_len 뒤에서 공백을 찾음
                end = space_after
    
    if end is None: # 공백이 없음
        end = len(text)
    return text[:end].rstrip()
```

* (1)annotation expression
    * parameter (매개 변수) : 매개변수는 콜론 (`:`) 뒤에 애너테이션 표현식을 추가 (ex. ':str', ':'value > 0')
    * return value (반환값) : 반환값은 매개변수를 닫는 괄호 뒤, 함수 선언 가장 마지막의 콜론 (`:`) 사이에 `->` 기호를 쓴 후 애너테이션 표현식을 추가 (ex. '...) -> float:')


```python
# 함수의 애너테이션 확인
clip.__annotations__
```




    {'text': str, 'max_len': 'int > 0', 'return': str}



### package for functional programming (함수형 프로그래밍을 위한 패키지)
* built-in module
    * operator
    * functools

#### operator 

* itemgetter(indices)
    * 시퀀스에서 항목을 가져옴
    * callable function `itemgetter(indices)(obj)`
    * lambda 대체 가능 
    * `__getitem__` 속성을 가진 객체면 사용 가능


```python
from operator import itemgetter

member_data = [
    ('SJ', 'female', 25, 'programmer'),
    ('DW', 'male', 4, 'puppy'),
    ('JH', 'male', 27, 'data-scientist')
]

for member in sorted(member_data, key=lambda x : x[2], reverse=True): # what I used to do using lambda 
    print(member)
print()    

for member in sorted(member_data, key=itemgetter(2), reverse=True): # new method using operator.itemgetter()
    print(member)

print()
info_job = itemgetter(0,-1) # parameter = tuple of index
for member in member_data:
    print(info_job(member))
```

    ('JH', 'male', 27, 'data-scientist')
    ('SJ', 'female', 25, 'programmer')
    ('DW', 'male', 4, 'puppy')
    
    ('JH', 'male', 27, 'data-scientist')
    ('SJ', 'female', 25, 'programmer')
    ('DW', 'male', 4, 'puppy')
    
    ('SJ', 'programmer')
    ('DW', 'puppy')
    ('JH', 'data-scientist')


* attrgetter(attributes)
    * 속성을 가진 객체에서 속성을 가져옴
    * callable function : `attrgetter(attributes)(obj)`


```python
from operator import attrgetter
from collections import namedtuple

BasicInfo = namedtuple('BasicInfo','name age extra')
ExtraInfo = namedtuple('ExtraInfo', 'gender job')

members = [BasicInfo(name, age, ExtraInfo(gender, job))
              for name, gender, age, job in member_data]

members
```




    [BasicInfo(name='SJ', age=25, extra=ExtraInfo(gender='female', job='programmer')),
     BasicInfo(name='DW', age=4, extra=ExtraInfo(gender='male', job='puppy')),
     BasicInfo(name='JH', age=27, extra=ExtraInfo(gender='male', job='data-scientist'))]




```python
name_job = attrgetter('name', 'extra.job')

for member in sorted(members, key=attrgetter('extra.job')): 
    print(name_job(member))
```

    ('JH', 'data-scientist')
    ('SJ', 'programmer')
    ('DW', 'puppy')


* methodcaller(method name, `*args`, `**kwargs`)
    * 객체의 메서드를 호출
    * callable function : `methodcaller(method name)(obj)`


```python
from operator import methodcaller

s = 'The time has come'
upcase = methodcaller('upper')
print(upcase(s))

hiphenate = methodcaller('replace', ' ', '-')
print(hiphenate(s))


```

    THE TIME HAS COME
    The-time-has-come


#### functools

* partial(func, `*args`, `**kwargs`)
    * 원래 함수의 일부 인수를 특정 값으로 고정한 함수를 생성
    * 하나 이상의 인수를 받는 함수를 그보다 적인 인수를 받는 콜백 함수(특정 함수에 인자로 입력되는 함수)로 만들어 API에 사용하고자 할 때 유용함


```python
from operator import mul
from functools import partial

triple = partial(mul, 3) # mul() 함수의 첫번째 인수를 3으로 바인딩해 triple이란 새로운 함수를 만든다
print(triple(7))

print(list(map(triple, range(1,10)))) # triple은 인수를 하나만 받는 콜백 함수 
```

    21
    [3, 6, 9, 12, 15, 18, 21, 24, 27]


* 유니코드 정규화 방식
    * nfc : Normalization Form Canonical Composition
    * 문자열 유니코드 정규화가 필요한 이유 : [참고 url](https://velog.io/@leejh3224/%EB%B2%88%EC%97%AD-%EC%9C%A0%EB%8B%88%EC%BD%94%EB%93%9C-%EC%8A%A4%ED%8A%B8%EB%A7%81%EC%9D%84-%EB%85%B8%EB%A9%80%EB%9D%BC%EC%9D%B4%EC%A7%95-%ED%95%B4%EC%95%BC%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0)


```python
import unicodedata

nfc = partial(unicodedata.normalize, 'NFC')
s1 = 'café'
s2 = 'cafe\u0301'
print(s1, s2)
print(s1 == s2)
print(nfc(s1) == nfc(s2))
```

    café café
    False
    True



```python

```
