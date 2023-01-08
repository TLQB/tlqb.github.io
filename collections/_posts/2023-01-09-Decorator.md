---
title: Decorator
author: tlqb
date: 2023-01-09
categories: [Python,Advance,Decorator]
tags: [python]
math: true
mermaid: true
---

## Function Decorator

Decorator là một mẫu thiết kế cấu trúc cho phép bạn gắn các hành vi mới vào các
đối tượng bằng cách đặt các đối tượng này bên trong các đối tượng bao bọc đặc
biệt có chứa các hành vi.


![](https://refactoring.guru/images/patterns/content/decorator/decorator-
comic-1.png)

Tôi sẽ bắt đầu với một ví dụ về hàm rất đơn giản như sau.

```python
def func(a):
  def f_nested(b):
    return b + b 
    
  return f_nested
```

Chúng ta có một hàm tên là f_nested với một tham số là a bên trong một hàm có
tên func, có tham số là b. Thực thi chương trình này bằng cách đưa lần lượt các
đối số với giá trị 10, 20.

```python
func(10)(20)
```

Đoạn code trên có thể viết ngắn gọn hơn với việc sử dụng hàm lambda.

```python
def func(a):
  return lambda b: b+b
```

```python
func(10)(20)
```

Hãy xem đoạn code sau:

```python
from types import FunctionType

def f_nested(b):
  print("Chạy vào hàm f_nested")
  return b + b 

def func(func_obj : FunctionType):
  print("chạy vào hàm func")
  return func_obj

func(f_nested(10))
```

Từ việc thực thi đoạn code bên trên chúng ta có thể đi đến kết luận decorator là
quá trình gọi một hàm trong một hàm với việc gắn một hàm làm tham số cho hàm còn
lại.

```python
from types import FunctionType

def func(func_obj : FunctionType):
  print("chạy vào hàm func")
  return func_obj

@func
def f_nested(b):
  print("Chạy vào hàm f_nested")
  return b + b 

f_nested(10)
```

Thử tạo decorator kiểm tra thời gian thực thi của một hàm bất kỳ với đoạn code
đơn giản như sau.

```python
from types import FunctionType
import time 

def check_time(func : FunctionType):
  def handle(*args):
    print("Bắt đầu chạy ...")
    start_time = time.time()
    result = func(*args)
    print("Hoàn thành trong {} giây".format(time.time() - start_time))

    return result 
  return handle

@check_time
def length(items):
  n = 0 
  while items is not None and items != items[1:]:
    n,items = n+1, items[1:]
  return n

length(range(10))
```

Mở rộng thêm

```python
from types import FunctionType
import time 

def check_time(prefix = ""):
  def check(func : FunctionType):
    print(f"{prefix}\n")
    def handle(*args):
      print("Bắt đầu chạy hàm {}".format(func.__name__))
      start_time = time.time()
      result = func(*args)
      print("Hoàn thành trong {} giây".format(time.time() - start_time))

      return result 
    return handle
  return check

@check_time(prefix = "***Check time***")
def length(items):
  """ 
  Hàm kiểm tra số lượng phần tử của items
  """
  n = 0 
  while items is not None and items != items[1:]:
    n,items = n+1, items[1:]
  return n

length(range(10))
```

Kiểm tra name func

```python
length.__name__
```

```python
length.__doc__
```

Giá trị name và docstring của hàm length trả về không chính xác. Để khắc phục
vấn đề này chúng ta có thể xử lý như sau.

```python
from types import FunctionType
import time 

def check_time(prefix = ""):
  def check(func : FunctionType):
    print(f"{prefix}\n")
    def handle(*args):
      print("Bắt đầu chạy hàm {}".format(func.__name__))
      start_time = time.time()
      result = func(*args)
      print("Hoàn thành trong {} giây".format(time.time() - start_time))
      handle.__name__ = func.__name__ 
      handle.__doc__ = func.__doc__ 
      return result 
    return handle
  return check

@check_time(prefix = "***Check time***")
def length(items):
  """ 
  Hàm kiểm tra số lượng phần tử của items
  """
  n = 0 
  while items is not None and items != items[1:]:
    n,items = n+1, items[1:]
  return n

length(range(10))
```

```python
length.__name__
```

```python
length.__doc__
```

Và tốt hơn nên dùng giải pháp có sẵn là decorator wraps được tích hợp trong
functools

```python
from functools import wraps 
from types import FunctionType
import time 

def check_time(prefix = ""):
  def check(func : FunctionType):
    print(f"{prefix}\n")
    @wraps(func)
    def handle(*args):
      print("Bắt đầu chạy hàm {}".format(func.__name__))
      start_time = time.time()
      result = func(*args)
      print("Hoàn thành trong {} giây".format(time.time() - start_time))
      # handle.__name__ = func.__name__ 
      # handle.__doc__ = func.__doc__ 
      return result 
    return handle
  return check

@check_time(prefix = "***Check time***")
def length(items):
  """ 
  Hàm kiểm tra số lượng phần tử của items
  """
  n = 0 
  while items is not None and items != items[1:]:
    n,items = n+1, items[1:]
  return n

length(range(10))
```

```python
length.__name__
```

```python
length.__doc__
```

## Class Decorator

```python
from types import FunctionType

class CheckTime:
  def __init__(self,func : FunctionType):
    self.func = func 

  def __call__(self,*args,**kwargs):
    
    return self.func(*args,**kwargs)

@CheckTime
def length(items):
  """ 
  Hàm kiểm tra số lượng phần tử của items
  """
  n = 0 
  while items is not None and items != items[1:]:
    n,items = n+1, items[1:]
  return n

length(range(10))
```

```python
from types import FunctionType

class CheckTime:
  def __init__(self,func : FunctionType):
    self.func = func 

  def __call__(self,*args,**kwargs):
    print("Bắt đầu chạy hàm {}".format(self.func.__name__))
    start_time = time.time()
    result = self.func(*args, **kwargs)
    print("Hoàn thành trong {} giây".format(time.time() - start_time))
    return result

@CheckTime
def length(items):
  """ 
  Hàm kiểm tra số lượng phần tử của items
  """
  n = 0 
  while items is not None and items != items[1:]:
    n,items = n+1, items[1:]
  return n

length(range(10))
```

```python
length.__doc__
```

```python
length.__name__
```

```python
dir(CheckTime)
```

```python
from types import FunctionType

class CheckTime:
  def __init__(self,func : FunctionType):
    self.func = func 

  def __call__(self,*args,**kwargs):
    print("Bắt đầu chạy hàm {}".format(self.func.__name__))
    start_time = time.time()
    result = self.func(*args, **kwargs)
    print("Hoàn thành trong {} giây".format(time.time() - start_time))
    return result

  def __name__(self):
    return self.func.__name__

@CheckTime
def length(items):
  """ 
  Hàm kiểm tra số lượng phần tử của items
  """
  n = 0 
  while items is not None and items != items[1:]:
    n,items = n+1, items[1:]
  return n

length(range(10))
```

```python
length.__name__()
```

```python
from types import FunctionType
import time 
import functools 

class CheckTime:
  def __init__(self,func : FunctionType):
    self.func = func 
    functools.update_wrapper(self,func)

  def __call__(self,*args,**kwargs):
    print("Bắt đầu chạy hàm {}".format(self.func.__name__))
    start_time = time.time()
    result = self.func(*args, **kwargs)
    print("Hoàn thành trong {} giây".format(time.time() - start_time))
    return result

@CheckTime
def length(items):
  """ 
  Hàm kiểm tra số lượng phần tử của items
  """
  n = 0 
  while items is not None and items != items[1:]:
    n,items = n+1, items[1:]
  return n

length(range(10))
```

```python
length.__doc__
```

## Improve Decorator

### problem

Việc gọi lại decorator "check_time" bên trên mỗi một hành vi của đối tượng là
việc làm hiệu quả nhưng khá cồng kềnh.

```python
from functools import wraps 
from types import FunctionType
import time 

def check_time(prefix = ""):
  def check(func : FunctionType):
    print(f"{prefix}\n")
    @wraps(func)
    def handle(*args):
      print("Bắt đầu chạy hàm {}".format(func.__name__))
      start_time = time.time()
      result = func(*args)
      print("Hoàn thành trong {} giây".format(time.time() - start_time))
      # handle.__name__ = func.__name__ 
      # handle.__doc__ = func.__doc__ 
      return result 
    return handle
  return check

class Person:
  def __init__(self, name : str, age : int , gender : str):
    self.name = name 
    self.age  = age 
    self.gender = gender

  @check_time(prefix = "check func study ...")
  def study(self, counrce_name : str):
    print("Bạn {} đã tham gia khóa học {}".format(self.name,counrce_name))

  @check_time(prefix = "check func play_game ...")
  def play_game(self):
    print("play game ...")
```

```python
p = Person("Python",25,"male")
```

```python
p.study("math")
```

```python
p.play_game()
```

### solution

Giải pháp được đưa ra ở đây là chúng ta sẽ tìm cách kiểu tra phương thức nào
được gọi (thông qua callable) và đặt decorator "cehck_time" vào nó (thông qua
setattr). Điều này sẽ giúp chúng ta thực thi hiệu quả và ngắn gọn.

```python
from functools import wraps 
from types import FunctionType
import time 

def check_time(prefix = ""):
  def get_class(cls):
    def check(func : FunctionType):
      print(f"{prefix}\n")
      @wraps(func)
      def handle(*args,**kwargs):
        print("Bắt đầu chạy hàm {}".format(func.__name__))
        start_time = time.time()
        result = func(*args,**kwargs)
        print("Hoàn thành trong {} giây".format(time.time() - start_time))
        return result 
      return handle

    methods = vars(cls)

    for name, value in methods.items():
      if callable(value):
        print(value,"????????")
        setattr(cls,name,check(value))

    return cls 

  return get_class

@check_time(prefix="***Check***")
class Person:
  def __init__(self, name : str, age : int , gender : str):
    self.name = name 
    self.age  = age 
    self.gender = gender

  def study(self, counrce_name : str):
    print("Bạn {} đã tham gia khóa học {}".format(self.name,counrce_name))

  def play_game(self):
    print("play game ...")
```

```python
p = Person("Python",25,"male")
```

```python
p.study("math")
```

```python
p.play_game()
```

Mở rộng cho class

```python
from types import FunctionType
import time 
import functools 

class CheckTime:
  def __init__(self,func : FunctionType):
    self.func = func 
    functools.update_wrapper(self,func)

  def __call__(self,*args,**kwargs):
    print("Bắt đầu chạy hàm {}".format(self.func.__name__))
    start_time = time.time()
    result = self.func(*args, **kwargs)
    print("Hoàn thành trong {} giây".format(time.time() - start_time))
    return result


@CheckTime
class Person:
  def __init__(self, name : str, age : int , gender : str):
    self.name = name 
    self.age  = age 
    self.gender = gender

  def study(self, counrce_name : str):
    print("Bạn {} đã tham gia khóa học {}".format(self.name,counrce_name))

  def play_game(self):
    print("play game ...")
  
```

```python
dir(CheckTime)
```
