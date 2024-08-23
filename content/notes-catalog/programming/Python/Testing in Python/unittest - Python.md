---
id: 01J5XP84H3YYPTC1DMA16E71RS
modified: 2024-08-22T14:47:12-04:00
title: unittest - Python
description: How to use unittest Python
tags:
  - python
  - unit-test
  - unittest
  - tests
  - programming
---
# unittest - Python
- ## Basic Example
```python
import unittest

class TestStringMethods(unittest.TestCase):

    def test_upper(self):
        self.assertEqual('foo'.upper(), 'FOO')

    def test_isupper(self):
        self.assertTrue('FOO'.isupper())
        self.assertFalse('Foo'.isupper())

    def test_split(self):
        s = 'hello world'
        self.assertEqual(s.split(), ['hello', 'world'])
        # check that s.split fails when the separator is not a string
        with self.assertRaises(TypeError):
            s.split(2)

if __name__ == '__main__':
    unittest.main()
```
A testcase is created by subclassing [`unittest.TestCase`](https://docs.python.org/3/library/unittest.html#unittest.TestCase "unittest.TestCase"). The three individual tests are defined with methods whose names start with the letters `test`. This naming convention informs the test runner about which methods represent tests.

The crux of each test is a call to [`assertEqual()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertEqual "unittest.TestCase.assertEqual") to check for an expected result; [`assertTrue()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertTrue "unittest.TestCase.assertTrue") or [`assertFalse()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertFalse "unittest.TestCase.assertFalse") to verify a condition; or [`assertRaises()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertRaises "unittest.TestCase.assertRaises") to verify that a specific exception gets raised. These methods are used instead of the [`assert`](https://docs.python.org/3/reference/simple_stmts.html#assert) statement so the test runner can accumulate all test results and produce a report.

The [`setUp()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.setUp "unittest.TestCase.setUp") and [`tearDown()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.tearDown "unittest.TestCase.tearDown") methods allow you to define instructions that will be executed before and after each test method. They are covered in more detail in the section [Organizing test code](https://docs.python.org/3/library/unittest.html#organizing-tests).

The final block shows a simple way to run the tests. [`unittest.main()`](https://docs.python.org/3/library/unittest.html#unittest.main "unittest.main") provides a command-line interface to the test script. When run from the command line, the above script produces an output that looks like this:

looks like this:
```python
...
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
```

Passing the `-v` option to your test script will instruct [`unittest.main()`](https://docs.python.org/3/library/unittest.html#unittest.main "unittest.main") to enable a higher level of verbosity, and produce the following output:

```python
test_isupper (__main__.TestStringMethods.test_isupper) ... ok
test_split (__main__.TestStringMethods.test_split) ... ok
test_upper (__main__.TestStringMethods.test_upper) ... ok

----------------------------------------------------------------------
Ran 3 tests in 0.001s

OK
```

The above examples show the most commonly used [`unittest`](https://docs.python.org/3/library/unittest.html#module-unittest "unittest: Unit testing framework for Python.") features which are sufficient to meet many everyday testing needs. The remainder of the documentation explores the full feature set from first principles.