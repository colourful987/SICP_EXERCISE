> 总结来说：`new-if` is a function. When a function is called, what's the first thing that Scheme does with the argument list? It evaluates *all* the arguments.



如果将 procedure 当做函数，对于Applicative-Order的硬性规则是先将所有参数 evaluate 一遍！

*talk is cheap, show me the code!*

```
(sqrt 2) 
```

替换sqrt 往下走:

```
(sqrt-iter 1 2)
```

继续替换sqrt-iter:

```
(new-if (good-enough? guess x)
			guess
			(sqrt-iter (improve guess x)
			x))
```

这里就是问题的关键所在，new-if 标识的是一个procedure，正如我前面说的，所有的传参都要evaluate一遍，也就是计算一遍，包括：`(good-enough? guess x)`, `guess`,`(sqrt-iter (improve guess x) x)`。

前两个显然很容易就计算出来了，但是第三个参数貌似有点问题，我先替换一部分：

```
(new-if false
			1
			(sqrt-iter 2 3))
```

显然 `(sqrt-iter 2 3)`  作为传参我们并没有完全计算出来，那么需要进一步替换成新的 `new-if` procedure，但是传入的`guess=2 x=3`，替换后你会发现，死循环了! new-if 套一个new-if，无穷无尽。

那么问题来了，为什么用 if 就没有问题呢？ 省略之前一样的替换操作，直到关键部分：

```
(if (false) 
	1
	(sqrt-iter 2 3)))
```

if 只是个 primitive-predicate ，即原始操作符，并非函数，需要等待参数计算完毕才能执行，这里会先执行if(false) ，最后只剩下：

```
(sqrt-iter 2 3)
```

之后就好理解了，不进一步展示。


