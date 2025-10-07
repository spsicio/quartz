---
title: 重返 SKI，发现应用函子
created: 2025-09-30
modified: 2025-10-05
tags: [pl-theory]
---

## SKI Combinator

[Lambda 演算](https://en.wikipedia.org/wiki/Lambda_calculus) 使用「变量 Variable」、「应用 Application」、「抽象 Abstraction」三种形式的项就实现了图灵完备的计算模型。而 [SKI 组合子](https://en.wikipedia.org/wiki/SKI_combinator_calculus) 更进一步，在引入 $S, K$ 两个组合子之后消除了所有「抽象」形式的项。其中「应用」与「抽象」分别指的是函数的调用与定义。也就是说，使用 SKI 组合子表示函数时，我们无法在函数的内部引用参数，只能使用已有的函数进行组合，这对应了 [point-free](https://en.wikipedia.org/wiki/Tacit_programming) 这种编程范式。

以函数的组合 $(f \circ g)(x) = f(g(x))$ 为例，Haskell 可以使用以下任意方式对 $\circ$ 进行定义，其中只有第四种符合 point-free 的风格。

```haskell
-- not point-free
compose1 :: (b -> c) -> (a -> b) -> (a -> c)
compose1 f g x = f $ g x

-- not point-free
compose2 :: (b -> c) -> (a -> b) -> (a -> c)
compose2 f g = f . g

-- not point-free
compose3 :: (b -> c) -> (a -> b) -> (a -> c)
compose3 f = (f .)

-- point-free
compose4 :: (b -> c) -> (a -> b) -> (a -> c)
compose4 = (.)
```

我认为 SKI 组合子的一个重要意义就是保证了所有的函数都可以用 point-free 的风格表示。（不过这样表示之后是否容易理解就是另一回事。)

那么 SKI 组合子是如何将函数中所有的参数消去的呢？先对其使用的组合子进行介绍：

$$
\begin{aligned}
  S\,x\,y\,z &= x\,z\,(y\,z) \\
  K\,x\,y &= x \\
\end{aligned}
$$

就我个人的体验来说，第一次看到他们的定义时，我觉得十分困惑。这里先以 SKI 组合子中的函数组合 $\mathrm{compose} = S\,(K\,S)\,K$ 为例，演示一下这几个组合子的使用：

$$
\begin{aligned}
  & (S\,(K\,S)\,K)\,f\,\textcolor{grey}{g\,z} \\
  \overset{S}{\rightarrow}\ & (K\,S)\,f\,(K\,f)\,\textcolor{grey}{g\,z} \\
  \overset{K}{\rightarrow}\ & S\,(K\,f)\,g\,z \\
  \overset{S}{\rightarrow}\ & (K\,f)\,z\,(g\,z) \\
  \overset{K}{\rightarrow}\ & f\,\textcolor{gray}{(g\,z)}
\end{aligned}
$$

使用 $T[\ ]$ 表示 Lambda 演算项到 SKI 组合子项的转换。具体规则可以参考维基百科的介绍，这里仅列出我认为最重要的两条：

1. $T[\lambda x. M] = K\;T[M]$，其中 $M$ 项中不包含 $x$
2. $T[\lambda x. M\,N] = S\;T[\lambda x. M]\;T[\lambda x. N]$

## Reader Applicative

在了解了 [Functor](https://en.wikipedia.org/wiki/Functor_(functional_programming)), [Applicative](https://en.wikipedia.org/wiki/Applicative_functor), [Monad](https://en.wikipedia.org/wiki/Monad_(functional_programming))（函子、应用函子、单子）等概念之后重温，$K, S$ 的定义变得格外熟悉。实际上，他们和 Reader Applicative 中 `pure` 和 `apply` 定义是一致的。

Reader Applicative 用于共享外部环境时的函数计算。Reader 代表读取外部环境，Applicative 代表函数计算。为了共用外部环境，Reader Applicative 在所有函数的定义中增加一个代表环境的参数，并在调用其他函数的时将环境参数继续传递。变量也相应地转化为函数，从而涵盖使用环境中变量的情况。

考虑环境的类型为 `r`，我们用 `Reader r a` 表示 `a` 类型转换之后的结果，也就是函数类型 `r -> a`。在值或函数不依赖外部的环境时，我们使用 `pure`（对应组合子的 $K$）将类型从 `a` 转换到 `r -> a`。所有的类型均转换之后，进入到了 Reader 的计算世界。Reader 世界中，函数应用使用 `apply`（对应组合子的 $S$）实现。具体而言，为 `x` 与 `y` 提供环境参数 `z`，可以得到原始的的函数与参数（可视为使用全局变量共用环境的版本），然后执行函数应用即可得到结果。

```haskell
-- pure
k :: a -> (r -> a)
k x y = x

-- apply
s :: (r -> (a -> b)) -> (r -> a) -> r -> b
s x y z = x z (y z)

-- also point-free
compose5 :: (b -> c) -> (a -> b) -> (a -> c)
compose5 = s (k s) k
```

虽然表面上 `(s x y z) :: b` 离开了 Reader 的世界，但是 `(s x y) :: r -> b` 仍停留其中。在传递环境参数之前，我们可以随意执行 Reader 世界中的函数应用，就好像 `r` 并不存在一般。

理解了 Reader Applicative 之后，可以认为 $T[\ ]$ 的转换思路在于将所有函数体内使用的参数均当作外部环境的一部分，利用 Reader Applicative 统一管理。
