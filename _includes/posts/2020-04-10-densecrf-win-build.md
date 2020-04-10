# 【计算机视觉】pydensecrf在windows上进行build的错误

## pydensecrf build 安装

先clone下来源码：`git clone https://github.com/lucasb-eyer/pydensecrf.git` 

然后编译：`python setup.py build_ext`

此时会出现类似如下的错误：

```c
C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.13.26128\bin\HostX86\x64\cl.exe /c /nologo /Ox /W3 /GL /DNDEBUG /MD -Ipydensecrf/densecrf/include -Ipydensecrf -Ic:\users\brain\miniconda3\include -Ic:\users\brain\miniconda3\include "-IC:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.13.26128\ATLMFC\include" "-IC:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.13.26128\include" "-IC:\Program Files (x86)\Windows Kits\NETFXSDK\4.6.1\include\um" "-IC:\Program Files (x86)\Windows Kits\10\include\10.0.16299.0\ucrt" "-IC:\Program Files (x86)\Windows Kits\10\include\10.0.16299.0\shared" "-IC:\Program Files (x86)\Windows Kits\10\include\10.0.16299.0\um" "-IC:\Program Files (x86)\Windows Kits\10\include\10.0.16299.0\winrt" "-IC:\Program Files (x86)\Windows Kits\10\include\10.0.16299.0\cppwinrt" /EHsc /Tppydensecrf/eigen.cpp /Fobuild\temp.win-amd64-3.6\Release\pydensecrf/eigen.obj
  eigen.cpp
  c:\users\brain\appdata\local\temp\pip-install-wj25x65h\pydensecrf\pydensecrf\densecrf\include\eigen\src/Core/VectorBlock.h(120): error C2373: 'Eigen::DenseBase<Derived>::segment': redefinition; different type modifiers
  c:\users\brain\appdata\local\temp\pip-install-wj25x65h\pydensecrf\pydensecrf\densecrf\include\eigen\src/Core/DenseBase.h(291): note: see declaration of 'Eigen::DenseBase<Derived>::segment'
  c:\users\brain\appdata\local\temp\pip-install-wj25x65h\pydensecrf\pydensecrf\densecrf\include\eigen\src/Core/VectorBlock.h(121): error C2447: '{': missing function header (old-style formal list?)
  c:\users\brain\appdata\local\temp\pip-install-wj25x65h\pydensecrf\pydensecrf\densecrf\include\eigen\src/Core/VectorBlock.h(152): error C2373: 'Eigen::DenseBase<Derived>::head': redefinition; different type modifiers
  c:\users\brain\appdata\local\temp\pip-install-wj25x65h\pydensecrf\pydensecrf\densecrf\include\eigen\src/Core/DenseBase.h(294): note: see declaration of 'Eigen::DenseBase<Derived>::head'
  c:\users\brain\appdata\local\temp\pip-install-wj25x65h\pydensecrf\pydensecrf\densecrf\include\eigen\src/Core/VectorBlock.h(153): error C2447: '{': missing function header (old-style formal list?)
  c:\users\brain\appdata\local\temp\pip-install-wj25x65h\pydensecrf\pydensecrf\densecrf\include\eigen\src/Core/VectorBlock.h(184): error C2373: 'Eigen::DenseBase<Derived>::tail': redefinition; different type modifiers
  c:\users\brain\appdata\local\temp\pip-install-wj25x65h\pydensecrf\pydensecrf\densecrf\include\eigen\src/Core/DenseBase.h(297): note: see declaration of 'Eigen::DenseBase<Derived>::tail'
  c:\users\brain\appdata\local\temp\pip-install-wj25x65h\pydensecrf\pydensecrf\densecrf\include\eigen\src/Core/VectorBlock.h(185): error C2447: '{': missing function header (old-style formal list?)
  error: command 'C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Tools\\MSVC\\14.13.26128\\bin\\HostX86\\x64\\cl.exe' failed with exit status 2
```

查了半天，弄了好久，才搞定，原来是`Eigen`的版本问题，下载最新的`Eigen`，用`eigen-eigen-323c052e1731\Eigen`替换`pydensecrf\densecrf\include\Eigen`这个文件夹，再进行编译即可。

----

## 另一个问题 windows上多次运行出现 `segment fault`

可能是因为pydensecrf还是比较耗内存的，可能有时会出现`segment fault`的错误，查看一下内存是否够用。

总之，相同的代码，在Linux服务器上稳定多次运行没有问题！！！

----

2020-04-10




