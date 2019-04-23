---
layout: post
title: Understanding Python Iterators and Generators
image: https://source.unsplash.com/dJdcb11aboQ
---

## The Problem

Let's say you neeed to search for occurences an item in a *unsorted* list. Your first impression might be to create a list of the inices and return it.

This should pose some questions however:
- What if later on you only wanted the first result? Do we need to rewrite all of that logic?
- Do we need every element in a list when we'll just loop over them and discoard?

Generators are a way to what I'd like to think of as "lazy lists" in that they only give you the elements when you ask. Instead of making a gigantic list in memory with every item when called it returns each element individually.

> Iterators and generators **lazily** return values allowing us to save memory.

## Making an Iterator

Let's start with a simple task we have the following variables:

```python
s = 'baacabcaab'
p = 'a'
```

We need to find every index of `p` within `s` *without* using premade/purpose-built methods for the learning purposes of this article.

### Class

The old way of creating a iterator was through an explicitly defined class.

```python
class find:
  def __init__(self, string, character):
    self.string = string
    self.character = character
    self.index = 0

  def __iter__(self):
    return self

  def __next__(self):
    while 0 <= self.index < len(self.string):
      if self.string[self.index] == self.character:
        break

      self.index += 1
    else:
      raise StopIteration

    index = self.index
    self.index += 1
    return index

f = find(s, p)
print(list(find(s, p)))
# Out: [1, 2, 4, 7, 8]
```

#### Pros:
- Can have additional functions

#### Cons:
- Complicated
- Lots of boilerplate
- Not very "Pythonic"

Personally I would only use this when absolutely necessary.

### Yield

```python
def find(string, character):
  for index, str_char in enumerate(string):
    if str_char == character:
      yield index

f = find(s, p)
print(list(f))
# [1, 2, 4, 7, 8]
```

#### Pros

- Lorem

#### Cons

- Ipsum

### Generator Expression

Generator expressions are great for creating simple, inline generators, but they can easily get far too compact an unreadable

```python
f = (index for index, str_char in enumerate(string) if str_char == character)

print(list(f))
# [1, 2, 4, 7, 8]
```

#### Pros

- Lorem

#### Cons

- Ipsum

## Going Further

At this point you might be wondering why converted `f` to a list. A hint can be found in our first, class-based version. There's nothing that clearly shows that it is subscriptable and that's because it's an iterator!

If you look at the documentation constructor for list it takes in an `iterable` and guess what it is!

You could almost think of

```python
f_list = []

while True:
  try:
    f_list.append(next(f))
  except StopIteration:
    break
```

## Back to Class

In the case of our class method calling `next(f)` is equivalent to calling `f.__next__()`. So keeps calling `f.__next__()` until it `StopIteration` is raised.

## You better Yield

A function with a yield statement is a generator function that returns a generator object.

> Generator functions allow you to declare a function that behaves like an iterator, i.e. it can be used in a for loop. - [Python Documentation](https://wiki.python.org/moin/Generators)

This means that it implements `__iter__` and `next` in a similar fashion as our class. The difference being that it tracks previous calls ommitting the need for all this `self` nonsense.

On the first call of `next` on a generator function starts execution and returns the value after yield and "pauses" execution. The next time `next` is called it "resumes" execution until the next `yield`. It repeats this cycle until it reaches the end where it raises `StopIteration` just like the class. The massive benefit here is that Python allows us to create generators with a much simpler syntax

> Python allows us to create generators with a much simpler syntax

Additionally you can use `yield from` to direct future calls of `next` to the result of another iterator.

### Iterables

When iterating over an iterable Python makes it into an **ITERATOR** which is exactly what we created in our first example.

## Resources

https://wiki.python.org/moin/Generator

https://www.programiz.com/python-programming/iterator

https://www.geeksforgeeks.org/python-difference-iterable-iterator/
