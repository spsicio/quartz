---
title: 模式匹配的求值陷阱
created: 2025-10-19
modified: 2025-10-19
tags: [debug]
---

近日，一位朋友在完成 [fp-course](https://github.com/system-f/fp-course) 的练习时遇到了一个奇怪的问题。他对 Parser 的 `<*>` 实现看起来似乎没有任何问题，但是无法通过测试。

```haskell
instance Applicative Parser where
    pure :: a -> Parser a
    pure = valueParser
    (<*>) :: Parser (a -> b) -> Parser a -> Parser b
    (<*>) (P f) (P p) =
        P $ \input -> case f input of
            UnexpectedEof -> UnexpectedEof
            ExpectedEof i -> ExpectedEof i
            UnexpectedChar c -> UnexpectedChar c
            UnexpectedString s -> UnexpectedString s
            Result i func -> case p i of
                UnexpectedEof -> UnexpectedEof
                ExpectedEof i' -> ExpectedEof i'
                UnexpectedChar c -> UnexpectedChar c
                UnexpectedString s -> UnexpectedString s
                Result i' a -> Result i' (func a)
```

正确的 `<*>` 实现利用 Monad 可以很简单：

```haskell
(<*>) pf pa = (<$> pa) =<< pf
```

不过将这两个神秘的运算符 `<$>` 和 `=<<` 按照定义展开也会得到和上面实现类似的结构，做着类似的事情。

```haskell
instance Functor Parser where
    (<$>) :: (a -> b) -> Parser a -> Parser b
    (<$>) f (P p) =
        P $ \chars -> case p chars of
            UnexpectedEof -> UnexpectedEof
            ExpectedEof i -> ExpectedEof i
            UnexpectedChar c -> UnexpectedChar c
            UnexpectedString s -> UnexpectedString s
            Result i a -> Result i (f a)
            
instance Monad Parser where
    (=<<) :: (a -> Parser b) -> Parser a -> Parser b
    (=<<) f (P pa) =
        P $ \chars ->
            onResult (pa chars) $ \rest a -> parse (f a) rest

onResult ::
    ParseResult a ->
    (Input -> a -> ParseResult b) ->
    ParseResult b
onResult UnexpectedEof _ = UnexpectedEof
onResult (ExpectedEof i) _ = ExpectedEof i
onResult (UnexpectedChar c) _ = UnexpectedChar c
onResult (UnexpectedString s) _ = UnexpectedString s
onResult (Result i a) k = k i a           
```

经过探查，最终发现问题出现在一个细微的地方——函数定义时使用的**模式**：

```haskell
(<*>) (P f) (P a)
```

这里使用了 `(P a) :: Parser b` 进行**模式匹配**，从而触发了对它的**求值**——必须求值直到知道该 `Parser b` 的构造子是否是 `P`。（当然，变量模式和通配符模式不会触发求值）在传入的参数涉及到递归定义就可能出现问题：

```haskell
list1 :: Parser a -> Parser (List a)
list1 p = (:.) <$> p <*> list p

list :: Parser a -> Parser (List a)
list p = list1 p ||| pure Nil

(|||) :: Parser a -> Parser a -> Parser a
(|||) (P pa) (P pb) =
    P $ \chars ->
        let res = pa chars
         in if isErrorResult res
                then pb chars
                else res
```

在这个参数的位置传入 `list p` 时，Haskell 为了获得构造子按照函数的定义对 `list p` 进行展开。使用的 `|||` 也使用了模式匹配因此需要将 `list1 p` 进行展开。`list1 p` 又会使用 `<*>` 在这个参数位置传入 `list p`，陷入死循环之中。除了修改 `<*>` 之外，还可以考虑修改 `|||` 的定义，都能规避模式匹配时求值带来的死循环。

总之，模式匹配须谨慎，尤其是在函数定义这样不起眼的地方——这里没有使用 `case` 和 `let` 这样的关键字明确指出模式匹配的行为。
