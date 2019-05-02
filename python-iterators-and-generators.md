---
layout: post
title: Fantastic Iterators and How to Make Them
---

<!--
TODO:
- Add visual aids
- Get peer reviewed
- Publish
-->

<center>
  <img src="https://source.unsplash.com/dJdcb11aboQ" alt="Hero Image">
  Photo by John Matychuk on Unsplash
</center>

## The Problem

While learning at Make School I've seen my peers write functions that create lists of items that they very often iterate over them.

```python
s = 'baacabcaab'
p = 'a'

def find(string, character):
  indices = []

  for index, str_char in enumerate(string):
    if str_char == character:
      indices.append(index)

  return indices

print(find(s, p)) # [1, 2, 4, 7, 8]
```

This implementation works, but it poses a few problems:

- What if we only want the first result; will we need to make an entirely new function?
- What if all we do is loop over the result once, do we need to store every element in memory?

Iterators are the ideal solution to these problems. They function like "lazy lists" in that instead of returning a list with every value it produces and returns each element one at a time.

> Iterators **lazily** return values; saving memory.

***So let's dive into learning about them***

## Built-In Iterators

The iterators that are most often are `enumerate()`, and `zip()`. Both of these *lazily* return values by calling them on `next()`.

**Range, however, is *not* an iterator, but an *"lazy iterable"*** - [Python: range is not an iterator!](https://treyhunner.com/2018/02/python-range-is-not-an-iterator/)

We can convert `range()` into an iterator with `iter()`, so we'll do that for our examples for the sake of learning.

```python
my_iter = iter(range(10))
print(next(my_iter)) # 0
print(next(my_iter)) # 1
```

You can see that on each call of next we get the next value in our range. If you want to convert an iterator it to a list you just give it the the list constructor.

```python
my_range = iter(range(10))
print(list(my_range)) # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

You can mimick this behavior.

```python
my_range = iter(range(10))
my_list = list()

try:
  while True:
    my_list.append(next(my_range))
except StopIteration:
  pass

print(my_list) # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

You can see that we needed to wrap it in a try catch statement. That's because iterators raise `StopIteration` when they've exhausted themselves.

So if we call next on our range now that its exhaused, we'll get that error.

```python
next(my_iter) # Raises: StopIteration
```

## Making an Iterator

Let's try making a iterator that behaves like `range` with only the stop argument by using three common types of iterators: [Classes](#class), [Generator Functions (Yield)](#generator-function-yield) and [Generator Expressions](#generator-expression)

### Class

The old way of creating an iterator was through an explicitly defined class.

```python
class my_range:
  _current = -1

  def __init__(self, stop):
    self._stop = stop

  def __iter__(self):
    return self

  def __next__(self):
    self._current += 1

    if self._current == self._stop:
      raise StopIteration

    return self._current

r = my_range(10)
print(list(r)) # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

This isn't too bad, but we have to keep track of iternal variables. Personally I get tired of writing all of the boilerplate and changing my thinking that the loop is calling `__next__` repeatedly.

A benefit is that we can add additonal functions that modify its interal variables such as `_stop`.

> Class iterators have the downside of needing boilerplate, however they can have additional functions that modify state.

### Generators

[PEP 255](https://www.python.org/dev/peps/pep-0255/) introduced "simple generators" using the `yield` keyword.

> Today, generators are iterators that are must easier to make than their class counterparts.

#### Generator Function

```python
def my_range(stop):
  index = 0

  while index < stop:
    yield index
    index += 1

r = my_range(10)
print(list(r)) # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

Do you see how beautiful those 4 lines of code are? It's even slightly shorter than the list method and acts exactly the same!

> Generator functions iterators with less boilerplate than classes with a normal logic flow.

Generator functions automagically **"pause"** execution and return the specified value with every call of `next()`. This means that *no code* is run until the **first** `next()` call.

This means the flow is like this:

1. `next()` is called,
1. Code is executed up to the next `yield` statement.
1. The value on the right of `yield` is returned.
1. Execution is paused.
1. 1-5 repeat for every `next()` call until last line of code is hit.
1. `StopIteration` is raised.

Generator functions also allow for you to use the `yield from` keyword which future `next()` calls to another iterable until said iterable has been exhausted.

```python
def example():
  yield from my_range(10)

print(list(example())) # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

That wasn't a particularly complex example. But you can even do it *recursively*!

```python
def my_range_recursive(stop, current = 0):
  if current >= stop:
    return

  yield current
  yield from my_range_recursive(stop, current + 1)

r = my_range_recursive(10)
print(list(r))
```

#### Generator Expression

Generator expressions allow us to create generators in one line and are good when it doesn't need to be reused. Unfortuately we can't make our custom range, but we can work on it.

```python
my_doubled_range_10 = (x * 2 for x in my_range(10))
print(list(my_doubled_range_10)) # 0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```
The cool thing about this is that it does the following:

1. The `list` asks `my_doubled_range_10` for its next value.
1. `my_doubled_range_10` asks `my_range` for its next value.
1. `my_doubled_range_10` returns `my_range`'s value multiplied by 2.
1. The `list` appends the value to itself.
1. 1-5 repeat until `my_doubled_range_10` raises `StopIteration` which happens when `my_range` does
1. The `list` returns what it created from those values

We can even do *filtering* using generator expressions!

```python
my_even_range_10 = (x for x in my_range(10) if x % 2 == 0)
print(list(my_even_range_10)) # [0, 2, 4, 6, 8]
```

This is very similar to the previous except `my_even_range_10` only returns values that match the given condition, so only even values between in the range [0, 10).

## The Benefit

Because generators are iterators, iterators are iterables, and iterators lazily return values. This means that using this knowledge we can create objects that will only give us objects when we ask for them and however many we like.

<center>
  <img src="https://nvie.com/img/relationships.png" alt="Relationships">
  <a href="https://nvie.com/posts/iterators-vs-generators/">Source</a>
</center>

We can rewrite the very [first example](#the-problem) to be much better using a generator function!

```python
s = 'baacabcaab'
p = 'a'

def find(string, character):
  for index, str_char in enumerate(string):
    if str_char == character:
      yield index

print(list(find(s, p))) # [1, 2, 4, 7, 8]
```

Now immediately there might be no obvious benefit, but lets go to my first question: "-What if we only want the first result; will we need to make an entirely new function?"

With a generator funcion we don't.

```python
print(next(find(s, p))) # 1
```

Now we *could* retrieve the first value of the list that our original soluton gave, but this way we only get the first match and stop iterating over the list, massively saving memory.

We can even pass it into functions like `sum()` without storing each value in a list beforehand.

```python
print(sum) # 22
```

## Conclusion

If you're ever creating a function the accumulates values in a list like this.

```python
def func(iterable):
  values = []

  for x in iterable:
    # some logic
    values.append(x)

  return values
```

Consider making it return an iterator with a class, generator function, or generator expression like so:

```python
def func(iterable):
  for x in iterable:
    # some logic
    yield x
```

## Resources and Sources

#### PEPs
- [Generators](https://www.python.org/dev/peps/pep-0255/)
- [Generator Expressions PEP](https://www.python.org/dev/peps/pep-0289/)
- [Yield From PEP](https://www.python.org/dev/peps/pep-0380/)

## Articles and Threads

- [Iterators](https://www.programiz.com/python-programming/iterator)
- [Iterable vs Iterator](https://www.geeksforgeeks.org/python-difference-iterable-iterator/)
- [Generator Documentation](https://wiki.python.org/moin/Generator)
- [Iterators vs Generators](https://nvie.com/posts/iterators-vs-generators/)
- [Generator Expression vs Function](https://stackoverflow.com/a/1995585)
- [Recrusive Generators](https://stackoverflow.com/a/8991864)

### Definitions

- [Iterable](https://docs.python.org/3/glossary.html#term-iterable)
- [Iterator](https://docs.python.org/3/glossary.html#term-iterator)
- [Generator](https://docs.python.org/3/glossary.html#term-generator)
- [Generator Iterator](https://docs.python.org/3/glossary.html#term-generator-iterator)
- [Generator Expression](https://docs.python.org/3/glossary.html#term-generator-expression)
