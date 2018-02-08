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

Что важно знать. Итераторы на каждой итерации возвращают свое значение, однако информации о размере итератора (количестве элементов в нем) как и индексация не предоставляются. Итератор можно использовать всего лишь раз, для повторного использования его нужно создать снова.

Генераторы существенно снижают потребление памяти, с их помощью можно достаточно просто решить ряд математических задач.

Например, вывести все тройки натуральных чисел, которые удовлетворяют теореме Пифагора.

~~~ py
# сперва определим функцию-генератор, которая будет возвращать последовательный 
# набор чисел (бесконечная последовательность натуральных чисел)
def natural():
    i=1
    while True:
        yield i
        i+=1

# определим функцию, возвращающую список из n зачений итерируемого объекта seq
def take(n, seq):
    seq=iter(seq)
    result=[]
    
    try:
        for i in range(n):
            result.append(seq.next())
    except StopIteration:
        pass
    
    return result

pythagor=((x,y,z) for z in natural() for y in range(1,z) for x in range(1,y) if x*x+y*y==z*z )

triplets = take(10, pythagor)
print(triplets)
[(3, 4, 5),
 (6, 8, 10),
 (5, 12, 13),
 (9, 12, 15),
 (8, 15, 17),
 (12, 16, 20),
 (15, 20, 25),
 (10, 24, 26),
 (20, 21, 29),
 (18, 24, 30)]

~~~

Это удобно, выглядит красиво
Этот же код, но записанный с помощью итератора (объекта, реализующего метод __next__() и __iter__()) этот код выглядел бы так:

~~~ py
class PythagorNum:
    def __init__(self):
        self.z=0
    
    def __next__(self):
        self.z=self.z+1
        while True:
            for y in range(1,self.z):
                for x in range(1,y):
                    if x*x+y*y==self.z*self.z:
                        return (x,y,self.z)
            self.z+=1
    
    def __iter__(self):
        return self

pythagor=PythagorNum()
take(10, pythagor)
[(3, 4, 5),
 (6, 8, 10),
 (5, 12, 13),
 (9, 12, 15),
 (8, 15, 17),
 (12, 16, 20),
 (15, 20, 25),
 (10, 24, 26),
 (20, 21, 29),
 (18, 24, 30)]
~~~

Очевидно, код, записанный с  использованием генераторов более читабелен, понятен и лаконичен.
