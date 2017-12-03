---
layout: post
title: "Python装饰器"
date: 2017-12-01
---


> *长期寻找好的装饰器*

#### 1. Log

##### (1) 传统函数

```python
from functools import wraps
import time


def logit(filename='out.log'):
    def logging_decorator(func):
        @wraps(func)
        def wrapped_func(*args, **kwargs):
            t = time.strftime("%Y-%m-%d %H:%M:%S", time.gmtime())
            log_string = func.__name__ + " was called on " + t
            print(log_string)
            with open(filename, 'a') as f:
                f.write(log_string + '\n')
            return func(*args, **kwargs)
        return wrapped_func
    return logging_decorator

@logit()
def my_func1():
    pass

@logit('shit.log')
def my_func2():
    pass

my_func1()
my_func2()
```

##### (2) 使用类

```python
class Cls_logit(object):
    _filename = 'cls_logit.log'
    def __init__(self, func):
        self.func = func

    def __call__(self, *args):
        t = time.strftime("%Y-%m-%d %H:%M:%S", time.gmtime())
        log_string = self.func.__name__ + " called on " + t
        print(log_string)
        with open(self._filename, 'a') as f:
            f.write(log_string + '\n')
        self.notify()
        return self.func(*args)

    def notify(self):
        pass


Cls_logit._filename = 'cls_logit2.log'

@Cls_logit
def my_func3(arg1):
    print('in my_func3 with', arg1)

my_func3(3)
```


#### 2. Auth验证

```python
from functools import wraps

def requires_auth(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        auth = request.authorization
        if not auth or not check_auth(auth.username, auth.password):
            authenticate()
        return f(*args, **kwargs)
    return decorated

@requires_auth
def user_some_action():
    pass

```

#### 3. 递归内存优化

```python
from functools import wraps

def memoize(function):
    print('wizardry here')
    memo = {}
    @wraps(function)
    def wrapper(*args):
        if args in memo:
            return memo[args]
        else:
            rv = function(*args)
            memo[args] = rv
            return rv
    return wrapper

@memoize
def fibonacci(n):
    if n < 2: return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# fibonacci(25)
```

执行上面代码，虽然注释掉了`fibonacci(25)`(也就是没调用函数)，但还是会打印`wizardry here`这行代码，可见装饰器的魔法就在于直接在函数定义后立马替换原来函数。



*未完待续*

