# Design Goals
1. Modern feel
   - No archaic syntax or keywords besides the parentheses
   - No over-use of the word 'def' (**def**macro, **def**un, **def**generic, etc.) 
   - Strong text manipulation out of the box for terminal scripting purposes
   - A sane package system
2. Fast
3. JIT compiled with Julia-style method monomorphisation
4. Macros!!
5. Case sensitive
6. Lisp-1
7. Racket-style brace semantics (i.e. `()`, `[]`, and `{}` all mean the same)

## Modern Feel
Perhaps one of the biggest detractors of Lisp (besides the parentheses) is how alien it looks as a language.
There are `defn`s and `car`s and `cadaddr`s and weird `@,foo` strewn everywhere.

For as simple as Lisp is or can be, it sure doesn't look very simple.

It's no secret that I _love_ Lisp as a concept, but every Lisp I try has a little thing about it that just rubs me the wrong way.
* **Common Lisp** is old, archaic, and has decades of features piled on top
  - Its functions and macros have strange names, which were formed before naming conventions were even a thing
  - Its module system&mdash;while existing&mdash; is quite cumbersome to work with
  - Despite being so huge, the language seems to lack basic things like an inverted `format` (i.e. `scanf`) or any decent string parsing tools
* **Scheme** is neat, it's compact, but it feels like you have to re-invent the wheel every time you have to use it
  - Every Scheme implementation is slightly different, and thus it's not portable
  - Macros seem to be done differently in every implementation
  - While having a unified `define` is nice, it's not consistent with `lambda`
* **Racket** is just R6RS done right, but it's still a Scheme
  - Typed Racket's type declarations are... weird
* **Clojure** is weird. It bucks traditions across the board, which makes it feel un-lispy
  - Variable arguments are a vector, despite function definition being a language construct, thus no speed benefit is gained from that vs a list
  - It uses different styles of parentheses for each type of built-in collection, which breaks homoiconicity
  - fields and methods are accessed differently... why?
    * `(.method obj)` vs `(.-field obj)`
  - Its `let` binding doesn't wrap each variable declaration
  - Lots of archaic and non-standard syntax for common operations like scope traversal and class operations

So here's what I plan to do:
- Simple standardised names 
  * `struct` instead of `defstruct`
  * `func` instead of `defn`
  * `new` instead of `make-instance`
  * `val` and `var` instead of `define` or `defvar`
  * etc.
- Typical operator conventions `Foo.bar.baz` rather than `Foo/bar/baz`
- Capitalised struct names; `Foo` instead of `%foo`
- No split between lambdas and functions. Functions are just named lambdas, and should just use the same syntax (more later)
- Strong text manipulation capabilities

## Strong Text Manipulation
As mentioned a few times already, one of my biggest gripes about Lisps is their lack of any good built-in string manipulation facilities.
Having to rely on third-party libraries when wanting to write simple cli scripts isn't the way to go.
It makes your local configuration very un-portable, which makes it harder to deploy or use your scripts on other computers.

The ideal setup is to have a standard library as capable out of the box as that of Perl or Ruby.
Yes, this severely bloats it, but it has the upshot of being _practically_ useful, rather than theoretically so.

I'm considering (though haven't fully decided) having Perl-style variables and variable interpolation, 
so that you can simply write `"Hello, my name is $name"` and not have to bother with extraneous boilerplate.
Maybe even go so far as to copy Perl's `qq` syntax to enable a special `string` form in which you can write unescaped quotes, 
allowing for `(string <span id="example">I said 'Hey!'</span>)` instead of having to do `"<span id=\"example\">I said 'Hey!'</span>"`
or even `'<span id="example">I said \'Hey!\'</span>'`, which is the common problem with "normal" strings.

# Functions
A neat feature in some modern languages is to make functions and closures isomorphic.
That is to say, the syntax for defining a function and a closure is the same.

```lisp
;; this is a closure
(func (x y z) (* z (+ x y)))

;; this is a function
(func foo (x y z) (* z (+ x y)))
```

Note how they only differ in one aspect: the function is named.

## Multimethods
Multimethods are baked in, and types are specified in-line

```lisp
(func foo ((x num) (y num) (z num))
   (* z (+ x y)))
```

If need be, the multimethod's signature can be defined ahead of time

```lisp
(generic foo (x y z)
   (doc "This method adds `x` and `y` and multiplies the result with `z`"))
```

# Structs
Structs are basic collections of data with no notion of hierarchy.

```lisp
(struct Person
  "Documentation goes here"
  (name
   age
   email
   occupation))
```

Structs can also optionally be typed
```lisp
(struct Person
  "Documentation goes here"
  ((name string)
   (age int)
   (email string)
   (occupation string)))
```

The definition of a struct generates default accessors

```lisp
;; getter
(name some-person)

;; setter inspired by Common Lisp's setf macro
(set! (name some-person) "Charles McDonnel")
```

## Accessors
Accessors are just regular functions with one important change, they specify that they're overrides

```lisp
(func email :override ((self Person))
  "This email has been redacted for security reasons")

(func (set! email) :override (new-email (self Person))
  (if (validate-email new-email)
      ;; override can use the original setter
      (set! (email person) new-email)
    (panic! "Invalid email format!!")))
```
