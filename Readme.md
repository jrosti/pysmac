pysmac
======

Simple python wrapper to [SMAC](http://www.cs.ubc.ca/labs/beta/Projects/SMAC/), a versatile tool for optimizing algorithm parameters.

```
 fmin(objective, x0, xmin, xmax, x0_int, xmin_int, xmax_int, xcategorical, params)
    min_x f(x) s.t. xmin < x < xmax
    
  objective: The objective function that should be optimized.
```

Requirements
------------

SMAC (and therefore also pysmac) requires [Java 7](https://jdk7.java.net/download.html).

Installation
------------

### Pip


```
pip install pysmac
```


### Manually

```
python setup.py install
```
 
Example usage
-------------

Let's take for example the Branin function. (Note that the branin function is not the ideal use case for SMAC, which is designed to be a global optimization tool for costly functions. That said, it'll serve the purpose of checking that everything is working.)
```python
import numpy as np

def branin(x):
    b = (5.1 / (4.*np.pi**2))
    c = (5. / np.pi)
    t = (1. / (8.*np.pi))
    return 1.*(x[1]-b*x[0]**2+c*x[0]-6.)**2+10.*(1-t)*np.cos(x[0])+10.
```
For x1 ∈ [-5, 10], x2 ∈ [0, 15] the function reaches a minimum value of: *0.397887*.

Note: fmin accepts any function that has a parameter called `x` (the input array) and returns an objective value.

```python
from pysmac.optimize import fmin

xmin, fval = fmin(branin, x0=(0,0),xmin=(-5, 0), xmax=(10, 15), max_evaluations=5000)
```
As soon as the evaluations are finished, we can check the output:
```python
>>> xmin
{'x': array([ 3.14305644,  2.27827543])}

>>> fval
0.397917
```

Let's run the objective function with the found parameters:
```python
>>> branin(**xmin)
0.397917
```

License
-------

SMAC is free for academic & non-commercial usage. Please contact [Frank Hutter](mailto:fh@informatik.uni-freiburg.de) to discuss obtaining a license for commercial purposes.

Advanced
--------

### Custom arguments to the objective function:

Note: make sure there is no naming collission with the parameter names and the custom arguments.

```python
def minfunc(x, custom_arg1, custom_arg2):
    print "custom_arg1:", custom_arg1
    print "custom_arg2:", custom_arg2
    return 1


xmin, fval = fmin(minfunc, x0=(0,0),xmin=(-5, 0), xmax=(10, 15),
                  max_evaluations=5000,
                  custom_args={"custom_arg1": "test",
                               "custom_arg2": 123})
```


### Cross-validation

SMAC can run CV-folds intelligently, that is if a parameter configuration does not perform well a subset of the CV folds it can choose to evaluate other configurations rather than first running all the other folds, which will save time. In order to use this feature, just specify the `cv_folds` parameter as well add an `cv_fold` parameter to the objective function:

```python

def minfunc(x, cv_fold):
    #...
    return somevalue

xmin, fval = fmin(minfunc, x0=(0,0),xmin=(-5, 0), xmax=(10, 15), cv_folds=10, max_evaluations=5000)
```

### Integer parameters
Integer parameters can be encoded as follows:
```python

def minfunc(x, x_int):
    print "x: ", x
    print "x_int: ", x_int
    return 1.

xmin, fval = fmin(minfunc,
                  x0=(0,0), xmin=(-5, 0), xmax=(10, 15),
                  x0_int=(0,0), xmin_int=(-5, 0), xmax_int=(10, 15),
                  max_evaluations=5000)
```


### Categorical parameters

Categorical parameters can be specified as a dictionary of lists of values they can take on, e.g.:
```python
categorical_params = {"param1": [1,2,3,4,5,6,7],
                      "param2": ["string1", "string2", "string3"]}

def minfunc(x_categorical):
    print "param1: ", x_categorical["param1"]
    print "param2: ", x_categorical["param2"]
    return 1.

xmin, fval = fmin(minfunc,
                  x_categorical=categorical_params,
                  max_evaluations=5000)
```

#### Example

Let's for example setup 20 categorical parameters that can either take 1 or 0 as well as the objective function being the number of parameters minus the sum of all the parameter values. This objective function will be minimized if all parameters are set to 1.

```python

ndim = 10
categorical_params = {}
for i in range(ndim):
    categorical_params["%d" % i] = [0, 1]

def sum_binary_params(x_categorical):
    return len(x_categorical.values()) - sum(x_categorical.values())
```

Now we can go ahead and let SMAC minimize the objective function:

```python
xmin, fval = fmin(minfunc,
                  x_categorical=categorical_params,
                  max_evaluations=500)
```
Let's look at the result:
```python
xmin = {'x_categorical': {'0': 1,
  '1': 1,
  '2': 1,
  '3': 1,
  '4': 1,
  '5': 1,
  '6': 1,
  '7': 1,
  '8': 1,
  '9': 1}}
```
