# Exercise1.5 课后习题讨论

## 习题：

```lisp
(define (p) (p) )

(define (test x y) (if (= x 0)
                               0
                               y))
( test 0 (p) )
```

知识点：

>  Applicative-Order 和 Normal-Order ：前者为 Interpreter 解释器实际所用方式；后者又可称之为Lazy-evaluate。

* Applicative-Order 书中写到 evaluate the arguments then apply，先把参数列表中的 combinator或procedure（TODO：这两者的区别？？）计算到只有primitive data为止，然后再进一步的执行
* Normal-Order 书中写到fully expand then reduce ，完全展开表达式，然后替换表达式中的参数列表，这也就是为什么叫做lazy-evaluate。

注意点：

`(test 0 (p))` 和 `(test 0 p)` 是有区别的，前者是一个procedure 由于是Applicative-Order 需要提前evaluate，根据定义会循环调用 （我的猜测是无无限`((((p))))` 这种）；而后者是直接将`p`作为一个参数传入 是不需要提前evaluate的！这很关键。换做现在的说法，包了一层括号`()`，感觉就是调用并执行函数p，而后者是将函数p作为参数传入。

## 来自StackOverflow的解答：

What research have you done to answer that question? I just plugged it as it is in Google, and got as second answer (the first may be as good, I did not check) a reference to a section of a bible on your topic: Hal Abelson's, Jerry Sussman's and Julie Sussman's [Structure and Interpretation of Computer Programs](https://mitpress.mit.edu/sicp/) (MIT Press, 1984; ISBN 0-262-01077-1), aka the wizard book. The reference is to the section "[Normal Order and Applicative Order](https://mitpress.mit.edu/sicp/full-text/sicp/book/node85.html)".

It states:

> Scheme is an applicative-order language, namely, that all the arguments to Scheme procedures are evaluated when the procedure is applied. In contrast, normal-order languages delay evaluation of procedure arguments until the actual argument values are needed.

and adds that the latter is called *lazy evaluation*.

So you have your definitions wrong, and taking one for the other:

- applicative order evaluates subexpressions when, i.e. just before, the procedure is applied.
- normal order passes subexpressions as they are, without evaluation, and proceeds with the evaluation only when the corresponding formal parameter is actually to be itself evaluated. (there is a further twist to it regarding environment issues ... but we better forget that at this point).

Furthermore, you do not properly understand the call mechanism which involves two things:

- the parameter passing mechanism, which include proper processing of actual arguments to the call, depending on the evaluation rule;
- the "replacement" of the call to the function by the body of the function (without the header).

In the case of **applicative order evaluation** of ( test 0 (p) ), you are supposed to evaluate the argument subxpressions first. These are 0 and (p).

- evaluation of a literal value like 0 yield that value.
- the second argument however is a procedure call to a parameter-less procedure called p. It has no parameter, so that we have no worry about evaluation order. Then, in order to pursue evaluation, we have to replace the call by the body of the procedure which follows the list of arguments, and then evaluate that body. The body of procedure p, as defined by the declaration (define (p) (p) ), is (p), so that we are left with the evaluation of what we were just trying to evaluate. In order words, the evaluation process is caught in a loop, and will not terminate.

... and you never get to actually finish the call to the function test, since evaluation of its arguments does not terminate. Your program does not terminate.

This risk of non termination, even when the guilty argument will never be used in the call, is one of the reasons to use instead normal order evaluation, which may be a bit harder to implement, but may have better termination properties.

Under **normal order evaluation**, you do not touch the argument sub-expressions. What you do is replace the call ( test 0 (p) ) by the body of the function test, i.e. (if (= x 0) 0 y), where the names of the (formal) arguments x and y are replaced by the corresponding actual arguments 0 and (p) (up to environment, or renaming issues, that are important but would complicate the explanation here, and are the main difference between the original Lisp and the Scheme language).

Hence you replace the evaluation of ( test 0 (p) ) by the evaluation of (if (= 0 0) 0 (p)).

Now the function if is a primitive function that usually always evaluate its first argument, but evaluates its last 2 arguments in normal order, evaluating only the useful one, depending on whether the first evaluates to *false* or *true* (actually NIL or #f for *false*, or some other value for *true*, in the case of Scheme - if my memory does not fail me). Since (= 0 0) evaluates to *true*, evaluation of the conditional amount to evaluating the yet unevaluated second argument, which is 0, which unsurprisingly (except in old Fortran) evaluates to 0.

## 来自MIT课后解答：

In section , where we began our discussion of models of evaluation, we noted that Scheme is an *applicative-order* language, namely, that all the arguments to Scheme procedures are evaluated when the procedure is applied. In contrast, *normal-order* languages delay evaluation of procedure arguments until the actual argument values are needed. Delaying evaluation of procedure arguments until the last possible moment (e.g., until they are required by a primitive operation) is called *lazy evaluation*.  Consider the procedure

```
(define (try a b)
(if (= a 0) 1 b))
```

Evaluating (try 0 (/ 1 0)) generates an error in Scheme. With lazy evaluation, there would be no error. Evaluating the expression would return 1, because the argument (/ 1 0) would never be evaluated.

An example that exploits lazy evaluation is the definition of a procedure unless

(define (unless condition usual-value exceptional-value)

  (if condition exceptional-value usual-value))

that can be used in expressions such as

```
(unless (= b 0)
        (/ a b)
        (begin (display "exception: returning 0")
               0))
```

This won't work in an applicative-order language because both the usual value and the exceptional value will be evaluated before unless is called (compare exercise ). An advantage of lazy evaluation is that some procedures, such as unless, can do useful computation even if evaluation of some of their arguments would produce errors or would not terminate.

If the body of a procedure is entered before an argument has been evaluated we say that the procedure is *non-strict* in that argument. If the argument is evaluated before the body of the procedure is entered we say that the procedure is *strict* in that argument.  In a purely applicative-order language, all procedures are strict in each argument. In a purely normal-order language, all compound procedures are non-strict in each argument, and primitive procedures may be either strict or non-strict. There are also languages (see exercise ) that give programmers detailed control over the strictness of the procedures they define.

A striking example of a procedure that can usefully be made non-strict is cons (or, in general, almost any constructor for data structures). One can do useful computation, combining elements to form data structures and operating on the resulting data structures, even if the values of the elements are not known. It makes perfect sense, for instance, to compute the length of a list without knowing the values of the individual elements in the list. We will exploit this idea in section  to implement the streams of chapter 3 as lists formed of non-strict cons pairs.

**Exercise.** Suppose that (in ordinary applicative-order Scheme) we define unless as shown above and then define factorial in terms of unless as

```
(define (factorial n)
  (unless (= n 1)
          (* n (factorial (- n 1)))
          1))
```

What happens if we attempt to evaluate (factorial 5)? Will our definitions work in a normal-order language?

**Exercise.** Ben Bitdiddle and Alyssa P. Hacker disagree over the importance of lazy evaluation for implementing things such as unless. Ben points out that it's possible to implement unless in applicative order as a special form. Alyssa counters that, if one did that, unless would be merely syntax, not a procedure that could be used in conjunction with higher-order procedures. Fill in the details on both sides of the argument. Show how to implement unless as a derived expression (like cond or let), and give an example of a situation where it might be useful to have unless available as a procedure, rather than as a special form.  