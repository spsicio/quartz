---
title: 如何生成勾股数
date: 2025-09-29
tags: [math]
---

## 理论

勾股数是满足 $a^2 + b^2 = c^2$ 的正整数三元组 $(a, b, c)$。如果 $a, b, c$ 互质，这一组勾股数就称为本原勾股数。如果想要得到所有的勾股数，只需要先找到其中本原的部分即可。那么，如何高效地生成所有本原勾股数呢？

欧几里得给出了这样一个生成本原勾股数的公式：

$$
\begin{cases}
  a = m^2 - n^2 \\
  b = 2mn \\
  c = m^2 + n^2 \\
\end{cases}
$$

其中 $m, n$ 为互质的正整数，$m > n > 0$，且恰好一奇一偶。通过调整 $m, n$ 这两个参数，我们可以得到所有的本原勾股数，其中 $a$ 一定是奇数，$b$ 一定是偶数。具体的证明过程可以参考 [维基百科](https://en.wikipedia.org/wiki/Pythagorean_triple)。

如果将视角放到复数域，我们令

$$
z = m + n\mathrm{i}
$$

有

$$
\begin{aligned}
  z^2 &= a + b \mathrm{i} \\
  c^2 &= (a + b \mathrm{i}) \cdot (a - b \mathrm{i}) 
\end{aligned}
$$

我们可以发现本原勾股数对应的复数开方之后，实部和虚部仍均为整数！这是为何？

实际上，这样实部和虚部均为整数的复数被称为高斯复数。高斯复数形成的环为为唯一分解整环，相当于满足算数基本定理。在 $a, b$ 互质且一奇一偶的情况下，$a + b \mathrm{i}$ 和 $a - b \mathrm{i}$ 也互质。而他们的乘积为平方数（高斯整数的平方），因此 $a + b \mathrm{i}$ 一定也是平方数。

## 实现

知晓了欧几里得公式之后，问题可以转变为遍历所有的互质正整数对。这可以通过从 $(1, 2)$ 出发，应用更相减损的逆过程实现。为了确保生成的正整数对不能为两个奇数，可以对两个奇数的情况再次应用一遍此过程。

```haskell
{- |
> take 10 coprimePairs
> [(1,2),(1,3),(2,3),(1,4),(3,4),(2,5),(3,5),(1,5),(4,5),(3,7)]
-}
coprimePairs :: [(Int, Int)]
coprimePairs =
    (1, 2) : concatMap generate coprimePairs
  where
    generate (n, m) = [(n, n + m), (m, n + m)]

{- |
> take 10 euclidPairs
> [(1,2),(2,3),(1,4),(3,4),(2,5),(3,8),(5,8),(4,5),(1,6),(5,6)]
-}
euclidPairs :: [(Int, Int)]
euclidPairs =
    (1, 2) : concatMap generate euclidPairs
  where
    generate (n, m) =
        let (o, e) = if odd n then (n, m) else (m, n)
         in [(e, o + e), (o, e + o + o), (e + o, e + o + o)]
```

可以认为 `coprimePairs` 是对 [Calkin-Wilf Tree](https://en.wikipedia.org/wiki/Calkin%E2%80%93Wilf_tree) 的变体做宽度优先遍历的结果。 如果需要按照 $c$ 的大小从小到大生成勾股数，参考 Dijkstra 算法可以使用优先队列实现。
