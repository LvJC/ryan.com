+++
title = "How to Deal With Errors in Python"
date = "2022-10-24T15:00:00+00:00"
tags = ["post"]
draft = false
author = "Ryan"
+++

Recently when I reviewed codes from teammates, I found that we had different preference on how to deal with errors and expectations in Python. After investigation and reading, I summarize two aspects: one is how to pass errors and the others is preventing or hanlidng them.

# 3 Strategies to Pass Error Messages

There are 3 common strategies to pass error messages to the caller of a function, as a conclusion.

1. Return values. You can return 0 when the caller gets a normal results, and return non-zero values when a failure occurs. It's widely used in many system APIs like Windows and Linux.
2. Global variables.
3. Exceptions. From my point of view, this is the best choice for Python since many pre-defined expections are in Python. sing exceptions for flow control is common and normal.

|                  | Advantages                                                                    | Disadvantages                                                                         |
| ---------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| Return Values    | Consistent with system APIs                                                   | Inconvenient to use results                                                           |
| Global Variables | Convenient to use results                                                     | Possible for callers to forget to check variable                                      |
| Exceptions       | Clear code structure isolating blocks for normal execution and error handling | Some languages don't support exceptions, and exceptions impact performance negatively |

## Where to Use Exceptions Catch

Generally you use try/except when you handle things that are outside of the parameters that you can influence. Exceptions mostly happen in concrete types, and they are pre-defined well in Python's document. (See in [Built-in Exceptions](https://docs.python.org/3/library/exceptions.html#bltin-exceptions))

With so many built-in exceptions and usage scenarios, it's recommended that, **try not to handle an arbitrary exception like as best you can like below.** If you do so, all other more specific exceptions have to be caught as well. It's always important for developers to anticipate what kind of exceptions will occur in a code snippet and to give a way to handle them.

```python
try:
    #your code
except Exception as ex:
    print(ex)
```

Therefore, a more appropriate way of using exception handling is like this:

One exception

```python
try:
    file = open('input-file', 'open mode')
except EOFError as ex:
    print("Caught the EOF error.")
    raise ex
```

Mutiple exceptions

```python
try:
    file = open('input-file', 'open mode')
except (IOError, EOFError) as e:
    log.error("Testing multiple exceptions. {}".format(e.args[-1]))
```

Re-raising after handling

```python
try:
    # Intentionally raise an exception.
    raise RuntimeError('I learn Python!')
except RuntimeError as e:
    print("Entered in except."). # handling
    # Re-raise the exception.
    raise e
```

# Preventing Errors or Handling?

We can have 2 coding styles to deal with exceptions in our codes: **look before you leap** (LBYL), and **easier to ask forgiveness than permission** (EAFP).

1. **LBYL: Prevent** errors or exceptional situations from happening
2. **EAFP: Handle** errors or exceptional situations after they happen

Here is a small case to understand their difference better. Say that you have a dictionary and want to access the value of one key, but you don't know if the key is present.

```python
# LBYL
if "possible_key" in data_dict:
    value = data_dict["possible_key"]
else:
    # Handle missing keys here...
```

```python
# EAFP
try:
     value = data_dict["possible_key"]
except KeyError:
    # Handle missing keys here...
```

We can even find there are brief introductions of LBYL and EAFP styles in Python's official glossary of terms.

> **LBYL**
>
> Look before you leap. This coding style explicitly tests for pre-conditions before making calls or lookups. This style contrasts with the [EAFP](https://docs.python.org/3/glossary.html#term-EAFP) approach and is characterized by the presence of many `if` statements.
>
> In a multi-threaded environment, the LBYL approach can risk introducing a race condition between “the looking” and “the leaping”. For example, the code,` if key in mapping: return mapping[key]` can fail if another thread removes *key* from *mapping* after the test, but before the lookup. This issue can be solved with locks or by using the EAFP approach.

> **EAFP**
>
> Easier to ask for forgiveness than permission. This common Python coding style assumes the existence of valid keys or attributes and catches exceptions if the assumption proves false. This clean and fast style is characterized by the presence of many `try` and `except` statements. The technique contrasts with the [LBYL](https://docs.python.org/3/glossary.html#term-LBYL) style common to many other languages such as C.

## Which is better, LBYL or EAFP

Although it shows that EAFP is generally recommended by Python's official document, there still remains controversial. Even Guido van Rossum, the creator of Python disagreed EAFP is better than LBYL. Hence, it depends and you can compare both styles using a few relevant comparison criteria.

| Criteria                | LBYL                                                                                                                       | EAFP                                                                                                                                        |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| Number of checks        | Repeats checks typically provided by Python                                                                                | Runs the checks provided by Python only once                                                                                                |
| Readability and clarity | Has poor readability and clarity because exceptional situations seem to be more important than the target operation itself | Has enhanced readability because the target operation is front and center, while the exceptional situations are relegated to the background |
| Race condition risk     | Implies a risk of race conditions between the check and the target operation                                               | Prevents the risk of race conditions because the operation runs without doing any checks                                                    |
| Code performance        | Has poor performance when the checks almost always succeed, and better performance when the checks almost always fail      | Has better performance when the checks almost always succeed, and poor performance when the checks almost always fail                       |

Take the divide function as an example,

```python
>>> def divide(a, b, default=None):
...     if b == 0:  # Exceptional situation
...         print("zero division detected")  # Error handling
...         return default
...     return a / b  # Most common situation
...

>>> divide(8, 2)
4.0

>>> divide(8, 0)
zero division detected

>>> divide(8, 0, default=0)
zero division detected
0
```

The readability is obviously supposed to be improved by chaning it to EAFP style with `try` block. And this function is about computing the division of two numbers rather than about making sure that the denominator is not `0`.

```python
>>> def divide(a, b, default=None):
...     try:
...         return a / b  # Most common situation
...     except ZeroDivisionError:  # Exceptional situation
...         print("zero division detected")  # Error handling
...         return default
...

>>> divide(8, 2)
4.0

>>> divide(8, 0)
zero division detected

>>> divide(8, 0, default=0)
zero division detected
0
```

The other comparisons can be found in https://realpython.com/python-lbyl-vs-eafp/#the-lbyl-and-eafp-coding-styles-in-python

## When to Use LBYL or EAFP

Here's a summary to know when use LBYL or EAFP.

| Use LBYL for                                                           | Use EAFP for                                                               |
| ---------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| Operations that are likely to fail                                     | Operations that are unlikely to fail                                       |
| Irrevocable operations, and operations that may have a side effect     | Input and output (IO) operations, mainly hard drive and network operations |
| Common exceptional conditions that can be quickly prevented beforehand | Database operations that can be rolled back quickly                        |

# Reference

1. Errors and Exceptions, https://docs.python.org/3/tutorial/errors.html
2. LBYL vs EAFP: Preventing or Handling Errors in Python, https://realpython.com/python-lbyl-vs-eafp/
3. High Quality Code, *Coding Interviews: Questions, Analysis and Solutions*
4. How to Best Use Try Except in Python – Especially for Beginners, https://www.techbeamers.com/use-try-except-python/
