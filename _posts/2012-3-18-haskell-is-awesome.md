---
layout: post
title: "Haskell is awesome"
author: Ulf Lilleengen
categories: haskell programming
---
 I have started to learn myself haskell using the book named "Real world Haskell".
 I have so far only come to chapter 4, but I am already in love with some of the
 features:

* Its strict static type system, which makes it easy to understand what a
  function does. Moreover, it allows you to think through what your code is going
  to do as well as make the decisions of what to do for special cases up front.
  The following is a definition of a function which compares the length of two
  lists, and returns their order (==, <, >). The definition clearly states
  that it operates on two lists of any type, and returns a value of type
  Ordering. Crystal clear!

        listCmp :: [a] -> [a] -> Ordering

* Partially due to the above point, one can avoid unpleasant bugs later on,
  because you chose to postpone your decision on what to do with your
  input.

* Pattern matching. I came across this in the Oz programming language when I was
  in university, but I didn't really understand how powerful and readable
  everything becomes until using it in Haskell. The following function takes a
  separator and a list of lists as argument, and combines the lists using the
  separator:

        intersperse :: a -> [ [a] ] -> [a]
        intersperse sep [] = []
        intersperse sep (x:[]) = x
        intersperse sep (x:xs) = x ++ [sep] ++ (intersperse sep xs)

  I love how you can just look at the patterns to see what cases is covered by
  the function, rather than nesting into some complex if sentence.

* Readability when using 'where' syntax. This is the implementation of the
  listCmp function:

        listCmp lhs rhs
            | lengthLhs < lengthRhs = LT
            | lengthLhs > lengthRhs = GT
            | otherwise             = EQ
          where lengthLhs = (length lhs)
                lengthRhs = (length rhs)

  What I like about it is that you can separate the logic performed on values
  from the function calls, so that when you read the code, you see the actual
  computation done by the function in the different cases. You can also do this
  with the let syntax, but I think the above reads really well.
