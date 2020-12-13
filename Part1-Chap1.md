# Part1. Python Data Model (파이썬 데이터 모델) 
> 현재 **Fluent Python** 을 공부하고, 파트별로 핵심 내용을 정리하고 있다.   
> 오늘은 첫번째 파트인 **파이썬 데이터 모델**, 즉 **파이썬의 객체 구조**를 공부한다.

<p>
    <div style="text-align:center"><img src="https://files.realpython.com/media/Object-Oriented-Programming-OOP-in-Python-3_Watermarked.0d29780806d5.jpg" alt="python object model" width="600" height="300" /></div>
		<div style="text-align:center"><em>img source : "https://realpython.com/python3-object-oriented-programming/"</em></div>
</p>

## Chapter1. Special method (특별 메소드)
> 특별 메소드를 적절히 사용해 클래스를 만들면 이 클래스가 build-in object(내장형 객체)가 되어 build-in function(내장 함수)을 사용할 수 있다는 장점이 있다.   
> 따라서 특별 메소드는 *pythonic*하게 코딩하기 위해 친숙해져야 하는 개념이다. 

* 종류 
	* 객체 생성 및 소멸
  ```python
	__init__
	__del__
	```
	
	* 객체를 문자열로 반환
  ```python
	__repr__
	__str__
	```
		
	* 반복
  ```python
	__iter__
	__next__
	__reversed__
	```
		
	* collection emulation (컬렉션 에뮬레이션)
  ```python
	__getitem__
	__len__
	__contains__
	__setitem__
	```
		
	* callable emulation (콜러블 에뮬레이션)
  ```python
	__call__
	```	
	
	* 속성 관리
  ```python
	__getattr__
	__setattr__
	__delattr__
	__dir__
	```

* 용도
	*  특별 메소드는 유저가 아니라 파이썬 인터프리터를 위한 메소드이다. 
	*  클래스에 특별 메소드(ex. `__len__(self)`)가 정의됐다면 사용자가 내장 함수(ex. `len(obj)`)를 호출할 때  파이썬 인터프리터가 특별 메소드(ex. `obj.__len__()`) 를 호출한다.
	*  따라서 사용자는 **특별 메소드가 구현된  클래스**에 **내장함수**를 사용할 수 있어 한층 편리해진다. 

* 팁
	* `__init__` 을 제외하고는 특별 메소드를 직접 구현하는 것보다 내장 함수(`len()`, `str()`, `iter()`, ...)를 사용하는 것이 효율적이다. 
	* 메소드나 attribute(속성)을 만들 때 dunder(ex. `__myfunc__()`, `self.__myattr__`)의 사용은 특별 메소드와 헷갈릴 수 있기 때문에 지양한다. 

---
#### 1-1. Card


```python
import collections

Card = collections.namedtuple('Card', ['rank','suit']) #(1)

class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list('JQKA') 
    suits = 'spades diamonds clubs hearts'.split()
    
    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits for rank in self.ranks]
        
    
    def __len__(self): #(2-1) 
        return len(self._cards)
    
    
    def __getitem__(self, position): #(2-2) 
        return self._cards[position]
        
```

* (1) collections.namedtuple(): 메소드 없이 속성으로만 구현된 클래스

```python
Card = collections.namedtuple('Card', ['rank','suit'])
beer_card = Card("7","diamonds")
beer_card.rank
```




    '7'



* (2) 내장 함수 사용
	* `__getitem__`, `__len__`특별 메소드를 생성해 FrenchDeck 클래스에 내장 함수인  `len()`, `list[i]`, `random.choice(list)` 등을 사용할 수 있게 됨

```python
deck = FrenchDeck()

print(len(deck)) #(2-1)
print(deck[0]) #(2-2) __getitem__ 메소드를 가지면 sequence object가 되어 slicing이 가능해진다. 
print(deck[51])


```

    52
    Card(rank='2', suit='spades')
    Card(rank='A', suit='hearts')


* (2-2) `__getitem__` -> Sequence Object (시퀀스 객체) 화

```python
from random import choice # sequence object에서 임의의 값을 가져오는 메소드
print(choice(deck))

print(deck[12::13]) # slicing

for card in reversed(deck): # for loop
    print(card)
    break


```

    Card(rank='A', suit='clubs')
    [Card(rank='A', suit='spades'), Card(rank='A', suit='diamonds'), Card(rank='A', suit='clubs'), Card(rank='A', suit='hearts')]
    Card(rank='A', suit='hearts')



```python
# in 사용
Card(rank='A', suit='clubs') in deck 
```




    True




```python
# priority로 sorting
suit_values = dict(spades=3, hearts=2, diamonds=1, clubs=0)

def spades_high(card):
    rank_values = FrenchDeck.ranks.index(card.rank)
    return rank_values * len(suit_values) + suit_values[card.suit] # rank가 우선 순위, rank가 같다면 suit비교

for card in sorted(deck, key = spades_high):
    print(card)
    break
```

    Card(rank='2', suit='clubs')


#### 1-2. Vector


```python
from math import hypot # Euclidean norm

class Vector:
    
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y
    
    
    def __repr__(self): #(1)
        return f"Vector({self.x},{self.y})"
    
    
    def __abs__(self): 
        return hypot(self.x, self.y)
    
    
    def __bool__(self): #(2)
        return bool(abs(self))
    
    
    def __add__(self, other):
        x = self.x + other.x
        y = self.y + other.y
        return Vector(x,y)
    
    
    def __mul__(self, scalar):
        return Vector(self.x*scalar, self.y*scalar)
    
```


* (1) 객체를 문자열(class 호출하는 모습과 동일)로 표현하는 특별 메소드 `__repr__()`

```python
print(Vector(2,3))
```




    Vector(2,3)



* (2) 파이썬 인터프리터는 오브젝트에 `__bool__()`이 정의되지 않는다면 `__len__()`을 호출 ex. list object

```python
 object
print(bool(Vector()))

obj = Vector()
if obj:
    print(f"truthy {obj}")
else:
    print(f"falsy {obj}")
```

    False
    falsy Vector(0,0)



---
## [additional] Python container datatypes
* python library container datatypes (__iterable objects__)
	* 종류
		* collections
			* 정의 : no-deterministic ordering (정해진 순서가 없음)
       * 종류: **set**, **dict**
		* sequence
       * 정의 : deterministic ordering (정해진 순서가 있음)
       * 종류: **list, tuple, string, range objects, bytes array(mutable), bytes sequence(immutable)**
	* [보충 설명](https://data-flair.training/blogs/python-sequence/)

## [additional] Mutable object vs Immutable object

* 정의
    * immutable : 변수에 값을 한 번 지정한 후에는 값 변경이 불가능. 만약 변수의 값을 변경하면 변수의 주소값(id)가 달라짐. 즉, 다른 변수가 됨.
    * mutable : 변수에 값을 지정한 후 값 변경이 가능. 주소값도 동일.
* 종류
    * immutable: **string, tuple, bytes sequence**
    * mutable: **list, range objects, bytes array, set, dict**
