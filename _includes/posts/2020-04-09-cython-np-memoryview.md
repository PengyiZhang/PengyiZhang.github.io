# 【python开发技术】Cython 开发学习之np.ndarray与memoryviews

## 问题描述

Cython中常用的np.ndarray只能作为函数的local局部变量定义，而不能作为global，比如cdef class类的成员变量来管理。

```
# [Error]: Buffer types only allowed as function local variables
cdef class Seg:
    DTYPE = np.float32
    ctypedef np.float32_t DTYPE_t
    cdef np.ndarray[DTYPE_t] delay_buffer # np.ndarray

```

## 解决方法

此时，需要定义类成员变量为memoryview才可以实现。

```
# [right]
cdef class Seg:
    DTYPE = np.float32
    ctypedef np.float32_t DTYPE_t
    cdef DTYPE_t[:] delay_buffer # memoryviews
```
## np.ndarray与memoryviews之间的相互转化

此时，需要使用np的函数进行调用，需要将memoryviews转成np.ndarray进行处理。操作的时候直接调用`np.asarray()`即可。

而np.ndarray是可以直接赋值给相同形状的 memoryviews

```
cdef np.uint8_t[:,:,:] outputMask
self.outputMask = np.zeros_like(self.images)
```
## np.asarray与np.array的区别和注意

这两者的区别是，对于一个Image.open打开的图像，如果直接使用np.asarray得到的array是一个read_only的buffer，而使用np.array则会是一个新的。原因是asarray用的还是原来的数据buffer，而array则会新建一个buffer。

此时对于一个read_only的array，如果给其赋值则会报错。

-----

2020-04-09 

