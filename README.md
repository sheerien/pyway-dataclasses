# pyway-dataclasses
dataclasses crash course 

---

# 1- Introduction to the Python dataclass

> ```Python``` introduced the ```dataclass``` in ```version 3.7 (PEP 557)```. The ```dataclass``` allows you to define ```classes``` with less code and more functionality out of the box.

## regular python class
```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __repr__(self) -> str:
        return f"Name: {self.name}, Age: {self.age}"

    def __eq__(self, other) -> bool:
        if other.__class__ is self.__class__:
            return( self.name, self.age) == (other.name, other.age)
        else:
            return NotImplemented

    def __ne__(self, other) -> bool:
        result = self.__eq__(other)
        if result is NotImplemented:
            return NotImplemented
        else: 
            return not result


    
    def __lt__(self, other) -> bool:
        return self.age < other.age

    def __gt__(self, other) -> bool:
        return self.age > other.age
```
## python dataclasses

> if you use the dataclass, you’ll have all of these features (and even more) without implementing these dunder methods.
>> To make the Person class a ```dataclass```, you follow these steps:

```python
# First, import the dataclass decorator from the dataclasses module:

from dataclasses import dataclass

# Second, decorate the Person class with the dataclass decorator and declare the attributes:

@dataclass
class Person:
    name: str
    age: int
```
---

> if you ```compare``` two ```Person's``` ```objects``` with the same attribute value, it’ll return True. For example:

```python
p1 = Person('John', 25)
p2 = Person('John', 25)
print(p1 == p2)
```
### output

```python
True
```
---

## 2- Default values

> When using a regular class, you can define default values for attributes. For example, the following Person class has the iq parameter with the default value of 100.

```python:

class Person:
    def __init__(self, name, age, iq=100):
        self.name = name
        self.age = age
        self.iq = iq
```

> To define a default value for an attribute in the ```dataclass```, you assign it to the attribute like this:

```python:

from dataclasses import dataclass


@dataclass
class Person:
    name: str
    age: int
    iq: int = 100


print(Person('John Doe', 25))
```

### or

```python:

from dataclasses import dataclass, field


class Person:
    name: str
    age: int
    iq: int = field(default=100)
```

### In the case of using an external function

```python:

import random
import string
from dataclasses import  asdict, dataclass, field


def generate_id() -> str:
    return "".join(random.choices(string.ascii_uppercase, k=12))

class Person:
    name: str
    age: int
    id: int = field(default_factory=generate_id)


```

---

## 3- Convert to a tuple or a dictionary

> The dataclasses module has the ```astuple()``` and ```asdict()``` functions that convert an instance of the dataclass to a tuple and a dictionary. For example:

```python:
from dataclasses import dataclass, astuple, asdict


@dataclass
class Person:
    name: str
    age: int
    iq: int = 100


p = Person('John Doe', 25)

print(astuple(p))
print(asdict(p))

```
### output

```python:
('John Doe', 25, 100)
{'name': 'John Doe', 'age': 25, 'iq': 100}
```
---
## Create immutable objects

> To create readonly objects from a ```dataclass```, you can set the ```frozen``` argument of the ```@dataclass``` decorator to True. For example:

```python:

from dataclasses import dataclass, astuple, asdict


@dataclass(frozen=True)
class Person:
    name: str
    age: int
    iq: int = 100
```
> If you attempt to change the attributes of the object after it is created, you’ll get an error. For example:

```python:
p = Person('Jane Doe', 25)
p.iq = 120
```

### Error:

```shell:
dataclasses.FrozenInstanceError: cannot assign to field 'iq'
```
---
## 4- Customize attribute behaviors

> If don’t want to initialize an attribute in the ```__init__``` method, you can use the ```field()``` function from the dataclasses module.
>> The following example defines the ```can_vote``` attribute that is initialized using the ```__init__``` method:

```python:

from dataclasses import dataclass, field


class Person:
    name: str
    age: int
    iq: int = 100
    can_vote: bool = field(init=False)
```

> The ```field()``` function has multiple interesting parameters such as ```repr```, ```hash```, ```compare```, and ```metadata```.
>>If you want to initialize an attribute that depends on the value of another attribute, you can use the ```__post_init__``` method. As its name implies, Python calls the ```__post_init__``` method after the ```__init__``` method.
>>>The following use the ```__post_init__``` method to initialize the ```can_vote``` attribute based on the ```age``` attribute:

```python:

from dataclasses import dataclass, field


@dataclass
class Person:
    name: str
    age: int
    iq: int = 100
    can_vote: bool = field(init=False)

    def __post_init__(self):
        print('called __post_init__ method')
        self.can_vote = 18 <= self.age <= 70


p = Person('Jane Doe', 25)
print(p)
```

### output

```shell
called the __post_init__ method
Person(name='Jane Doe', age=25, iq=100, can_vote=True)
```
---
## 5- Sort objects

> By default, a dataclass implements the ```__eq__``` method.
>> To allow different types of comparisons like ```__lt__```, ```__lte__```, ```__gt__```, ```__gte__```, you can set the order argument of the ```@dataclass``` decorator to True:

```python:

@dataclass(order=True)
```

> By doing this, the ```dataclass``` will ```sort``` the ```objects``` by every field until it finds a value that’s not equal.
>>In practice, you often want to compare objects by a particular attribute, not all attributes. To do that, ```you need to define a field called sort_index``` and ```set its value to the attribute that you want to sort```.
>>>For example, suppose you have a ```list``` of ```Person's objects``` and want to ```sort``` them by ```age```:

```python:

members = [
    Person('John', 25),
    Person('Bob', 35),
    Person('Alice', 30)
]
```

#### To do that, you need to:

* First, pass the ```order=True``` parameter to the ```@dataclass``` decorator.

* Second, define the ```sort_index``` attribute and set its init parameter to False.

* Third, set the sort_index to the age attribute in the ```__post_init__``` method to sort the Person‘s object by age.

### Example:

```python:

from dataclasses import dataclass, field


@dataclass(order=True)
class Person:
    sort_index: int = field(init=False, repr=False)

    name: str
    age: int
    iq: int = 100
    can_vote: bool = field(init=False)

    def __post_init__(self):
        self.can_vote = 18 <= self.age <= 70
        # sort by age
        self.sort_index = self.age


members = [
    Person(name='John', age=25),
    Person(name='Bob', age=35),
    Person(name='Alice', age=30)
]

sorted_members = sorted(members)
for member in sorted_members:
    print(f'{member.name}(age={member.age})')
```

### Output:

```shell:

John(age=25)
Alice(age=30)
Bob(age=35)
```

> Note:> if ```frozen=True``` You have to use
this code internal ```__post_init```  => ``` object.__setattr__(self, "sort_index", self.age) ```

---
## 6- ```dataclass``` to JSON in Python

```python:
from dataclasses import dataclass, asdict
import json

@dataclass
class Person:
        name:str
        age: int
        iq: int

    @property
    def __dict__(self):
        return asdict(self)

    @property
    def json(self):
        return dumps(self.__dict__)

person = Person(name="Kelvin", age=22, iq=100)
print(person.json)

```
---
## Summary

* Use the ```@dataclass``` decorator from the dataclasses module to make a class a dataclass. The dataclass object implements the ```__eq__``` and ```__str__``` by default.

* Use the ```astuple()``` and ```asdict()``` functions to convert an object of a ```dataclass``` to a ```tuple``` and ```dictionary```.

* Use ```frozen=True``` to define a class whose objects are immutable.

* Use ```__post_init__``` method to initialize attributes that depends on other attributes.

Use ```sort_index``` to specify the sort attributes of the dataclass objects.
