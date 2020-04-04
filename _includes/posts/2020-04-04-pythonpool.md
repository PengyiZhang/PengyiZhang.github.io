# 【python开发技术】进程池pool的使用

## 基本说明与demo

使用进程池，python自动解决多进程的问题，特别对于没有数据依赖的列表多进程并行处理：

```
import multiprocessing as mp

# 定义工作worker
def job(x):
    return x+2

# 定义进程池
Pool = mp.Pool()

# 往进程池中送入数据，进程池就会返回worker的job的返回值
res = Pool.map(job, range(10))

```
Pool与Process不同是送入Pool的函数有返回值，而Process没有返回值。使用map获取返回结果，在map中放入worker的job函数和需要进行迭代的运算值，然后它会自动分配给CPU核，返回结果，放到list里面。

## 自定义核数量

Pool进程池设置默认为对应的设备CPU核数，当然也可以手工设置：

```
Pool = mp.Pool(processes=4) # 定义使用的CPU核数为4
```

## 其它函数

### apply_async()

只能传入一个value，且只会放入到一个CPU核中进行运算，获取结果使用get()

```
res = Pool.apply_async(job, (3,))
print(res.get())
```

当然要对多个进行处理的方式：

```
[Pool.apply_async(job, (i,)) for i in range(10).get()]
```

----
2020-04-04




