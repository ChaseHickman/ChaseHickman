
### Control Flow{#arconflow}

Control flows in R can be split into *choices* and *loops*. Technically, *conditions* like messages, 
warnings, and erros offer a non-local form a control flow (see [conditions](https://adv-r.hadley.nz/conditions.html)).

#### Choices

`if` takes the basic form of: 

```r
if (condition) true_action
if (condition) true_action else false_action
```
It returns the value of true_action or false_action; and can be compounded within `{` to 
peform multiple tests:


```r
x <- 2

# if (condition) evaluates a scalar
if (x == 1) "single" else "multiple"
## [1] "multiple"

# compound if statments takes the form:
if (x == 1){
    "single"
  }else if(x == 2){
      "double"
  }else{
      "multiple"
  }
## [1] "double"
```

`if` will silently return `NULL` when no else statement is given:


```r
presence <- TRUE

if (!presence) "Elvis has left the building."
```

##### Vectorized `if`

`if` only works with a single `TRUE`/`FALSE`. You can use `ifelse()` to vectorize `if`:


```r
x <- 1:5

ifelse(test = x %in% c(1,2), yes = 0, no = x)
## [1] 0 0 3 4 5
```

To vectorize compound `if` statements use `dplyr::case_when()` *(note the syntax change)*:


```r
set.seed(1)
test_scores <- rnorm(n = 5, mean = 82, sd = 10)

# case_when() uses (conditional) ~ result syntax; TRUE ~ "F" is equivelant to else{}
dplyr::case_when(
  test_scores > 90 ~ "A",
  test_scores > 80 ~ "B",
  test_scores > 70 ~ "C",
  test_scores > 60 ~ "D",
  TRUE ~ "F"
)
## [1] "C" "B" "C" "A" "B"
```

##### `switch()` statement

`switch()` statements give you a more succint form of compound `if{}`
syntax:


```r
greeting <- function(person){
  switch(person,
         swab = ,
         captain = ,
         pirate = ,
         privateer = ,
         buccaneer = ,
         sailor = "Ahoy",
         stop("It's mutiny!")
         )
}

greeting(person = "privateer")
## [1] "Ahoy"
greeting(person = "taxidermist")
## Error in greeting(person = "taxidermist"): It's mutiny!
```

1. when no value is defined `switch()` will "fall through" to the next defined 
value (e.g. "Ahoy")

2. the last argument uses `stop()` to throw an error; else `switch()` will return `NULL` and fail silently

`switch()` can throw undesirable errors when used with `numeric`, so it's 
advisable to only use it with type `character`.

#### Loops

*note: loops are generally not needed for data anlysis tasks as there are functionals (e.g. `map()` and `apply()`) which vectorize them for you.*

`for` is an iterator which takes each element of a vector until either there are no
elements left, or `next`/`break` is encountered. It takes the form:

```r
for (item in vector) perform_action

# or can be extended to compound statements similar to `if`:

for (item in vector){
  action_1
  action_2
  etc.
}
```

`i` in this example is a binding within `{}` environment of the current vector element, and will
be overwritten each iteration:


```r
for (i in letters[1:3]) print(i)
## [1] "a"
## [1] "b"
## [1] "c"

for (i in letters[1:10]){
  if (i == "b")
    next
  
  if (i == "d")
    break
  
  print(i)
}
## [1] "a"
## [1] "c"
```

1. *Why your `for` loop is slow:*

Because of dynamic memory allocation, it is better in R to pre-allocate vector containers so 
the program is not repeatedly asking for more and more memeory as the object grows.

It also minimizes the number of object copies required.

`vector()` can be used to create the vector container of the right type and length:


```r
# create an integer vector the same length as our loop
v <- vector(mode = "integer", length = 10)
v
##  [1] 0 0 0 0 0 0 0 0 0 0

# R has already allocated space for vector v, so updating each element with `for`
# does 
for (i in 1:10){
  v[i] <- i
}

v
##  [1]  1  2  3  4  5  6  7  8  9 10
```

2. *`seq_along()` instead of `1:length()`*

It is better to generate a regular sequence with `seq_along()` because it will always 
generate a `numeric` sequence the same length as the input vector.

`1:length(x)` can fail when vector length is 0 because `:` will actually create vector `[1] 1 0` (since 
`:` creates descending vectors as well).


```r
v <- c()

# error here when accessing vector position 0, which, in R, doesn't exist
for (i in 1:length(v)){
  v[[i]] <- i
}
## Error in v[[i]] <- i: attempt to select less than one element in integerOneIndex
```

3. *dropping S3 object attributes*

It is easy to drop object attributes when iterating, as `for` will loop the underlying, in this 
case, `double` values:


```r
v <- as.Date(x = c('2020-04-01','2020-04-02'))

for (i in v){
  print(i)
}
## [1] 18353
## [1] 18354
```

You can use `seq_along()` and access the element directly using `[[` to avoid droping attributes:


```r
v <- as.Date(x = c('2020-04-01','2020-04-02'))

for (i in seq_along(v)){
  print(v[[i]])
}
## [1] "2020-04-01"
## [1] "2020-04-02"
```

##### Other loops

Other loops in R:

* `while (condition) do_action`: this is more flexible than loops because you don't have to know the 
length of the vector apriori.
* `repeat(do_action)`: where do_action repeats forever until it encounters `break`
