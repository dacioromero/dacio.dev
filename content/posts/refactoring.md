---
title: "Getting to Refactoring"
slug: "refactoring"
date: 2019-05-07T14:07:22-07:00
---

<center>
  <img src="https://source.unsplash.com/vGgn0xLdy8s" alt="Hero Image">
  Photo by Annie Spratt on Unsplash
</center>

Every developer at some point will revisit their old project or snippet of code and be disgusted with the work they had. It is natural though: the name of the game as a developer is to *continually learn and improve*, so looking at old code is almost always when you didn't know as much as you do now.

At first, you might want to avoid messing with old code, but you're missing out an opportunity to learn, improve the maintainability of the code, make it more performant, or even make it a better portfolio item if it's something you can show publicly.

> Knowing that you should improve is the first step to *refactoring* your code.

# What is it?

Refactoring, in a nutshell, is making changes that **don't** change the functionality. A real life example would be that we don't care exactly how our local restaurant makes their food, so long as every time, we get the same food if we ask for the same item regardless of whatever changes the restaurant makes.

> If the restaurant hires new staff or gets our food to us faster to doesn't affect how we eat it.

# Examples

## Syntactic

Following up on [my previous post on iterators](/2019/05/03/python-iterators-and-generators/) let's see how I used my knowledge of iterators to refactor some code in the past.

### Code

```python
def find_index(text, pattern):
  if pattern == '':
    return 0

  for i in range(len(text) - len(pattern) + 1):
    if text[i:i+len(pattern)] == pattern:
      return i

  return None


def find_all_indexes(text, pattern):
  if pattern == '':
    return list(range(len(text)))

  indices = []

  for i in range(len(text) - len(pattern) + 1):
    if text[i:i+len(pattern)] == pattern:
      indices.append(i)

  return indices
```

[Commit](https://gitlab.com/dacio/cs-1.3/blob/42d317279d0e6e392785eb0fa59404a1a9a8a813/Lessons/source/strings.py)

Now there should be a couple of red flags going off in your head right about now.

Lines 5-7 and 18-20 are incredibly similar. This code isn't as *maintainable* because if anyone wants to change the search logic now there are **two points** to change.

At the time of writing the first example that I didn't fully understand how iterators and how they could improve this code. If you don't know what an iterator is it *lazily* produces values allowing you to perform actions on them one at a time. So let's make an iterator with a generator function that combines the logic of both of them.

```python
def _find_indexes(text, pattern):
  if pattern == '':
    yield from range(len(text))
    return

  for i in range(len(text) - len(pattern) + 1):
    if text[i:i+len(pattern)] == pattern:
      yield i
```

This generator function that returns an iterator which produces the starting indexes of matching patterns in our text. Now all we can change our old functions to be just:

```python
def find_index(text, pattern):
  return next(_find_indexes(text, pattern), None)

def find_all_indexes(text, pattern):
  return list(_find_indexes(text, pattern))
```

[Commit](https://gitlab.com/dacio/cs-1.3/blob/ff0cd765a3dec0973e60f682b4bd49153658f669/Lessons/source/strings.py)

### Result

We've reduced the complexity and improved the DRYness of our code. This helps *maintainability and readability*. If we want to improve the performance or any other aspect it'd be much easier now. So let's do that.

## Performance

In both previous snippets, I used slicing to create substrings to compare to the pattern, but it's not a very performant operation. Checking if "helps" and "hello" were equal it's unnecessary to create an entirely new string for each potential spot when in this example we'd know they aren't equal at the fourth character.

So let's see some code that uses another iterator to check each character individually.

```python
def _find_indexes(text, pattern):
  if pattern == '':
    yield from range(len(text))
    return

  # Find matching indices
  for index in range(len(text) - len(pattern) + 1):
    char_gen = (text[i] for i in range(index, index + len(pattern)))

    for char, pat_char in zip(char_gen, pattern):
      if char != pat_char:
        break
    else:
      yield index
```

[Commit](https://gitlab.com/dacio/cs-1.3/commit/ff0cd765a3dec0973e60f682b4bd49153658f669)

By creating `char_gen` and using it to check each character individually we've reduced the number of operations done per substring we're checking. This increases the performance of our generator as subsequently both `find_all_indexes` and `find_index`!

# Conclusion

There are *many* other ways that you can refactor your code, but hopefully, this sets you down the right path to starting. What I've shown are some pretty trivial examples, but the same pattern can be applied to most types of code.

> Look for places that have similar logic, are underperforming, or just simply are hard to understand and **refactor** it.
