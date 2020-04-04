# 【python开发技术】面向对象的文件系统路径库 pathlib

## 基本函数使用说明

```
from pathlib import Path
a = Path(".")

- a.iterdir() # 迭代目录下的内容
- a.is_dir() # 判断是否为目录
- a.exists() # 路径属性
- a.resolve() # 消除..,.等，获取绝对路径
- a.glob("**/**.py") # 迭代当前目录树下所有python源文件
- b = a / "test" # 在目录树中移动
- a.parents # 逻辑父路径们a.parents[0]==a.parent a.parents[0].parent==a.parents[1]...

- a.parent # 逻辑父路径
- a.stem # 文件名，不含路径，不含后缀
- a.name # 文件名，排除目录，含有后缀
- a.suffix # 文件后缀名
- a.is_absolute() 判断是否为绝对路径
- a.cwd() # 返回一个新的当前目录的路径对象 与os.getcwd()返回相同
- a.home() # 返回当前用户家目录的新路径对象

- a.chmod, a.exists(), a.is_file(), a.is_socket(), a.is_symlink(), a.is_fifo(), a.is_block_device(), a.is_char_device(), a.mkdi(model=0o777, parents=False, exist_ok=False)

- a.open(model="r") # 与内置的open一样

- a.rename(target)  # 重命名

- a.replace(target) # 使用给定的target重命名文件或目录，如果target指向现存的文件或者目录，则被无条件覆盖

- a.rmdir() # 移除该目录，此目录必须为空
- a.samefile(other_path) # 返回此目录是否指向与可能时字符串或者另一个路径对象的other_path相同的文件。

- a.symlink_to(target, target_is_directory=False) # 将此路径创建为指向target的符号链接。

- a.touch(model=0o666, exist_ok=True) # 将给定的路径创建为文件

- a.unlink() # 移除此文件或者符号链接


```


----
2020-04-04




