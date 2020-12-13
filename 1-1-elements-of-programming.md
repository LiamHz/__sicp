## Notes
Scheme uses prefix notation, where the operator is to the left of the operands

```scheme
(+ 12 13 2)
(+ (* 3 5) (- 10 6))
```

Normal-order evaluation: Fully expand then reduce

Applicative-order evaluation: Evaluate the arguments and then apply

The below shows an example of how these two evaluation orders differ

```scheme
(define (square x) (* x x))

(define (sum-of-squares x y)
  (+ (square x) (square y)))

; Applicative-order
(sum-of-squares (+ 5 1) (* 5 2))
(+ (square 6) (square 10))
(+ (* 6 6) (* 10 10))
(+ 36 100)
136

; Normal-order
(sum-of-squares (+ 5 1) (* 5 2))
(+ (square (+ 5 1)) (square (* 5 2)))
(+ (* (+ 5 1) (+ 5 1)) (* (* 5 2) (* 5 2)))
(+ (* 6 6) (* 10 10))
(+ 36 100)
136
```

Lisp uses short-circuiting when evaluating conditionals

Declarative descriptions are "what is". EX: sqrt(x) is y such that y^2 = x.

Imperative descriptions are "how to". EX: Use successive approximations to find sqrt(x)

Computer science usually cares about imperative descriptions.

Procedural abstraction is where you can treat a procedure as a "black box". You know what input it needs, what it'll output, but how it works doesn't need to be known.

## Exercises
### Exercise 1.1: Evaluating Expressions
_Below is a sequence of expressions. What is the result printed by the interpreter in response to each expression? Assume that the sequence is to be evaluated in the order in which it is presented._
```scheme
10                                  ; 10
(+ 5 3 4)                           ; 12
(- 9 1)                             ; 8
(/ 6 2)                             ; 3
(+ (* 2 4) (- 4 6))                 ; 6
(define a 3)                         
(define b (+ a 1))                   
(+ a b (* a b))                     ; 19
(= a b)                             ; false
(if (and (> b a) (< b (* a b)))      
    b                                
    a)                              ; 4
(cond ((= a 4) 6)                    
      ((= b 4) (+ 6 7 a))            
      (else 25))                    ; 16
(+ 2 (if (> b a) b a))              ; 6
(* (cond ((> a b) a)                 
         ((< a b) b)                 
         (else -1))                  
   (+ a 1))                         ; 16
```

### Exercise 1.2: Prefix Form Practice
_Translate the following expression into prefix form:_

$$\frac{5+4+(2−(3−(6+\frac{4}{5})))}{3(6−2)(2−7)}$$

```scheme
(/ (+ 5
      4
      (- 2
        (- 3
          (+ 6 (/ 4 5))
        )
      )
    )
    (* 3
       (- 6 2)
       (- 2 7)
    )
)
```

### Exercise 1.3: Sum of Largest Squares
_Define a procedure that takes three numbers as arguments and returns the sum of the squares of the two larger numbers._

```scheme
(define (sum-of-squares a b) (+ (* a a) (* b b)))
(define (sum-of-largest-squares a b c)
  (cond ((and (< c a) (< c b)) (sum-of-squares a b))
        ((and (< b a) (< b c)) (sum-of-squares a c))
        (else (sum-of-squares b c))
  )
)

(sum-of-largest-squares 5 4 7)
(sum-of-largest-squares 3 4 7)
(sum-of-largest-squares 5 4 2)
```

### Exercise 1.4: Operators from Compound Expressions
_Observe that our model of evaluation allows for combinations whose operators are compound expressions. Use this observation to describe the behavior of the following procedure:_

```scheme
(define (a-plus-abs-b a b)
  ((if (> b 0) + -) a b))

(a-plus-abs-b 5 2)
(a-plus-abs-b 5 -2)
```

It adds `a` with the absolute value of `b`

### Exercise 1.5: Applicative-Order and Normal-Order Evaluation
_Ben Bitdiddle has invented a test to determine whether the interpreter he is faced with is using applicative-order evaluation or normal-order evaluation. He defines the following two procedures. What behavior will he observe after evaluation `(test 0 p)` with an interpreter that uses applicative-order evaluation? With normal-order evaluation?_

```scheme
(define (p) (p))

(define (test x y) 
  (if (= x 0) 0 y))

(test 0 p)
```

With applicative-order evaluation, the function is expanded first to `(if (= 0 0) 0 p)`, and due to conditional short-circuiting, p is never evaluated.

With normal-order evaluation, both 0 and `p` will be evaluated. `p` is undefined, and will cause an error.

### Exercise 1.6: The Special form of if
_Alyssa P. Hacker doesn't see why if needs to be provided as a special form. "Why can't I just define it as an ordinary procedure in terms of cond?" she asks. Alyssa's friend Eva Lu Ator claims this can indeed be done, and she defines a new version of if:_

```scheme
(define (new-if predicate 
                then-clause 
                else-clause)
  (cond (predicate then-clause)
        (else else-clause)))
```
_What happens when Alyssa attempts to use the following to compute square roots?_
```scheme
(define (sqrt-iter guess x)
  (new-if (good-enough? guess x)
          guess
          (sqrt-iter (improve guess x) x)))
```

Because of applicative-order evaluation, all the expressions in the `new-if` function that uses `cond` will be evaluated. This causes an infinite evaluation loop because of the recursive nature of `sqrt-iter`.

The special form `if` only evaluates the alternative expression if the predicate doesn't evaluate to true.

### Exercise 1.7: Improving The Iterative Square Root Procedure
_The `good-enough?` test used in computing square roots will not be very effective for finding the square roots of very small numbers. Also, in real computers, arithmetic operations are almost always performed with limited precision. This makes our test inadequate for very large numbers. Explain these statements, with examples showing how the test fails for small and large numbers._

```scheme
; Original sqrt-iter function
(define (average x y) 
  (/ (+ x y) 2))

(define (improve guess x)
  (average guess (/ x guess)))

(define (good-enough? guess x)
  (< (abs (- (square guess) x)) 0.001))

(define (sqrt-iter guess x)
  (if (good-enough? guess x)
      guess
      (sqrt-iter (improve guess x) x)))

(define (sqrt x)
  (sqrt-iter 1.0 x))

; Tests
(good-enough? 0.0005 0.0001)

; Checking for large number failure cases
(sqrt (expt 10 12))     ; This evaluation completes
; (sqrt (expt 10 13))   ; This evaluation doesn't complete
```

`good-enough?` stops once the square of the guess is less than 0.001 away from the desired result. This will cause it to always evaluate to true for small values.

_An alternative strategy for implementing `good-enough?` is to watch how guess changes from one iteration to the next and to stop when the change is a very small fraction of the guess. Design a square-root procedure that uses this kind of end test. Does this work better for small and large numbers?_

```scheme
; Original sqrt-iter function
(define (average x y) (/ (+ x y) 2))

(define (improve guess x)
  (average guess (/ x guess)))

(define (good-enough? guess prev-guess)
  (< (abs (- guess prev-guess)) (/ prev-guess 100)))

(define (sqrt-iter guess prev-guess x)
  (if (good-enough? guess prev-guess)
      guess
      (sqrt-iter (improve guess x) guess x)))

(define (sqrt x)
  (sqrt-iter 1.0 0 x))

; Tests
(sqrt 9)
(sqrt 0.0001)
(sqrt (expt 10 14))
```

### Exercise 1.8: Newton's Method for Cube Roots 
_Newton's method for cube roots is based on the fact that if y is an approximation to the cube root of x, then a better approximation is given by the expression below. Use this formula to implement a cube-root procedure analogous to the square-root procedure_

Let y be an approximation of the cube root of x

$$x^\frac{1}{3} = \frac{x / y^2 + 2y}{3}$$

```scheme
(define (improve guess x)
  (/ (+ (/ x
          (* guess guess)
        )
        (* 2 guess)
     )
     3
  )
)

(define (good-enough? guess prev-guess)
  (< (abs (- guess prev-guess)) (/ prev-guess 100)))

(define (sqrt-iter guess prev-guess x)
  (if (good-enough? guess prev-guess)
      guess
      (sqrt-iter (improve guess x) guess x)))

(define (cbrt x)
  (sqrt-iter 1.0 0 x))

; Tests
(cbrt 0.0005)       ; Returns ~0.079
(cbrt 512)          ; Returns ~8.00 
(cbrt (expt 10 13)) ; Returns ~21545.34
```
