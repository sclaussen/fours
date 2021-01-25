## Four Fours Problem


From [Wikipedia - Four Fours](https://en.wikipedia.org/wiki/Four_fours):

```
Four fours is a mathematical puzzle. The goal of four fours is to find
the simplest mathematical expression for every whole number from 0 to
some maximum, using only common mathematical symbols and the digit
four (no other digit is allowed).

...

There are many variations of four fours; their primary difference
is which mathematical symbols are allowed.

Essentially all variations at least allow addition ("+"),
subtraction ("−"), multiplication ("×"), division ("÷"), and
parentheses, as well as concatenation (e.g., "44" is allowed).

Most also allow the factorial ("!"), exponentiation (e.g. "444"),
the decimal point (".") and the square root ("√") operation.

Other operations allowed by some variations include the reciprocal
function ("1/x"), subfactorial ("!" before the number: !4 equals
9), overline (an infinitely repeated digit), an arbitrary root, the
square function ("sqr"), the cube function ("cube"), the cube root,
the gamma function (Γ(), where Γ(x) = (x − 1)!), and percent
("%").
```



## Algorithm

This algorithm generates permutations of the four fours expressions
and then evaluates the expressions.  If the evaluation of an
expression results in an integer between 0 and 1000 inclusively then a
solution is found (all other solutions are ignored).

The generation algorithm works like this:
- Stage 1: Numeric permutations are generated (eg 4 4 4 4)
- Stage 2: Infix operation permutations are generated (eg 4 + 4 + 4 + 4)
- Stage 3: Parenthesis permutations are generated (eg (((4 + 4) + 4) + 4))
- Stage 4: Prefix operation permutations are generated (eg sqrt(4) + sqrt(4) + sqrt(4) + sqrt(4))
- Stage 5: Postfix operation permutations are generated (eg sqrt(4!) + sqrt(4)! + sqrt(4) + sqrt(4!))
- Stage 6: Expression evaluation



## Expression Variability

Given the wikipedia article, here are the capabilities the algorithm
supports to generate expressions.

### Numbers

- Each expression can contain from one to four numbers as long as the
  expression contains precisely four fours.
  - Valid expressions: `4 + 4 + 4 + 4`, `44 + 44`, `444 + 4`, `4444`
  - Invalid expressions: `4 + 4`, `444 + 4444`

- The fours in each number can be concatenated together.
  - Valid numbers: `4`, `44`, `444`, `4444`

- The fours in each number can contain a leading decimal, a trailing
  decimal (resulting in an integer), or a decimal in between the
  digits.
  - Valid numbers (complete set): `4`, `.4`, `44`, `.44`, `4.4`, `444`, `.444`, `4.44`, `44.4`, `4444`, `.4444`, `4.444`, `44.44`, `444.4`
  - Invalid numbers: `40`, `.04`, `.004`, `4.04`

### Parenthesis

- Parenthesis are applied to the combination of the number and the
  infix operators to force evaluation precedence.
  - Valid expressions: `(((4 + 4) + 4) + 4)`, `((44 + 4) * 4)`, `(44 + (4 * 4))`

### Operators

- The operators +, -, *, /, and ^ (power) can be used.
  - Valid expressions: `4 + 4 * 4 / 4`,  `44 - 44`,  `4 * 4 ^ 4 * 4`

- The functions square, square root, and summation can be applied to
  any number (eg square(4)) or the result of an evaluation (directly
  prior to a parenthesis) (eg square(4 + 4)).
  - Valid expressions: `square(4) + square(4) * square(4) / square(4)`,  `square(44 - 44)`,  `square(4) * square(4) ^ square(4 * 4)`

- The factorial operator can be applied to any number (directly after
  the number) or the result of an evaluation (directly after any
  parenthesis).
  - Valid expressions: `((4! + 4)! + 4)! + 4)`,  `44! / 44!`,  `(44! / 44)!`,  `(4 ^ 4 ^ 4 ^ 4)!`

### Valid expressions and limits

Invalid mathematical expressions that are known at generation time to
be invalid are not generated.
- Invalid expression: `4.4! + sum(4.4)` (decimal factorial/sum)

Expressions that result in invalid evaluations at runtime are ignored.
- Invalid expression: `(4 * (4 / (4 - 4)))` (divide by zero)
- Invalid expression: `(((4 - 4) - 4) - 4)!` (negative factorial)

There are arbitrary limits placed on the size of the numbers and
operations to speed up execution:
- No factorial for a number greater than 10 (eg 10!)
- No exponents for less than -10 or greater than 10 (eg 4^10, 4^-10)
- No square of a number less than -100 or greater than 100 (eg square(100), square(-100))
- No summation for a number less than -100 or greater than 100 (eg sum(100), sum(-100))

Note: Summations are only applied to integers.
Note: Factorials are only applied to positive integers.
Note: Square roots are only applied to zero and positive numbers.

### Rule Sets

In order to enable different combinations of rules to be applied the
algorithm supports the concept of rule sets.  The rule sets enable a
definition for which numbers and operators should be applied to
generate the possible set of expressions.

Generating all the expression permutations for functions and factorial
result in a an explosion of expressions.  There is a rule named
applyToEvaluation that determines how functions and factorial are
applied.  When set to false, functions and factorial are only applied
directly to a number.  When true, they are applied to the number, as
well as the result of a prior evaluation.  For example:

- Expression: `(((4 + 4) + 4) + 4)`
- Factorial spots (applyToEvaluation=false): `(((4! + 4!) + 4!) + 4!)`
- Factorial spots (applyToEvaluation=true): `(((4! + 4!)! + 4!)! + 4!)!`
- Function spots (applyToEvaluation=false): `(((square(4) + square(4)) + square(4)) + square(4))`
- Function spots (applyToEvaluation=true): `square(square(square(square(4) + square(4)) + square(4)) + square(4))`

Unless indicated otherwise, apply to evaluation is set to false in the
rule sets.

Here's a summary of each rule set including the total generated
expressions, how many result in an integer solution between 0 and
1000, and a description of the numbers and operations they support.

```
                Expression    Solutions  Description
              Permutations       0-1000
simple                 320          227  4 * / + -
power                  625          352  4 * / + - ^
concat                 791          425  4 44 444 4444 * / + - ^
decimal             11,930        1,846  4 44 444 4444 4.4 4.44 44.4 4.444 44.44 444.4 * / + - ^
factorial          427,066       13,982  4 44 444 4444 4.4 4.44 44.4 4.444 44.44 444.4 * / + - ^ ! (applyToEvaluation=T)
grande           2,560,000      265,649  4 * / + - ^ ! sum sqrt square
functions       97,221,656    2,847,765  4 44 444 4444 4.4 4.44 44.4 4.444 44.44 444.4 * / + - ^ sum sqrt square (applyToEvaluation=T)
advanced  ~300,000,000,000      Unknown  4 44 444 4444 4.4 4.44 44.4 4.444 44.44 444.4 * / + - ^ ! sum sqrt square (applyToEvaluation=T)
```


# Results

Here are some of the output files for each rule set.

## simple

- [Solutions: 10 shortest + 10 longest](./data/simple-solutions-10.txt)
- [Solution count](./data/simple-count.txt)
- [Solution count sorted](./data/simple-count-sorted.txt)
- [Solutions](./data/simple/)


## power

- [Solutions: 10 shortest + 10 longest](./data/power-solutions-10.txt)
- [Solution count](./data/power-count.txt)
- [Solution count sorted](./data/power-count-sorted.txt)
- [Solutions](./data/power/)


## concat

- [Solutions: 10 shortest + 10 longest](./data/concat-solutions-10.txt)
- [Solution count](./data/concat-count.txt)
- [Solution count sorted](./data/concat-count-sorted.txt)
- [Solutions](./data/concat/)


## decimal

- [Solutions: 10 shortest + 10 longest](./data/decimal-solutions-10.txt)
- [Solution count](./data/decimal-count.txt)
- [Solution count sorted](./data/decimal-count-sorted.txt)
- [Solutions](./data/decimal/)


## factorial

- [Solutions: 10 shortest + 10 longest](./data/factorial-solutions-10.txt)
- [Solution count](./data/factorial-count.txt)
- [Solution count sorted](./data/factorial-count-sorted.txt)
- [Solutions](./data/factorial/)


## functions

- [Solutions: 10 shortest + 10 longest](./data/functions-solutions-10.txt)
- [Solution count](./data/functions-count.txt)
- [Solution count sorted](./data/functions-count-sorted.txt)
- [Solutions](./data/functions/)
