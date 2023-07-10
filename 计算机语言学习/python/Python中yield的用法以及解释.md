杨辉三角中yield的使用：
```
def triangles():
    N=[1]
    while True:
        yield N
        N.append(0)
        N=[N[i-1] + N[i] for i in range(len(N))]
n=0
for t in triangles():
    print(t)
    n=n+1
    if n == 10:
        break
```
yield的作用其实就是生成一个generator，使函数实例化成一个对象，for每次遍历的时候会调用generator的next()函数，对应执行函数的代码，遇到yield返回yield后面变量的值，下次遍历的时候会从上一次返回中断的地方继续执行，直到再次执行到yield,......(如此遍历下去)。当generator抛出StopIteration 异常，则表示generator遍历完成。

同样，在generator函数中也可以写return，如果函数执行遇到return，则直接抛出StopIteration 异常，表示终止遍历。
