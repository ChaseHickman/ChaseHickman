
### Functions{#arfunc}

Functions are first-class objects in R you create with `function()`. They can be bound to a name or
left anonymous; and have three components:

#### Function Components

1. Arguments: `formals()` are arguments passed to the function that determine how it is called
2. Body: `body()` the code being called
3. Environment: `environment()` a data structure that determines how R finds bound values within the function

Arguments and body are always specified explicitly in the function, and environment is implicitly specified based 
on *where* you define the function.


```r

f <- function(x, y, z){
  # comment within the function
  x + y + z
}

# to see the arguments
formals(f)
## $x
## 
## 
## $y
## 
## 
## $z

# to see the body
body(f)
## {
##     x + y + z
## }

# to see the environment
environment(f)
## <environment: R_GlobalEnv>
```

Because functions are objects, they can have attributes just like vectors. Base R functions have 
attribute `srcref`, which points to the source code of the function. It is more compact for printing 
and includes things like comments. Use `attr()`, or, simply, the name of the function, i.e. without calling the function using `(`:


```r
{
  attr(f, which = 'srcref')
  f
  }
## function(x, y, z){
##   # comment within the function
##   x + y + z
## }
```

#### Primitives

Some base R functions like `sum()`/`[` are "primitive functions", meaning they are written in C for 
speed:


```r
`[`
## .Primitive("[")

sum
## function (..., na.rm = FALSE)  .Primitive("sum")
```

#### Anonymous functions and Closures

Functions without binding names are called anonymous functions:


```r
# lapply calls a function on each element of a list. In this case each element is a 
# vector in a data frame. The anonymous function uses control flow to sum each vector
# if it is numeric.
lapply(iris, function(x) if (is.numeric(x)) sum(x))
## $Sepal.Length
## [1] 876.5
## 
## $Sepal.Width
## [1] 458.6
## 
## $Petal.Length
## [1] 563.7
## 
## $Petal.Width
## [1] 179.9
## 
## $Species
## NULL
```

In R you will often see functions referred to as **closures** because they enclose 
their environment. e.g. a binding within the function environment is not available 
globally.

#### Invoking functions

Functions are invoked by enclosing arguments withing `(` following the function
binding (e.g. `sum(x)`).

However, if you have the function arguments in a list you can envoke the function
using `do.call()`:


```r
do.call(what = rep, args = list(x = 1:3, each = 2))
## [1] 1 1 2 2 3 3
```

#### Function Composition

There are three ways to compose function transformations:

1. Nesting: `function_b(function_a(x, y), z)`

Functions are nested and the result of the inner function informs an 
argument of the outer. This can be difficult to read, inside-out, right-to-left, 
and can spread agruments away from the name being called.

2. Intermediate objects: `{result <- function_a(x, y); result <- function_b(result, z)}`

With intermediate objects the function results are saved to a common binding. This is more 
verbose and a weakness when the results are truly temporary.

3. Pipes: `x %>% function_a(y) %>% function_b(z)`

Pipes are a way to pass objects into the first argument of a function. `x %>% f()` is 
equivelant to `f(x)`, so `x %>% f(y)` is equivelant to `f(x, y)`. This is analogous 
to treating an object as a noun and functions as verbs, allowing you to write linear 
chains left-to-right, similar to F# and Haskell. Data analysis workflows are often 
multiple linear transformations of the same object, e.g. a dataframe.

#### Lexical scoping

**Scoping** is how a program finds the value associated with a name. R uses 
_*lexical*_ scoping, which means the scoping rules are parse-time^[When R evaluates 
the syntax of the written code.] rather than run-time. 

Lexical scoping follows the following four rules:

##### Name masking

With lexical scoping names defined within a function mask values found outside of 
that environment. R will look one level up to find the value if not defined in the 
environment. One level up could mean an outer function, the global environment, or,
finally, any loaded packages.


```r
# bind x, y, z in the global environment
x <- 1; y <- 2; z <- 3

f <- function(){
  # bind y, z within the environment for f
  y <- 20; z <- 
    
  f2 <- function(){
    # bind z within the environment for f2
    z <- 300
    
    # return a vector containing each variable
    c(x, y, z)
  }
  
  # return f2
  f2()
}

f()
## [1]   1  20 300
```

##### Functions vs variables

Just as with variables, functions can mask other functions:


```r
# bind f in the global environment
f <- function() 1

f2 <- function(){
  # bind f in the funciton environment
  f <- function() 100
  
  # return f
  f()
}

f2()
## [1] 100
```

It is possible, in two different environments, for a variable and function to share 
the same name. How R scopes these values becomes more complex, because, in a function 
call (e.g.`f()`) R will ignore non-function objects when scoping that name. So it is 
possible for a name to represent more than one value. This is confusing and is best 
avoided.


```r
# bind f a function in the global environment
f <- function(x) x

f2 <- function(){
  # bind f a scalar
  f <- 100
  
  # f represents both a variable and a function when calling f() because f() specifically 
  # scopes for function objects only
  f(f)
}

f2()
## [1] 100
```

##### Variable lifespan

Objects defined wihtin the environment of a function only exist while the 
function is invoked:


```r
# create a function either to instantiate a scalar integer or add to it
f <- function(){
  if (!exists("a")){
    a <- 1
  }else{
    a <- a + 1
  }
  
  a
}

# both function calls return 1 because environment variables in the function are 
# rewritten at each function call
f()
## [1] 1
f()
## [1] 1
```

##### Dynamic lookup

Lexical scoping determines where, but not when, a value is retrieved. This can 
change the value a function returns because R does not scope the object until 
it is called:


```r
f <- function(){
  x * 2
}

x <- 1

f()
## [1] 2

x <- 2

f()
## [1] 4
```

`codetools::findGlobals()` can be used to find names in a function that are 
externally unbound.

R uses lexical scoping to find *everything*, including objects that may not be 
obvious like `+` and `[`.

#### Lazy evaluation

R does not compute an expression until it is needed. This is called *lazy evaluation*:


```r
f <- function(x){
  message("The function doesn't stop because x is not used in f().")
}

f(stop('Let me outa here!'))
## The function doesn't stop because x is not used in f().
```

##### Promises

Lazy evaluation is powered by a data structure called a promise, which has 
three components: the expression (e.g. `x + y`), an environment, and a value 
(which is cached and calculated only once).

##### Default arguments

You can use lazy evaluation to pass default arguments or reference variables 
later defined in the function. This is common in base R. However, this can be 
confusing because 
*variables in the function call are otherwise scoped outside the function environment*.

For example, `a` and `b` are not scoped from the global environment (`1e6`) because we reference 
them as variables in the __default argument__ `z`. Also, default arguments can reference 
other default arguments (see `x`), even if later defined in the function (`a` and `b`):


```r
# a & b are defined in the global environment
a <- 1e6
b <- 1e6

# use = to define the default argument
# default arguments can reference other arguments, even if defined later in the function
f <- function(x = 1, y = 1 + x, z = a * b){
  a <- 3
  b <- 1
  
  c(x, y, z)
}

f()
## [1] 1 2 3
```

##### Missing arguments

`missing()` can tell you if an argument was given by the user or was default:


```r

f <- function(x = 1){
  missing(x)
}

# TRUE; x was missing and the default value used
f()
## [1] TRUE
# FALSE; x was given to the function call
f(1)
## [1] FALSE
```

Read `?missing()` carefully because there are a lot of "gotchas" with `missing()` 
in base R. For example, read `sample`--`sample()` will assign a value for argument 
`size` even though it has no default argument.

###### `%||%` infix function

This common pattern, `if (missing(x)) x <- object`, in base R functions can be simplified by the `%||%` infix function from rlang. Instead of pretending an argument is required, set the default argument 
to `NULL`:


```r
library(rlang)
## Warning: package 'rlang' was built under R version 3.6.3

# %||% is relatively simple; if left side is NULL return right side else return left side
function (x, y) 
{
    if (is_null(x)) 
        y
    else x
}
## function (x, y) 
## {
##     if (is_null(x)) 
##         y
##     else x
## }

# in this example, it's explicit x is optional and control flow is concise if NULL
f <- function(x = NULL){
  x %||% 7
}

f()
## [1] 7
```

#### `...` (dot-dot-dot)

`...` in other languages is often called *varargs* (variable arguments). In R it's
called "dots". It allows functions to take any number of additional arguments:


```r
# f only takes 3 arguments, so calling with 4 returns an error
f <- function(x, y, z){
  c(x,y,z)
}

f(1,2,3,4)
## Error in f(1, 2, 3, 4): unused argument (4)

# ... allows you to pass more arguments
f <- function(x, y, z, ...){
  c(x,y,z)
}

f(1,2,3,4)
## [1] 1 2 3
```

When creating a [functional](#arfunctionals) and passing dots use `...` in the sub-functions
requiring them:


```r
# create functional to return aggregate of random samples from a distribution (rnorm, runif, rt etc)
functional <- function(f, distr, ... ){
  f(distr(...))
}

functional(f = sum, 
           distr = rnorm, 
           n = 100, 
           mean = 7, 
           sd = 2)
## [1] 678.5773
```


The primary use of `...` is to pass additional arguments to other functions or methods. For example, 
`lapply()` applys a function over each element of a list. In this case, I'm using the `...` 
argument of `lapply()` to pass the `trim` argument to `mean()`:


```r
lapply(X = mtcars[1:2], FUN = mean, trim = .25)
## $mpg
## [1] 19.175
## 
## $cyl
## [1] 6.375
```

It is possible to use `..N` to specify the position of the additional arguments:


```r
# this function returns the first three additional arguments in reverse order
f <-function(...){
  c(..3, ..2, ..1)
}

f(1,2,3)
## [1] 3 2 1
```

However, it may be more useful to store them in a list:


```r
f <-function(...){
  list(...)
}

# a list of the additional arguments (with name if defined)
str(f(arg1 = 1, arg2 = 2, 3))
## List of 3
##  $ arg1: num 1
##  $ arg2: num 2
##  $     : num 3
```

Beware that misspelled arguments does not raise an error:


```r
# mean argument trim is mispelled and therfor silently dropped
lapply(mtcars["mpg"], mean, trimm = .25)
## $mpg
## [1] 20.09062
```

#### Exiting a function

Most functions exit by either returning a value or throwing an error.

##### Explicit vs implicit

R will implicitly return the last value encountered, or you can explicitly 
define the return with `return()`:


```r
f <- function(x){
  x*2
  x
}

f(1)
## [1] 1

f <- function(x){
  return(x*2)
  x
}

f(1)
## [1] 2
```

##### Visible vs invisible

Functions return values visibly by default (i.e. they get printed); however, you 
can turn this off with `invisible()`. The value still is return, just not printed:


```r
f <- function(x) invisible(x)

f(2)

# enclosing with () forces print; showing the value is indeed returned
(f(2))
## [1] 2
```

`<-`, `plot()`, `print()` or any function that exists primarily for a side-effect, 
the value should be returned invisibly. This is what allows `x <- y <- 2` type 
assignment chaining.

##### Errors

`stop()` should be used to halt and exit a function if it cannot be completed:


```r
f <- function(x){
  if (is.numeric(x)) 
    sum(x) 
  else 
    stop('x must be numeric.')
}

f("banjos")
## Error in f("banjos"): x must be numeric.
```

##### Exit handlers

Sometimes you modify objects in a function which need to be cleaned up whether 
the code executes successfully or not. Use `on.exit(add = TRUE)`:


```r
old_dir <- getwd()
old_dir
## [1] "C:/Users/hickmancr/Desktop/ChaseHickman"

f <- function(x, ...){
  if (x){
    # set working directory to one level up
    setwd('..')
    # reset the original working directory whether the function executes or not
    on.exit(setwd(old_dir), add = TRUE)
    
    getwd()
  }else{
    stop("You're not my supervisor!")
  }
}

f(FALSE)
## Error in f(FALSE): You're not my supervisor!

# on.exit() cleaned up the working directory even though the function threw an error
getwd()
## [1] "C:/Users/hickmancr/Desktop/ChaseHickman"
```

`on.exit()` can be called from anywhere in the function, so it's helpful to place it 
next to the code to be cleaned up.

User `after` argument to help order the execution of `on.exit()`.

**It's important to set `add = TRUE` or each exit handler will overwrite the previous!**

#### Function forms

Every action in R is a function call.

They take four forms:

1. *Prefix*: `function(x, y)` where the name comes before the arguments
2. *Infix*: `x + y` where the name comes between the arguments
3. *Replacement*: `function(x) <-` updates values by assignment
4. *Specials*: `[[`, `for`, `if` etc. have non standard structures

All functions can be written in *prefix* form, however (e.g. `` `+`(x, y)``)

##### Prefix form

`function(x, y)` where the name comes before the arguments.

This is the most common function form in R and beyond. Arguments are mached 
*exact name* > *unique prefix* > *position*:


```r
f <- function(x, yellow, zoo, zoom){
  l <- list(x, yellow, zoo, zoom)
  
  str(l)
}

# error because zo could be partially matching zoo or zoom
f(yel = 2, zo = 3, 4, x = 1)
## Error in f(yel = 2, zo = 3, 4, x = 1): argument 2 matches multiple formal arguments

# R matches unspecified arguments by position
f(yel = 2, 3, 4, x = 1)
## List of 4
##  $ : num 1
##  $ : num 2
##  $ : num 3
##  $ : num 4
```

##### Infix form

`x + y` where the name comes inbetween the arguments, and therefor take two arguments composed 
left to right.

A full list of base [infix functions](#infixfuncs) and their *rank*, which determines the 
order R processes the functions.

You can assign your own infix functions using `%{var}%` form using any characters except `%`. 
Special characters have to be escaped when assigning, but not when calling infix functions.


```r
`%\\%` <- function(x,y) paste0('I don\'t know; ', x + y, '?')

1 %\% 2
## [1] "I don't know; 3?"
```

`+` and `-` are two infix functions that can be called with only one argument.

##### Replacement form

Replacement functions appear to update their value in place, `names() <-`, but 
they actually do create object copies you can see with `tracemem()`.

They must take the general form: `` `function_name<-` <- function(x, value)`` where `x` and `value` 
must be the first and last arguments, with addition arguments inbetween if needed, 
and must return `x`:


```r
# create replacement function to modify even position elements of a vector
`evens<-` <- function(x, value){
  x[1:length(x) %% 2 == 0] <- value
  x
}

v <- 1:5

# could be expressed as evens(x = v, value = 0)
evens(v) <- 0

v
## [1] 1 0 3 0 5
```

##### Specials

Everything else in R fall under the special form, including control flow (`if`,`for`, `break` etc), 
subsetting operators (`[`, `[[`), parentheses and braces (`(`,`{`), `function` etc.

Anything you have questions about refer to ``?`function_name```

