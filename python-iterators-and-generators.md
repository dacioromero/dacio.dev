---
layout: post
title: Understanding Python Iterators and Generators
---

<!--
TODO:
- Add visual aids
- Get peer reviewed
- Publisg
-->

![Hero Image](https://source.unsplash.com/dJdcb11aboQ)
<center>Photo by John Matychuk on Unsplash</center>

## The Problem

Let's say you need to search for occurrences an item in an *unsorted* list. Your first impression might be to create a list of the indices and return it.

```python
s = 'baacabcaab'
p = 'a'

def find(string, character):
  indices = []

  for index, str_char in enumerate(string):
    if str_char == character:
      indices.append(index)

  return indices

print(find(s, p))
# [1, 2, 4, 7, 8]
```

But I would then ask you the following:

- What if later on you only wanted the first result? Do we need to rewrite all of that logic?
- Do we need every element in a list when we'll just loop over them and discard them afterward?

This is where iterators and generators come in. They're what I'd like to think of as "lazy lists" in that they only give you the elements when you ask. Instead of making a gigantic list in memory with every item when called it "returns" each element one at a time.

> Iterators and generators **lazily** return values allowing us to save memory.

## Making an Iterator

Let's try improving our code by using three common types of iterators: [Classes](#class), [Generator Functions (Yield)](#generator-function-yield) and [Generator Expressions](#generator-expression)

### Class

The old way of creating an iterator was through an explicitly defined class.

```python
class find:
  def __init__(self, string, character):
    self.string = string
    self.character = character
    self.index = 0

  def __iter__(self):
    return self

  def __next__(self):
    while self.index < len(self.string):
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

Personally, I would only use this when absolutely necessary. You could create additional functions that could modify the "state" of the iterator, but in most cases, you probably won't need to do that. I'd recommend trying one of the methods below.

#### Pros

- Can have additional functions state

#### Cons

- Lots of boilerplate
- Not very "Pythonic"

### Generator Function (Yield)

```python
def find(string, character):
  for index, str_char in enumerate(string):
    if str_char == character:
      yield index

f = find(s, p)
print(list(f))
# [1, 2, 4, 7, 8]
```

Do you see how beautiful those 4 lines of code are? It's even slightly shorter than the list method!

#### Pros

- Short and concise
- No boilerplate

#### Cons

- No code will run until `next()` is called

### Generator Expression

Generator expressions are great for creating simple, inline generators, but they can easily get far too compact and unreadable.

```python
f = (index for index, str_char in enumerate(string) if str_char == character)

print(list(f))
# [1, 2, 4, 7, 8]
```

This may seem incredibly similar to a list comprehension, that's because it could be written as the following one-liner:

```python
print([index for index, str_char in enumerate(string) if str_char == character])
```

The difference we'll go further into in when we [go further](#going-further)

#### Pros

- Super short

#### Cons

- Can quickly become very unreadable
- Slightly slower than [Generator Functions](#generator-function-yield) - [Source](https://stackoverflow.com/a/1995585)

## Going Further

At this point, you might be wondering why we keep converting `f` to a list. A hint can be found in our first, class-based version. There's nothing that clearly shows that it is subscriptable and that's because it's an iterator!

If you look at the documentation constructor for list it takes in an `iterable`.

> Iterators are iterables and generators are iterators, therefore generators are iterators

You could almost think of `list(f)` as the following

```python
def convert_list(iterator):
  i_list = []

  while True:
    try:
      i_list.append(next(iterator))
    except StopIteration:
      break

  return i_list
```

### Classes

In the case of our **class method** calling `next(f)` is equivalent to calling `f.__next__()`. So keeps calling `f.__next__()` until it `StopIteration` is raised.

Both our generator functions and expressions create objects with the next

### Generator Functions/Expressions

A function with a yield statement is a generator function that returns a generator object.

> Generator functions allow you to declare a function that behaves like an iterator, i.e. it can be used in a for a loop. - [Python Documentation](https://wiki.python.org/moin/Generators)

Generator expressions create generator objects as well.

This means that they implement `__iter__` and `__next__` in a similar fashion to our class. The difference is that the function tracks the past omitting the need for an `__init__`, thus allowing it to "resume" execution.

On the first call of `next` on a generator, the function starts execution and "returns" the value after yield and "pauses" execution. The next time `__next__` is called it "resumes" execution until the next `yield` statement.

It repeats this cycle until it reaches the end where it raises `StopIteration` just like the class.

> This allows us to create generators with a much simpler syntax

Additionally, you can use `yield from` to direct future calls of `next` to the result of another iterator. It even has the potential of creating [*recursive generators*](https://stackoverflow.com/a/8991864)

### Conclusion

When creating functions that return iterables consider making them into iterators or generators to save memory and potentially even speed up. your code by reducing memory usage.

I'm actively looking for internships! Check out my [LinkedIn](https://www.linkedin.com/in/dacioromero/) or [GitLab](https://gitlab.com/dacio) and get in contact with me if you're interested!

## Resources and Sources

[Generator Expression vs Function](https://stackoverflow.com/a/1995585)

[Generator Documentation](https://wiki.python.org/moin/Generator)

[Recrusive Generators](https://stackoverflow.com/a/8991864)

[More on Iterators](https://www.programiz.com/python-programming/iterator)

[Iteratable vs Iterator](https://www.geeksforgeeks.org/python-difference-iterable-iterator/)
