# 【python开发技术】`Boost.Python` 封装`python`接口的`C/C++`代码

## 何为`Boost.Python`？

`Boost.Python`是 `Boost` 用来给 `python` 提供调用 `C++` 代码的库。

所以，首先得下载Boost源码，然后编译，使能python：

```shell

b2.exe --with-python link=shared

```

`windows` 上会生成：`boost_python36-vc141-mt-x64-1_72.lib/dll`，这两个东西将是后面所有将 `C/C++` 代码封装为 `python` 库使用所需要的。


## `Boost.Python` 封装`python`接口的`C/C++`代码

### 整个流程说明

#### 一个 `hello world`

```C++
int fact(int n)
{
    if (n<=1) return 1;
    else return n*fact(n-1);
}

int my_mod(int x, int y)
{
    return (x%y);
}


char const* greet()
{
    return "hello, world";
}
```

很简单的三个函数功能，下面很快用Boost.Python进行封装：

```C++
#include <boost/python.hpp>

BOOST_PYTHON_MODULE(hello_ext)
{
    using namespace boost::python;
    def("greet", greet);
    def("my_mod", my_mod);
    def("fact", fact);

}
```

这样就直接串联起来了，上面所有代码放置在 `hello_wrap.cpp`中。那么类似Cython\SWIG\Python单独封装，这里仍然采用 `distutils` 来进行构建：

```python

from distutils.core import setup, Extension

example_module = Extension('hello_ext',
                    define_macros = [('MAJOR_VERSION', '0'),
                                     ('MINOR_VERSION', '1')],
                    sources = ['hello_wrap.cpp', ],
                    libraries = [],
                    include_dirs = ["D:\\boost_1_72_0"],
                    library_dirs = ["D:\\boost_1_72_0\\stage\\lib"],
                    extra_compile_args = ["/MD", "/O1"])

setup(name='example',
      version="0.1",
      author="ZhangPY",
      description="Simple Boost.Python",
      ext_modules=[example_module],
      py_modules=['example']
    )

```

这样就可以很容易的链接到上面 `boost_python36-vc141-mt-x64-1_72.lib`，然后编译并生成 `hello_ext.cp36-win_amd64.pyd`，导入即可发现：`hello_ext.greet`、`hello_ext.my_mod`、`hello_ext.fact`

就可以方便的使用啦！

## 参考文档

更加详细的内容可以参看：

[https://www.boost.org/doc/libs/1_66_0/libs/python/doc/html/tutorial/index.html](https://www.boost.org/doc/libs/1_66_0/libs/python/doc/html/tutorial/index.html)

-----
20200412

