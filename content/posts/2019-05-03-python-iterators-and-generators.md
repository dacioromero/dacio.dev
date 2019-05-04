---
layout: post
title: Fantastic Iterators and How to Make Them
date: 2019-05-03 00:32 -0700
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

While learning at Make School I've seen my peers write functions that create lists of items.

```python
s = 'baacabcaab'
p = 'a'

def find_char(string, character):
  indices = list()

  for index, str_char in enumerate(string):
    if str_char == character:
      indices.append(index)

  return indices

print(find_char(s, p)) # [1, 2, 4, 7, 8]
```

This implementation works, but it poses a few problems:

- What if we only want the first result; will we need to make an entirely new function?
- What if all we do is loop over the result once, do we need to store every element in memory?

Iterators are the ideal solution to these problems. They function like "lazy lists" in that instead of returning a list with every value it produces and returns each element one at a time.

> Iterators **lazily** return values; saving memory.

***So let's dive into learning about them!***

## Built-In Iterators

The iterators that are most often are `enumerate()`, and `zip()`. Both of these *lazily* return values by `next()` with them.

**`range()`, however, is *not* an iterator, but an *"lazy iterable."*** - [Explanation](https://treyhunner.com/2018/02/python-range-is-not-an-iterator/)

We can convert `range()` into an iterator with `iter()`, so we'll do that for our examples for the sake of learning.

```python
my_iter = iter(range(10))
print(next(my_iter)) # 0
print(next(my_iter)) # 1
```

Upon each call of `next() `we get the next value in our range; makes sense right? If you want to convert an iterator it to a list you just give it the list constructor.

```python
my_iter = iter(range(10))
print(list(my_iter)) # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

If we mimic this behavior we'll start to understand more about how iterators work.

```python
my_iter = iter(range(10))
my_list = list()

try:
  while True:
    my_list.append(next(my_iter))
except StopIteration:
  pass

print(my_list) # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

You can see that we needed to wrap it in a try catch statement. That's because iterators raise `StopIteration` when they've been exhausted.

So if we call next on our exhausted range iterator, we'll get that error.

```python
next(my_iter) # Raises: StopIteration
```

## Making an Iterator

Let's try making an iterator that behaves like `range` with only the stop argument by using three common types of iterators: [Classes](#class), [Generator Functions (Yield)](#generator-function) and [Generator Expressions](#generator-expression)

### Class

The old way of creating an iterator was through an explicitly defined class. For an object to be an iterator it must implement `__iter__()` that returns itself and `__next__()` which returns the next value.

```python
class my_range:
  _current = -1

  def __init__(self, stop):
    self._stop = stop

  def __iter__(self):
    return self

  def __next__(self):
    self._current += 1

    if self._current >= self._stop:
      raise StopIteration

    return self._current

r = my_range(10)
print(list(r)) # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

That wasn't too hard, but unfortunately, we have to keep track of variables between calls of `next()`. Personally, I don't like the boilerplate or changing how I think about loops because it isn't a drop-in solution, so I prefer [generators](#generators)

The main benefit is that we can add additional functions that modify its internal variables such as `_stop` or create new iterators.

> Class iterators have the downside of needing boilerplate, however, they can have additional functions that modify state.

### Generators

[PEP 255](https://www.python.org/dev/peps/pep-0255/) introduced "simple generators" using the `yield` keyword.

> Today, generators are iterators that are just easier to make than their class counterparts.

#### Generator Function

Generator functions are what was ultimately being discussed in that PEP and are my favorite type of iterator, so let's start with that.

```python
def my_range(stop):
  index = 0

  while index < stop:
    yield index
    index += 1

r = my_range(10)
print(list(r)) # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

Do you see how beautiful those 4 lines of code are? It's slightly significantly shorter than our list implementation to top it off!

> Generator functions iterators with less boilerplate than classes with a normal logic flow.

Generator functions automagically **"pause"** execution and return the specified value with every call of `next()`. This means that *no code* is run until the **first** `next()` call.

This means the flow is like this:

1. `next()` is called,
1. Code is executed up to the next `yield` statement.
1. The value on the right of `yield` is returned.
1. Execution is paused.
1. 1-5 repeat for every `next()` call until the last line of code is hit.
1. `StopIteration` is raised.

Generator functions also allow for you to use the `yield from` keyword which future `next()` calls to another iterable until said iterable has been exhausted.

```python
def yielded_range():
  yield from my_range(10)

print(list(yielded_range())) # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

That wasn't a particularly complex example. But you can even do it *recursively*!

```python
def my_range_recursive(stop, current = 0):
  if current >= stop:
    return

  yield current
  yield from my_range_recursive(stop, current + 1)

r = my_range_recursive(10)
print(list(r)) # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

#### Generator Expression

Generator expressions allow us to create iterators as one-liners and are good when we don't need to give it external functions. Unfortunately, we can't make another `my_range` using an expression, but we can work on iterables like our last `my_range` function.

```python
my_doubled_range_10 = (x * 2 for x in my_range(10))
print(list(my_doubled_range_10)) # 0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```
The cool thing about this is that it does the following:

1. The `list` asks `my_doubled_range_10` for its next value.
1. `my_doubled_range_10` asks `my_range` for its next value.
1. `my_doubled_range_10` returns `my_range`'s value multiplied by 2.
1. The `list` appends the value to itself.
1. 1-5 repeat until `my_doubled_range_10` raises `StopIteration` which happens when `my_range` does.
1. The `list` is returned containing each value returned by `my_doubled_range`.

We can even do *filtering* using generator expressions!

```python
my_even_range_10 = (x for x in my_range(10) if x % 2 == 0)
print(list(my_even_range_10)) # [0, 2, 4, 6, 8]
```

This is very similar to the previous except `my_even_range_10` only returns values that match the given condition, so only even values between in the range [0, 10).

Throughout all of this, we only create a list because we told it to.

## The Benefit

<center>
  <img src="https://nvie.com/img/relationships.png" alt="Relationships">
  <a href="https://nvie.com/posts/iterators-vs-generators/">Source</a>
</center>

Because generators are iterators, iterators are iterables, and iterators lazily return values. This means that using this knowledge we can create objects that will only give us objects when we ask for them and however many we like.

This means we can pass generators into functions that reduce each other.

```python
print(sum(my_range(10))) # 45
```

Calculating the sum in this way avoids creating a list when all we're doing is adding them together and then discarding.

We can rewrite the very [first example](#the-problem) to be much better using a generator function!

```python
s = 'baacabcaab'
p = 'a'

def find_char(string, character):
  for index, str_char in enumerate(string):
    if str_char == character:
      yield index

print(list(find_char(s, p))) # [1, 2, 4, 7, 8]
```

Now immediately there might be no obvious benefit, but let's go to my first question: "what if we only want the first result; will we need to make an entirely new function?"

> With a generator function we don't need to rewrite as much logic.

```python
print(next(find_char(s, p))) # 1
```

Now we *could* retrieve the first value of the list that our original solution gave, but this way we only get the first match and stop iterating over the list. The generator will be then discarded and nothing else is created; massively saving memory.

## Conclusion

If you're ever creating a function the accumulates values in a list like this.

```python
def foo(bar):
  values = []

  for x in bar:
    # some logic
    values.append(x)

  return values
```

Consider making it return an iterator with a class, generator function, or generator expression like so:

```python
def foo(bar):
  for x in bar:
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
