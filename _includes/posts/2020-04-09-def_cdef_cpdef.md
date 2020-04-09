# 【python开发技术】Cython 开发学习之def, cdef, cpdef


由于Cython是基于C运行时，因此，允许使用cdef和cpdef。



这个三个东西的使用有点乱七八糟，下面梳理一下。

关键的不同在于这些`function`可以在哪里面进行调用

def可以从Python，Cython中调用，而cdef可以被Cython和C中调用。都可以用混合 typed and untyped 变量进行声明，这两种情况内部都会被Cython编译为C，被编译的代码应该会非常相像。

```
# A Cython class for illustrative purposes
cdef class C:
   pass

def f(int arg1, C arg2, arg3):
    # takes an integer, a "C" and an untyped generic python object
    pass

cdef g(int arg1, C arg2, arg3):
    pass

```

这例子中，`f`对python可见，g不能被python调用，g将会翻译成`C signature`: `PyObject* some_name(int, struct __pyx_obj_11name_of_module_C *, PyObject*)`，这样g就可以当做函数指针被传递给C的函数，而`f`则不能直接被C调用。





## cdef 的一些限制

cdef不能在别的函数中被定义，这是因为在C的函数指针中无法存储任何捕获的变量，下面的例子就是非法的：

```
# WON'T WORK!
def g(a):
   cdef (int b):
      return a+b
```

cdef函数不能接受*args, **kwargs类型的变量，因为它们不容易被翻译为C标签。

## cdef的好处

cdef可以使用任何类型的变量，包括 int* a等。而def不能这么定义。

cdef可以声明返回值类型，如果没有被声明，默认返回`PyObject* in C`。而def总是返回一个Python对象，所以不能声明返回值。


```
cdef int h(int* a):
    # specify a return type and take a non-Python compatible argument
    return a[0]
```

cdef比def更快，因为它被翻译为简单的C函数调用。

## cpdef的函数

cpdef会让Cython生成`一个cdef函数`（允许一个从Cython中来的快速的函数调用）和`一个def函数`（允许从python进行调用）。而在内部这个def函数是直接调用了cdef的函数。所以，变量类型限制是cdef和def两者的交集。

## 什么时候使用cdef函数？

当函数被调用后，这cdef和def两者内部的速度是没有差别的。因此，在下列情况再用cdef：

- 需要传递non-Python类型 in or out
- 需要作为函数指针传递给C
- 经常调用，加速该函数调用很重要，而不需要从python中调用。

-----------
2020-04-09

