---
layout: page
title:  "Итераторы и Генераторы"
date:   2018-01-12 10:54:00 +1000
categories: python
publish: True
---

### Итераторы и итерируемые выражения
Что такое итераторы и генераторы? Для начала рассмотрим, что же такое итерируемый объект. Итерируемый объект, это такой объект, от которого встроенная функция  `iter()` возвращает итератор. Для этого объект сам должен реализовывать метод `__iter__()` или `__getitem__()`, который принимает индекс с нуля.

Работа функции `iter()` сводится к следующему: сначала она смотрит, а не реализует ли объект метод `__iter__()`, если нет, то смотрится наличие метода `__getitem__()`. Если и он не реализован, возвращается исключение `TypeError`.

Что с итераторами в питоне? 

~~~ python
class Iterator(Iterable):

    __slots__ = ()

    @abstractmethod
    def __next__(self):
        'Return the next item from the iterator. When exhausted, raise StopIteration'
        raise StopIteration

    def __iter__(self):
        return self

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Iterator:
            return _check_methods(C, '__iter__', '__next__')
        return NotImplemented
~~~

В питоне итератор - это объект, который реализует метод `__next__()`, который возвращает следующее по счету значение либо вызывает исключение `StopIteration`.
Итератор сам является итерируемым объектом и реализует метод `__iter__()`, 

Таким образом мы можем создать итерируемый объект и его итератор так:

~~~ python
class ListIterator(collections.abc.Iterator):
    def __init__(self, collection, cursor):
        self._collection = collection
        self._cursor = cursor

    def __next__(self):
        if self._cursor + 1 >= len(self._collection):
            raise StopIteration()
        self._cursor += 1
        return self._collection[self._cursor]

class ListCollection(collections.abc.Iterable):
    def __init__(self, collection):
        self._collection = collection

    def __iter__(self):
        return ListIterator(self._collection, -1)
~~~

эти объекты модно использовать следующим образом:

~~~ py
arr = [1,2,3,5,2,2,3,4,2,1]
itobj = ListCollection(arr)

for item in itobj:
    print(item, end=', ')
print()
>> 1,2,3,5,2,2,3,4,2,1,

# или так
itr=iter(itobj)

while True:
    try:
        print(next(itobj), end=', ')
    except StopIteration:
        break

>> 1,2,3,5,2,2,3,4,2,1,
~~~

если в функцию `next()` передать второй аргумент, она будет возвращать его вместо исключения в конце итерации.

если в функцию `iter()` передать второй аргумент, то итератор будет возвращать значения, пока не выпадет то, что было передано вторым параметром.

### Генераторы
Генераторы упрощают создание итераторов. Реализация их в Python возможна с помощью функции с ключевым словом `yield` или генераторного выражения. В результате вызова функции или вычисления генераторного выражения получаем объект `generator`.

~~~ python
# пример генераторного выражения
a=(i**2 for i in range(10))
print(type(a))
>>generator

# посмотрим какие методы реализует объект
dir(a)
>>['__class__',
 '__del__',
 '__delattr__',
 '__dir__',
 '__doc__',
 '__eq__',
 '__format__',
 '__ge__',
 '__getattribute__',
 '__gt__',
 '__hash__',
 '__init__',
 '__init_subclass__',
 '__iter__', # метод создания итератора
 '__le__',
 '__lt__',
 '__name__',
 '__ne__',
 '__new__',
 '__next__', # метод долучения следующего значения
 '__qualname__',
 '__reduce__',
 '__reduce_ex__',
 '__repr__',
 '__setattr__',
 '__sizeof__',
 '__str__',
 '__subclasshook__',
 'close',
 'gi_code',
 'gi_frame',
 'gi_running',
 'gi_yieldfrom',
 'send',
 'throw']

# Пример функции-генератора

def fib():
    prev, cur = 0, 1    
    while True:
        yield prev
        prev,cur = cur, prev+cur


for num, fib in enumerate(fib()):
    print('{0}: {1}'.format(num, fib))
    if num > 9:
        break
~~~

любая функция, где встречается выражение `yield` называется генераторной. При вызове, она возвращает объект генератор. Он реализует интерфейс итератора, т.е. с ним можно работать как с любым итерируемым объектом.


1. при вызове `fib()` функции создается объект-генератор
2. ключевое слово `enumerate` вызывает `iter()` с этим объектом и получает итератор
3. в цикле вызывается функция `next()` до тех пор, пока не будет получено исключение `StopIteration` или цикл не завершится изнутри.
4. при каждом `next()` выполнение функции продолжается с последнего `yield`.

Создается state-машина, при вызове `next()` меняется её состояние.
