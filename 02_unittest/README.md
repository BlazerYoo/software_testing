# Unit Testing

Unit testing involves writing tests for standalone units of code such as a function. You could write your own testing framework maybe
using the `assert` statement but it is better to use a framework that already exists.

## unittest from the Python Standard Library

A good starting point for unit testing is the [`unittest`](https://docs.python.org/3/library/unittest.html) module of the Python Standard Library. If you have Python installed then you have this module:

```bash
$ module load anaconda3/2021.5
$ python
>>> import unittest
```

The idea to unit testing is to write test code in a separate file for a given source code unit. If the tests run successfully then you will have more confidence in the source code.

## Example

Consider a function to compute the area of a circle:

```python
import math

def circle_area(radius):
  return math.pi * radius**2
```

Let's test this script against different input values for the radius (`software_testing/02_unittest/circle_area_values.py`):

```python
import math

def circle_area(radius):
  return math.pi * radius**2

radius_values = [2, 0, -3, 2 + 5j, True, "cat"]
for radius in radius_values:
  area = circle_area(radius)
  print(f"Area of circle with radius = {radius} is {area}")
```

Run the script above with these commands:

```
$ cd software_testing/02_unittest
$ python circle_area_values.py
Area of circle with radius = 2 is 12.566370614359172
Area of circle with radius = 0 is 0.0
Area of circle with radius = -3 is 28.274333882308138
Area of circle with radius = (2+5j) is (-65.97344572538566+62.83185307179586j)
Area of circle with radius = True is 3.141592653589793
Traceback (most recent call last):
  File "circle_area_values.py", line 8, in <module>
    area = circle_area(radius)
  File "circle_area_values.py", line 4, in circle_area
    return math.pi * radius**2
TypeError: unsupported operand type(s) for ** or pow(): 'str' and 'int'
```

The function produces nonsensical output for three of the inputs and crashes when encountering a string. Note that if a user accidentally used `True` as an input, the code would not fail. Can we improve on our function to make it more robust? Let's also write a series of tests to make sure the mistakes highlighted in the example above are caught. For more see this Socratica [video](https://www.youtube.com/watch?v=1Lfv5tUGsn8).

## First unit test

Consider the code below (see `circle_area.py`):

```python
import math

def circle_area(radius):
    return math.pi * radius**2
  
if __name__ == "__main__":
    print("Success")
```

Let's look at a unit test for the `circle_area` function. The test is stored in a separate file (see `test_circle_area.py`). It is conventional to prepend `test_` to the name of the original source file. Here are the contents of `test_circle_area.py`:

```python
import unittest
from circle_area import circle_area
import math

class TestCircleArea(unittest.TestCase):
    def test_area(self):
        # test areas when radius >= 0
        self.assertAlmostEqual(circle_area(1), math.pi)
        self.assertAlmostEqual(circle_area(0), 0)
        self.assertAlmostEqual(circle_area(2.1), math.pi * 2.1**2)
```

One can see that we wrote a class that derives from `unittest.TestCase`. We then write a class method for a unit test composed of three assert methods to check cases where the radius is greater than or equal to zero. Here are the [most popular](https://docs.python.org/3/library/unittest.html#unittest.TestCase.debug) assert methods.

The name of the test within the class must begin with `test_`. Unit tests will be ignored if they don't follow that convention, which is useful if you need a helper method for one of the test functions.

Note that this is an example of *white box testing* where we can inspect the code that we are writing the tests for. *Black box testing* is when tests are written when the code is not available. [Test-driven development](https://en.wikipedia.org/wiki/Test-driven_development) is an example of black box testing since the tests are written before the code exists.

We now have two files:

```
circle_area.py
test_circle_area.py
```

To run the test:

```
$ python -m unittest test_circle_area.py
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
```

The `.` in the first line of output implies success of one test. While there are three assert methods, they are grouped in a single test.

Add the `-v` flag for verbose output:

```
$ python -m unittest test_circle_area.py -v
```

Return to the source code (`circle_area.py`) and handle the case of a negative area:

```python
def circle_area(radius):
    if radius < 0:
        raise ValueError("The radius cannot be negative.")   
    return math.pi * radius**2
```

See a list of [built-in Python exceptions](https://docs.python.org/3/library/exceptions.html) which includes `ValueError`.

Let's add an additional test to handle radius values less than zero:

```python
import unittest
from circle_area import circle_area
import math

class TestCircleArea(unittest.TestCase):
    def test_area(self):
        # test areas when radius >= 0
        self.assertAlmostEqual(circle_area(1), math.pi)
        self.assertAlmostEqual(circle_area(0), 0)
        self.assertAlmostEqual(circle_area(2.1), math.pi * 2.1**2)
        
    def test_values(self):
        # raise value error when radius is negative
        self.assertRaises(ValueError, circle_area, -2)
```

You could also write the new test as:

```python
    def test_values(self):
        # raise value error when radius is negative
        with self.assertRaises(ValueError):
            circle_area(-2)
```

Once again run the tests:

```
$ python -m unittest test_circle_area.py -v
test_area (test_circle_area.TestCircleArea) ... ok
test_values (test_circle_area.TestCircleArea) ... ok

----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
```

Lastly, let's handle type errors such as complex numbers and booleans:

```python
import unittest
from circle_area import circle_area
import math

class TestCircleArea(unittest.TestCase):

    def test_area(self):
        # test areas when radius >= 0
        self.assertAlmostEqual(circle_area(1), math.pi)
        self.assertAlmostEqual(circle_area(0), 0)
        self.assertAlmostEqual(circle_area(2.1), math.pi * 2.1**2)
        
    def test_values(self):
        # raise value error when radius is negative
        self.assertRaises(ValueError, circle_area, -2)
        
    def test_types(self):
        # handle type errors
        self.assertRaises(TypeError, circle_area, 3+5j)
        self.assertRaises(TypeError, circle_area, True)
        self.assertRaises(TypeError, circle_area, "cat")
```

Run the unit tests (without modifying the source code as an exercise):

```
$ python -m unittest test_circle_area.py -v
test_area (test_circle_area.TestCircleArea) ... ok
test_types (test_circle_area.TestCircleArea) ... FAIL
test_values (test_circle_area.TestCircleArea) ... ok

======================================================================
FAIL: test_types (test_circle_area.TestCircleArea)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/scratch/gpfs/jdh4/TESTING/test_circle_area.py", line 19, in test_types
    self.assertRaises(TypeError, circle_area, True)
AssertionError: TypeError not raised by circle_area

----------------------------------------------------------------------
Ran 3 tests in 0.000s

FAILED (failures=1)
```

We see that there is a failure. Let's modify our source code so that the tests succeed:

```python
def circle_area(radius):
    if not type(radius) in (int, float):
        raise TypeError("The radius is not an int or float.")
    if radius < 0:
        raise ValueError("The radius cannot be negative.")
    return math.pi * radius**2
```

Now run the unit tests:

```
$ python -m unittest test_circle_area.py -v
test_area (test_circle_area.TestCircleArea) ... ok
test_types (test_circle_area.TestCircleArea) ... ok
test_values (test_circle_area.TestCircleArea) ... ok

----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
```

We see all the tests pass. Our source code has been much improved.

## Exercise 1

Create a file called `bmi.py` that provides a function to compute the body mass index which is `bmi = mass / height**2`. Then create a second file called `test_bmi.py` for the unit tests. In addition to previous tests, be sure to include a check for division by zero (`ZeroDivisionError`) and a unit test where `mass` or `height` is `None`.

If you finish early then start writing unit tests for your own research software or look at the [`unittest`](https://docs.python.org/3/library/unittest.html) documentation.

## setUp() and tearDown()

This section will illustrate the idea of `setUp()` and `tearDown()` operations. Consider the following class (`shapes.py`):

```python
import math

class Circle:

    def __init__(self, radius, color):
        self.radius = radius
        self.color = color

    def compute_area(self):
        # tests removed for brevity
        return math.pi * self.radius**2

    def change_color(self, new_color):
        self.color = new_color
```

You might naively begin with unit tests like the following:

```python
import unittest
from shapes import Circle
import math

class TestCircle(unittest.TestCase):

    def test_colors(self):
        c1 = Circle(3, "red")
        c2 = Circle(5, "green")

        self.assertEqual(c1.color, "red")
        self.assertEqual(c2.color, "green")

        c1.change_color("blue")
        c2.change_color("blue")
        self.assertEqual(c1.color, "blue")
        self.assertEqual(c2.color, "blue")

    def test_area(self):
        c1 = Circle(3, "red")
        c2 = Circle(5, "green")

        self.assertAlmostEqual(c1.compute_area(), math.pi * 3**2)
        self.assertAlmostEqual(c2.compute_area(), math.pi * 5**2)
```

The above set of tests is reasonable but we can do better by recognizing that both tests create the same Circle objects. If the initialization method changes (maybe by adding a third required parameter) then changes must be made to both tests. The Python unittest module provides  `setUp()` and `tearDown()` for dealing with this.

```python
import unittest
from shapes import Circle
import math

class TestCircle(unittest.TestCase):

    def setUp(self):
        # do setup operations here for each test
        self.c1 = Circle(3, "red")
        self.c2 = Circle(5, "green")
        
    def test_colors(self):
        self.assertEqual(self.c1.color, "red")
        self.assertEqual(self.c2.color, "green")

        self.c1.change_color("blue")
        self.c2.change_color("blue")
        self.assertEqual(self.c1.color, "blue")
        self.assertEqual(self.c2.color, "blue")

    def test_area(self):
        self.assertAlmostEqual(self.c1.compute_area(), math.pi * 3**2)
        self.assertAlmostEqual(self.c2.compute_area(), math.pi * 5**2)

    def tearDown(self):
        # do teardown operations here for each test
        pass
```

Now run the tests:

```
$ cd 02_unittest/misc
$ python -m unittest test_shapes.py -v
test_area (test_shapes.TestCase1) ... ok
test_colors (test_shapes.TestCase1) ... ok

----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
```

Let's add print statements to see the order that the various operations are ran:

```python
import unittest
from shapes import Circle
import math

class TestCircle(unittest.TestCase):

    def setUp(self):
        # do setup operations here for each test
        print("setUp")
        self.c1 = Circle(3, "red")
        self.c2 = Circle(5, "green")
      
    def test_colors(self):
        print("test_colors")
        self.assertEqual(self.c1.color, "red")
        self.assertEqual(self.c2.color, "green")

        self.c1.change_color("blue")
        self.c2.change_color("blue")
        self.assertEqual(self.c1.color, "blue")
        self.assertEqual(self.c2.color, "blue")

    def test_area(self):
        print("test_area")
        self.assertAlmostEqual(self.c1.compute_area(), math.pi * 3**2)
        self.assertAlmostEqual(self.c2.compute_area(), math.pi * 5**2)

    def tearDown(self):
        # do teardown operations here for each test
        print("tearDown")
```

Here is the output:

```
$ python -m unittest test_shapes_print.py
setUp
test_area
tearDown
.setUp
test_colors
tearDown
.
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
```

## setUpClass() and tearDownClass()

We saw above that `setUp()` and `tearDown()` run for each test. In some cases you will want to perform some operations once before all the tests and once after all the tests. For this the `setUpClass()` and `tearDownClass()` methods can be used:

```python
import unittest
from shapes import Circle
import math

class TestCircle(unittest.TestCase):

    @classmethod
    def setUpClass(cls):
        # do setup operations here for all tests once
        print("setUpClass")
    
    def setUp(self):
        # do setup operations here for each test
        self.c1 = Circle(3, "red")
        self.c2 = Circle(5, "green")
        
    def test_colors(self):
        self.assertEqual(self.c1.color, "red")
        self.assertEqual(self.c2.color, "green")

        self.c1.change_color("blue")
        self.c2.change_color("blue")
        self.assertEqual(self.c1.color, "blue")
        self.assertEqual(self.c2.color, "blue")

    def test_area(self):
        self.assertAlmostEqual(self.c1.compute_area(), math.pi * 3**2)
        self.assertAlmostEqual(self.c2.compute_area(), math.pi * 5**2)

    def tearDown(self):
        # do teardown operations here for each test
        pass
        
   @classmethod
    def tearDownClass(cls):
        # do teardown operations here for all tests once
        print("teardownClass")
```

By running the tests we see the order in which the various functions are executed:

```
$ python -m unittest test_shapes_print_class.py
setUpClass
setUp
test_area
tearDown

.setUp
test_colors
tearDown

.teardownClass

----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
```

In the above nothing was done in `setUpClass()` or `tearDownClass()`. An example of using `setUpClass()` would be connecting to a database or generating a large file. For more details on the material above see the [video](https://www.youtube.com/watch?v=6tNS--WetLI) by Corey Schafer.

## Exercise 2

Develop the `setUp()`, `tearDown()`, `setUpClass()` and `tearDownClass()` methods for `misc/health.py` and `misc/test_health.py`. Or apply these methods to the tests of the scripts for your research work.

## Running multiple test files with TestSuite

Above we consider the source code `shapes.py` and the corresponding unit tests in `test_shapes.py`. What if your project is composed of multiple files such as `shapes.py` and `sizes.py`? How can you execute all the tests at once? The idea is to create a `TestSuite` and a `TestRunner` to run all the tests at once.

Here are the contents of `runner.py`:

```python
import unittest
import test_shapes
import test_sizes

loader = unittest.TestLoader()
suite = unittest.TestSuite()

suite.addTests(loader.loadTestsFromModule(test_shapes))
suite.addTests(loader.loadTestsFromModule(test_sizes))

runner = unittest.TextTestRunner(verbosity=3)
result = runner.run(suite)
```

Run all the tests with:

```
$ python runner.py
test_area (test_shapes.TestCircle) ... ok
test_colors (test_shapes.TestCircle) ... ok
test_thres (test_sizes.TestThresholds) ... ok

----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
```

## On writing tests

When writing unit tests it is common to feel like you are creating tests that will obviously work and therefore are not worth the effort. In this case, remind yourself that a lot of things can go wrong when writing software. For instance, stray characters can get entered into code accidentally (e.g., a literal sneeze) so continue writing these tests with this in mind. Also, some developers debug by modifying the code and forget to return it to its original state. In both of these cases, even simple unit tests are very likely to find the problem.
