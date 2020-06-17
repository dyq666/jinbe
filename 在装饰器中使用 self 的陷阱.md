## 问题描述

假设有一个装饰器想要为实例动态增加一个属性 name.

```python
from functools import wraps

def naming(f):
    @wraps(f)
    def wrapper(self, *args, **kwargs):
        self.name = f.__name__
        return f(self, *args, **kwargs)
    return wrapper

class A:
    @naming
    def xiaohong(self):
        pass

a = A()
a.xiaohong()
print(a.name)  # xiaohong
```

看起来好像一切顺利, 但如果手动调用 `naming` 就会报错.

```python
class A:
    @naming
    def xiaohong(self):
        pass
    
    def lilei(self):
        pass

a = A()
naming(a.lilei)()  # TypeError: wrapper() missing 1 required positional argument: 'self'
```

如果你在 flask-restful 中使用 method_decorators, 那么可能就会遇到上述问题. flask-restful 相关的核心代码如下, 从下面的代码中可以发现 method_decorators 和上面的 lilei 一样是手动调用装饰器的方式, 而不是通过 @xx 的方式.

```python
class Resource(MethodView):
    ...
    method_decorators = []
    ...
    def dispatch_request(self, *args, **kwargs):
        ...
        meth = getattr(self, request.method.lower(), None)
        ...
        for decorator in self.method_decorators:
            meth = decorator(meth)
        resp = meth(*args, **kwargs)
        ...
```

## 原因

实际上, 当你通过实例手动调用装饰器时传入的 f 是一个 method, 而使用 @xx 的方式传入的 f 是一个 function, 如下例所示, 可以借助 inspect 模块来观察此数据. 另外从下例中可以看到 f 是 function 时会传入一个参数 (这里传入的实例 a), 而 f 是 method 时不会传入参数.

```python
import inspect
from functools import wraps

def naming(f):
    @wraps(f)
    def wrapper(*args, **kwargs):
        print(inspect.isfunction(f), inspect.ismethod(f), args, kwargs)
        return f(*args, **kwargs)
    return wrapper

class A:
    @naming
    def xiaohong(self):
        pass
    
    def lilei(self):
        pass

a = A()
a.xiaohong()  # True, False, (<__main__.A object at 0x10c6189d0>,) {}
naming(a.lilei)()  # False, True, () {}
```

更进一步说, @xx 的方式等价于通过类手动调用装饰器, 而不是通过实例手动调用装饰器, 示例如下:

```python
class A:
    @naming
    def xiaohong(self):
        pass
    
    def lilei(self):
        pass
    
A.lilei = naming(A.lilei)
a = A()
a.xiaohong()  # True, False, (<__main__.A object at 0x10c6189d0>,) {}
a.lilei()  # True, False, (<__main__.A object at 0x10c6189d0>,) {}
```

## 解决办法

根据传入的 f 是 method or function, 分开处理就好, 代码如下. 当然更好的解决办法是不要在装饰器中修改 self !

```python
import inspect
from functools import wraps

def naming(f):
    @wraps(f)
    def wrapper(*args, **kwargs):
        if inspect.ismethod(f):
            # 可以通过阅读 inspect 模块, 知道 method 都有哪些属性.
            # https://docs.python.org/3/library/inspect.html
            f.__self__.name = f.__name__
        else:
            # args[0] == self
            args[0].name = f.__name__
        return f(*args, **kwargs)
    return wrapper
```

