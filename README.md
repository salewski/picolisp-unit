# Unit Testing framework for PicoLisp

[![GitHub release](https://img.shields.io/github/release/aw/picolisp-unit.svg)](https://github.com/aw/picolisp-unit) [![Build Status](https://travis-ci.org/aw/picolisp-unit.svg?branch=master)](https://travis-ci.org/aw/picolisp-unit)

This library can be used for Unit Testing your [PicoLisp](http://picolisp.com/) code.

![picolisp-unit](https://cloud.githubusercontent.com/assets/153401/6709935/073406ae-cd75-11e4-9b62-788356dc0aff.png)

  1. [Requirements](#requirements)
  2. [Getting Started](#getting-started)
  3. [Usage](#usage)
  4. [Examples](#examples)
  5. [Testing](#testing)
  6. [Alternatives](#alternatives)  
  7. [Contributing](#contributing)
  8. [License](#license)

# Requirements

  * PicoLisp 32-bit or 64-bit v3.1.9+

# Getting Started

This is a pure PicoLisp library with no external dependencies. You can either copy `unit.l` into your project(s), or add it as a git submodule to stay up-to-date (recommended).

### Add submodule to your project

  1. git submodule add https://github.com/aw/picolisp-unit vendor/picolisp-unit
  2. Include `unit.l` in your test file
  3. See the [examples](#examples) below

# Usage

Here are some guidelines on how to use `unit.l`, but you're free to poke around the code and use it as you wish.

There exists a few public functions: `(execute)`, `(report)`, and a bunch of `(assert-X)` where X is a type of assertion.

> **Note for 64-bit PicoLisp:** you can use `(symbols 'unit)` (or the prefix: `unit~`).

> **Note for 32-bit PicoLisp:** you don't have access to `(symbols)`, so these functions might clash with existing ones.

  * **(execute arg1 ..)** Executes arg1 to argN tests
    - `arg1` _Quoted List_: a list of assertions, example `'(assert-nil NIL "I AM NIL")`
  * **(report)** Prints a report of failed tests, and exits with 1 if there is a failure
  * **(assert-X)** Various assertions for testing

### Assertions table

Only a few assertions exist for the moment; more [can](#contributing)/will be added.

| Assertion + Arguments | Example |
| --------------------- | ------- |
| (assert-equal Expected Result Message) | `(assert-equal 0 0 "It must be zero")` |
| (assert-nil Result Message) | `(assert-nil NIL "It must be NIL")` |
| (assert-t Result Message) | `(assert-t T "I pity the fool!")` |
| (assert-includes String List Message) | `(assert-includes "abc" '("xyzabcdef") "It includes abc")` |
| (assert-kind-of Type Value Message) | `(assert-equal 'Number 42 "The answer..")` |

### (assert-kind-of) types

There are 5 types currently defined:

* **'Flag** Uses [flg?](http://software-lab.de/doc/refF.html#flg?) to test if the value is `T` or `NIL`.
* **'String** Uses [str?](http://software-lab.de/doc/refS.html#str?) to test if the value is a string.
* **'Number** Uses [num?](http://software-lab.de/doc/refN.html#num?) to test if a value is a number.
* **'List** Uses [lst?](http://software-lab.de/doc/refL.html#lst?) to test if a value is a list.
* **'Atom** Uses [atom](http://software-lab.de/doc/refA.html#atom) to test if a value is an atom.

>> **Warning:** `NIL` is also list, but will be asserted as a `'Flag` when using `(assert-kind-of)`. Use `(assert-nil)` if you specifically want to know if the value is `NIL`.

### Notes

  * Use `(execute)` at the start of your tests (see [examples](#examples))
  * Unit Tests are run in **RANDOM** order.
  * If your tests are **order dependent**, then you can: `(setq *My_tests_are_order_dependent T)`
  * Colours and bold text are only displayed if your terminal supports it, and if your system has the `tput` command.
  * The `(assert-includes)` function uses [sub?](http://software-lab.de/doc/refS.html#sub?) to find a substring in a string or list.

# Examples

It is recommended to create a "test runner" similar to what's found in [test.l](https://github.com/aw/picolisp-unit/blob/master/test.l).

Tests should be placed in a `test/` directory, and the test files should be prefixed with `test_`.

### A simple unit test

```lisp
pil +

(load "unit.l")

(setq *My_tests_are_order_dependent NIL)

[execute
  '(assert-equal 0 0 "(assert-equal) should assert equal values")
  '(assert-nil   NIL "(assert-nil) should assert NIL")
  '(assert-t     T   "(assert-t) should assert T")
  '(assert-includes "abc" '("xyzabcdef")  "(assert-includes) should assert includes")
  '(assert-kind-of  'Flag NIL  "(assert-kind-of) should assert a Flag (NIL)")
  '(assert-kind-of  'Flag T    "(assert-kind-of) should assert a Flag (T)")
  '(assert-kind-of  'String "picolisp"  "(assert-kind-of) should assert a String")
  '(assert-kind-of  'Number 42  "(assert-kind-of) should assert a Number")
  '(assert-kind-of  'List (1 2 3 4)  "(assert-kind-of) should assert a List")
  '(assert-kind-of  'Atom 'atom  "(assert-kind-of) should assert a Atom") ]

# output

  1) ✓  (assert-kind-of) should assert a List
  2) ✓  (assert-kind-of) should assert a Flag (T)
  3) ✓  (assert-includes) should assert includes
  4) ✓  (assert-t) should assert T
  5) ✓  (assert-kind-of) should assert a Number
  6) ✓  (assert-nil) should assert NIL
  7) ✓  (assert-equal) should assert equal values
  8) ✓  (assert-kind-of) should assert a Flag (NIL)
  9) ✓  (assert-kind-of) should assert a Atom
 10) ✓  (assert-kind-of) should assert a String

-> (1 2 3 4 5 6 7 8 9 10)
```

### When tests fail

```lisp
pil +

(load "unit.l")

[execute
  '(assert-equal 0 1 "(assert-equal) should assert equal values")
  '(assert-nil   NIL "(assert-nil) should assert NIL")
  '(assert-t     NIL   "(assert-t) should assert T")
  '(assert-includes "abc" '("hello")  "(assert-includes) should assert includes")
  '(assert-kind-of  'Flag "abc"  "(assert-kind-of) should assert a Flag (NIL)") ]

(report)

# output

  1) ✓  (assert-nil) should assert NIL
  2) ✕  (assert-kind-of) should assert a Flag (NIL)
  3) ✕  (assert-includes) should assert includes
  4) ✕  (assert-t) should assert T
  5) ✕  (assert-equal) should assert equal values

-> ----
1 test passed, 4 tests failed

  Failed tests: 

  - 2)  (assert-kind-of) should assert a Flag (NIL)
        Expected: "abc should be of type Flag"
          Actual: String

  - 3)  (assert-includes) should assert includes
        Expected: "abc in hello"
          Actual: "abc"

  - 4)  (assert-t) should assert T
        Expected: T
          Actual: NIL

  - 5)  (assert-equal) should assert equal values
        Expected: 0
          Actual: 1
```

# Testing

This library has its own set of tests. You can use those are examples as well. To run these types:

    ./test.l

# Alternatives

The alternative approaches to testing in PicoLisp involve the use of [test](http://software-lab.de/doc/refT.html#test) and [assert](http://software-lab.de/doc/refA.html#assert).

# Contributing

If you find any bugs or issues, please [create an issue](https://github.com/aw/picolisp-unit/issues/new).

If you want to improve this library, please make a pull-request.

# License

[MIT License](LICENSE)

Copyright (c) 2015 Alexander Williams, Unscramble <license@unscramble.jp>