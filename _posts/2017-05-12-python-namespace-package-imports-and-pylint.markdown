# Python Namespace Package Imports and Pylint
I recently had an interesting (and frustrating) experience while linting a Python 3.6 project with pylint 1.6.5.

## The Problem
Despite a functioning Python project, pylint would consistently fail to import namespace packages with the error `E401: Unable to import [package]`

**TL;DR**: [solution](#the-solution)

Described below is a simple project that demonstrates the issue.

Project structure:

```
myproject/
└── mypackage
    ├── badger.py
    ├── llama.py
    ├── .pylintrc
    ├── mysubpackage
    │   └── snake.py
```
Python file contents:
```
$ cat llama.py
#!/usr/bin/env python3

import sys
from sys import path
from os.path import dirname, abspath, join

path.append(abspath(join(dirname(__file__), '..')))

from mypackage import badger
from mypackage.mysubpackage import snake

if __name__ == '__main__':
    badger.whoami()
    snake.whoami()

$ cat badger.py
def whoami():
    print("I am badger!")

$ cat mysubpackage/snake.py
def whoami():
    print("I am snake!")
```

This all runs as expected:

```
$ ./llama.py
['/Users/rick/code/myproject/mypackage', '/usr/local/Cellar/python3/3.6.0/Frameworks/Python.framework/Versions/3.6/lib/python36.zip', '/usr/local/Cellar/python3/3.6.0/Frameworks/Python.framework/Versions/3.6/lib/python3.6', '/usr/local/Cellar/python3/3.6.0/Frameworks/Python.framework/Versions/3.6/lib/python3.6/lib-dynload', '/Users/rick/Library/Python/3.6/lib/python/site-packages', '/usr/local/lib/python3.6/site-packages', '/Users/rick/code/myproject']
I am badger!
I am snake!
```

However, pylint would still complain (*Unable to import ${package}*):

```
$ pylint --reports=no llama.py
************* Module llama
E: 10, 0: Unable to import 'mypackage' (import-error)
E: 11, 0: Unable to import 'mypackage.mysubpackage' (import-error)
```

I thought maybe it was a path issue in pylint’s environment, so I appended to the path in my `.pylintrc` too:
```
$ cat .pylintrc
[MASTER]
init-hook='import sys; import os; print(">>>INIT HOOK<<<"); sys.path.append(os.path.abspath(os.path.join(os.getcwd(), '..'))); print(sys.path); from mypackage import badger;badger.whoami()'
```

But that didn’t change the behavior:

```
$ pylint --reports=no llama.py
>>>INIT HOOK<<<
['/usr/local/bin', '/usr/local/Cellar/python3/3.6.0/Frameworks/Python.framework/Versions/3.6/lib/python36.zip', '/usr/local/Cellar/python3/3.6.0/Frameworks/Python.framework/Versions/3.6/lib/python3.6', '/usr/local/Cellar/python3/3.6.0/Frameworks/Python.framework/Versions/3.6/lib/python3.6/lib-dynload', '/Users/rick/Library/Python/3.6/lib/python/site-packages', '/usr/local/lib/python3.6/site-packages', '/usr/local/lib/python3.6/site-packages/astroid/brain', '/Users/rick/code/myproject']
I am badger!
************* Module llama
E: 10, 0: Unable to import 'mypackage' (import-error)
E: 11, 0: Unable to import 'mypackage.mysubpackage' (import-error)
```

## The Solution
After some discussion with colleagues, it turned out that while Python >= 3.3 no longer requires `__init__.py` files ([PEP-420](https://www.python.org/dev/peps/pep-0420/)), pylint still does (discussion [here](https://github.com/PyCQA/pylint/issues/842)).

The fix was to add `__init__.py` files to each package directory.
