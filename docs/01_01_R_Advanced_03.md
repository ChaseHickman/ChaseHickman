

### Subsetting{#arsub}

#### Selecting multiple elements

To select any number of elements from a vector use `[` *(see ``` ?`[` ``` for help)*.

##### Atomic Vectors

Atomic vectors can be subset in six ways:

1. *Positive integers* return elements at that position



```r
x <- 11:20

x[1]
## [1] 11
x[1:3]
## [1] 11 12 13

# repeating element positions returns multiple elements
x[c(1,1,10,10)]
## [1] 11 11 20 20
# real numbers are converted to integer
x[c(1.1,2.2,3.3)]
## [1] 11 12 13

# order() returns integers for elements of vector x arranged in ascending or decending order
order(x, decreasing = TRUE)
##  [1] 10  9  8  7  6  5  4  3  2  1
# pass that integer vector into `[` to select the reordered elements of x
x[order(x, decreasing = TRUE)]
##  [1] 20 19 18 17 16 15 14 13 12 11
```

2. *Negative integers* exclude elements at that position


```r
x[-c(1,3,5,7,9)]
## [1] 12 14 16 18 20

# you can't mix positive and negative integers
x[c(1,-2)]
## Error in x[c(1, -2)]: only 0's may be mixed with negative subscripts
```

3. *Logical vectors* include or excluded elements based on values `TRUE`/`FALSE`. This allows 
you to pass a logical test into `[`:


```r
# a logical test produces a logical vector
x < 15
##  [1]  TRUE  TRUE  TRUE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE

# this logical vector can subset elements based on TRUE/FALSE
x[x<15]
## [1] 11 12 13 14

# recylcing rules apply if the vector is an integer multiple of length(x)
x[c(TRUE, FALSE)]
## [1] 11 13 15 17 19

# NA will always return NA in the results
x[c(TRUE, NA)]
##  [1] 11 NA 13 NA 15 NA 17 NA 19 NA
```

4. *Nothing* returns the vector as is. This is more useful for 2d structures and multi-dim arrays.


```r
x[]
##  [1] 11 12 13 14 15 16 17 18 19 20
```

5. *Zero* returns a zero-length vector.


```r
x[0]
## integer(0)
```

6. *Character vectors* can be used to select vectors with attribute `names`. This does **not** work to factors and only
selects exact matches.


```r
x <- setNames(x, c(letters[11:20]))

x[c('m','o','o','n')]
##  m  o  o  n 
## 13 15 15 14
```

##### Lists

Use `[` to subset lists exactly as atomic vectors, only `[` will ever return a list. To
extract elements out of a list you need `[[` or `$` per below.

##### Matrices and arrays

Subsetting 2d matrices and >2d arrays with `[` can be done with:

1. multiple vectors *(delimited with `,`)*



```r
m <- matrix(data = 1:9, nrow = 3)
m
##      [,1] [,2] [,3]
## [1,]    1    4    7
## [2,]    2    5    8
## [3,]    3    6    9

# pass a vector for each dimension, the first two rows and not the third column
m[c(1:2),-3]
##      [,1] [,2]
## [1,]    1    4
## [2,]    2    5

# empty vectors are useful now to select all elements of a dimension
# select all rows and not the third column
m[, -3]
##      [,1] [,2]
## [1,]    1    4
## [2,]    2    5
## [3,]    3    6
```

2. single vectors *(not delimited with `,`)*


```r
# matrices and arrays are just vectors with class "dim", so passing single vectors is valid
m[1:3]
## [1] 1 2 3

m[c(T,T,T,F,F,F,T,T,T)]
## [1] 1 2 3 7 8 9
```
R will simplify to the lowest dimensionality possible, so passing a vector returns a vector. Passing 
two dimensions returns a matrix etc.

3. matrix


```r
# a matrix of (row, col) dimensions to select the diagonal in matrix m
m_select <- matrix(data = c(1,1,2,2,3,3), nrow = 3, byrow = T)
m_select
##      [,1] [,2]
## [1,]    1    1
## [2,]    2    2
## [3,]    3    3

# matrix m 
m
##      [,1] [,2] [,3]
## [1,]    1    4    7
## [2,]    2    5    8
## [3,]    3    6    9
# pass the matrix to subset the diagonal of m 
m[m_select]
## [1] 1 5 9
```

##### Data frames and tibbles

Data frames can be subset with `[` also, but they behave like lists when passing
a vector, and like a matrix when passing multiple vectors with `,`.


```r
# a data frame
df <- data.frame(x = 1:5, y = letters[1:5], z = LETTERS[1:5],
                 stringsAsFactors = FALSE)
df
##   x y z
## 1 1 a A
## 2 2 b B
## 3 3 c C
## 4 4 d D
## 5 5 e E

# passing a vector returns whole columns
df[c("x","x","z")]
##   x x.1 z
## 1 1   1 A
## 2 2   2 B
## 3 3   3 C
## 4 4   4 D
## 5 5   5 E

# passing vector pairs for each dimension subsets like a matrix
df[c(1,5), ]
##   x y z
## 1 1 a A
## 5 5 e E
```

We saw earlier that matrix subsetting will try and simplify vectors when possible. This has 
consequences for data frames when subsetting single columns like a matrix:


```r
# passing vector pairs may try and simplify vectors (in this case to an integer vector)
str(df[,"x"])
##  int [1:5] 1 2 3 4 5
```

To preserve dimensionality use `drop = FALSE`:


```r
str(df[,"x", drop = FALSE])
## 'data.frame':	5 obs. of  1 variable:
##  $ x: int  1 2 3 4 5
```


Tibbles will always return tibbles:


```r
str(tibble::as_tibble(df)[,"x"])
## Classes 'tbl_df', 'tbl' and 'data.frame':	5 obs. of  1 variable:
##  $ x: int  1 2 3 4 5
```

#### Selecting a single element

`[[` can be used for subsetting single elements, while `x$y` can be used similar to `x[["y"]]`.

##### [[

Recall `[` returns a list element as a list. So to access the value within that list you write `[[`:



```r
l <- list(1, matrix(1:9,3), c(TRUE,FALSE,TRUE))

# [ returns a list
str(l[3])
## List of 1
##  $ : logi [1:3] TRUE FALSE TRUE
# [[ returns the object in that list
l[[3]]
## [1]  TRUE FALSE  TRUE
```

##### $

`$` is a shortcut for something similar to `x$y` == `x[["y"]]` and is often used
to extract columns of data frames:


```r
df <- data.frame(x = 1:5, yolk=letters[1:5], zoo = LETTERS[1:5])
df$x
## [1] 1 2 3 4 5
```

However, there is a common misuse: $ does partial matching from left-to-right. 

```r
df$y # returns column name "yolk"
## [1] a b c d e
## Levels: a b c d e
```

You can use `options(warnPartialMatchDollar = TRUE)` to warn you when it happens:


```r
options(warnPartialMatchDollar=TRUE)

df$y
## Warning in df$y: partial match of 'y' to 'yolk'
## [1] a b c d e
## Levels: a b c d e
```

Tibbles never do partial matching.

#### Subassignment

Subsetting can be combined with assignment to update values at those indices. This
is called subassignment and takes the general form `x[i] <- value`.




```r
x <- 1:10

# update elements 1:5 and flip their sign
x[c(1:5)] <- c(1:5)*-1
x
##  [1] -1 -2 -3 -4 -5  6  7  8  9 10
```

__R will recycle subassignments__ if the subset and assgnment vectors are different lengths, so be
sure of both *length* and *duplicate values*.


```r
# R recyles the scalar to both the first and last elements
x[c(1,length(x))] <- 0
x
##  [1]  0 -2 -3 -4 -5  6  7  8  9  0
```

List elements can be removed with NULL subassignment:


```r
rm(l)

# Create list l where element 3 is a data frame
l <- list(1:3, 1, data.frame(x = 1:12))
# subassign the 3rd element to NULL
l[[3]] <- NULL

# the data frame is removed from list l
str(l)
## List of 2
##  $ : int [1:3] 1 2 3
##  $ : num 1
```

Empty subsets can be useful to preserve the original object class:


```r
rm(df)

df <- data.frame(x=c(1,1,0,0))

# modify the empty subset of the data frame rather than the data frame object
# i.e. modify the contents rather than the object
df[] <- lapply(df, as.logical)

# df retains data.frame class
str(df)
## 'data.frame':	4 obs. of  1 variable:
##  $ x: logi  TRUE TRUE FALSE FALSE

# whereas list apply updates df object to class list
df <- lapply(df, as.integer)
str(df)
## List of 1
##  $ x: int [1:4] 1 1 0 0
```

#### Applications

##### Lookup tables

Translating lookup values using subsetting:



```r
# a character vector of test scores
scores <- c('l','l','h','m','h','m','l')

# a lookup vector of descriptors for named scores
xlat_scores <- c(l = 'low',m = 'medium', h='high')

# subset xlat_scores with the observed scores
xlat_scores[scores]
##        l        l        h        m        h        m        l 
##    "low"    "low"   "high" "medium"   "high" "medium"    "low"

# or unname() to remove attribute "names"
unname(xlat_scores[scores])
## [1] "low"    "low"    "high"   "medium" "high"   "medium" "low"
```

##### Matching and merging by integer subsetting

To merge multiple columns from a lookup table manually you can use `match()`, 
which matches elements of `x` in `table` (see `?match()`) and returns the integer
position in `table`:


```r
pets <- c(1,2,1,3)

pet_xlat <- data.frame(petid = c(3,2,1), 
                       pet.type = c('cat','dog','bird'), 
                       pet.aka = c('kitty','doggie','birdie'))

pet_match <- match(x = pets,
                   table = pet_xlat$petid)

# the matching integer position of pets in pet_xlat
pet_match
## [1] 3 2 3 1

# subset the rows matched by pet_match
pet_xlat[pet_match,]
##     petid pet.type pet.aka
## 3       1     bird  birdie
## 2       2      dog  doggie
## 3.1     1     bird  birdie
## 1       3      cat   kitty

rm(pets)
rm(pet_xlat)
rm(pet_match)
```

To merge on more than one column you would typically use `interaction()` to collapse
into one and perform a similar operation. However, you should probably look to custom
built merge functions like `merge()` or `dplyr::left_join()` to ease the process. 

##### Random and Bootstrap sampling

Data frame subsetting makes it easy to create random samples and bootstrap samples in R. `sample()` returns elements of a vector (or creates an integer vector 1:`n` if given a scalar) with arguments for `size=` and 
`replace=`.


```r
rm(df)

df <- data.frame(x = 1:5, y = letters[1:5], z = LETTERS[1:5],
                 stringsAsFactors = FALSE)

set.seed(1)
# use integer indices from sample to return two randomly sampled rows from df without replacement
df[sample(5, size = 2),]
##   x y z
## 1 1 a A
## 4 4 d D

# use sample() to randomly sample an integer vector 1:length(df) with replacement 5 times 
# and subset the corresponding rows of df
set.seed(1)
df[sample(x = length(df), size = 5, replace = TRUE), ]
##     x y z
## 1   1 a A
## 3   3 c C
## 1.1 1 a A
## 2   2 b B
## 1.2 1 a A
# row 1 was selected three times (rownames: 1, 1.1, 1.2), and rows 2 & 3 only once in a bootstrap sample

rm(df)
```

##### Ordering{#arordering}

Use `order()` to return an `integer` vector describing the sorted elements of `x` or `...`, which
can be used to subset vectors in a new order:


```r
x <- letters[10:1]
x
##  [1] "j" "i" "h" "g" "f" "e" "d" "c" "b" "a"

# order() returns the integer positions of x sorted ascending by default
order(x)
##  [1] 10  9  8  7  6  5  4  3  2  1

# use this new vector to reorder the original vector
x[order(x)]
##  [1] "a" "b" "c" "d" "e" "f" "g" "h" "i" "j"

rm(x)
```

There are additional arguments to `order()` such as `decreasing` and `na.last` that 
allow you to sort descending or `NA` values first, last, or ommit them.

For >2d vectors, `order()` can be passed to each dimension allowing you to sort columns and 
rows independently:


```r
df <- data.frame(Alpha = 1:6, Bravo = TRUE, Charlie = c('red','blue',NA))
df
##   Alpha Bravo Charlie
## 1     1  TRUE     red
## 2     2  TRUE    blue
## 3     3  TRUE    <NA>
## 4     4  TRUE     red
## 5     5  TRUE    blue
## 6     6  TRUE    <NA>

# subset df by sorting rows by Charlie and removing NA; order columns descending by name
df[order(df$Charlie, na.last = NA), order(names(df), decreasing = T)]
##   Charlie Bravo Alpha
## 2    blue  TRUE     2
## 5    blue  TRUE     5
## 1     red  TRUE     1
## 4     red  TRUE     4
```

##### Random sort

As an extension of [ordering](#arordering), use `sample()` to randomly reorder rows 
of a data frame:


```r
set.seed(8)
# randomly sample integers from 1:"rows in df" "rows in df" times without replacement 
# and subset row dim
df[sample(nrow(df), nrow(df), replace = FALSE), ]
##   Alpha Bravo Charlie
## 4     4  TRUE     red
## 2     2  TRUE    blue
## 3     3  TRUE    <NA>
## 6     6  TRUE    <NA>
## 5     5  TRUE    blue
## 1     1  TRUE     red

rm(df)
```

##### Expanding aggregate counts

`rep()` replicates elements of `x` a number of `times`, where `times` can be a vector
for each element, or a scalar to recyle the entire vector `x`.

`rep()` also can accept argument `each` to repeat each element `each` number of times instead of 
recycling the entire vector; or `length.out` to recyle the vector the desired length *(even if not an integer multiple)*.


```r
df <- data.frame(crayon_color = c('red','blue','yellow'),
                 count = 1:3)
df
##   crayon_color count
## 1          red     1
## 2         blue     2
## 3       yellow     3

# rep() accepts an integer vector and vector `times` as the number to repeat each element
rep(x = 1:nrow(df), times = df$count)
## [1] 1 2 2 3 3 3

# used to subset rows in df expands the aggregate counts into observations
df[rep(1:nrow(df), df$count),]
##     crayon_color count
## 1            red     1
## 2           blue     2
## 2.1         blue     2
## 3         yellow     3
## 3.1       yellow     3
## 3.2       yellow     3

rm(df)
```

##### Removing data frame columns

Data frame columns can be removed in two ways, by setting the column to `NULL` or 
by subsetting/subassigning the columns you do want:


```r
df <- mtcars[1:4]

df$mpg <- NULL
str(df)
## 'data.frame':	32 obs. of  3 variables:
##  $ cyl : num  6 6 4 6 8 6 8 4 4 6 ...
##  $ disp: num  160 160 108 258 360 ...
##  $ hp  : num  110 110 93 110 175 105 245 62 95 123 ...

df <- df[c('cyl','hp')]
str(df)
## 'data.frame':	32 obs. of  2 variables:
##  $ cyl: num  6 6 4 6 8 6 8 4 4 6 ...
##  $ hp : num  110 110 93 110 175 105 245 62 95 123 ...

rm(df)
```

You can use set operations to select all except the columns you don't want: 


```r
df <- mtcars[1:4]

yuck <- c("cyl","disp")

# setdiff() returns the unique names found in names(df) and not contained in set yuck
head(df[setdiff(names(df),yuck)])
##                    mpg  hp
## Mazda RX4         21.0 110
## Mazda RX4 Wag     21.0 110
## Datsun 710        22.8  93
## Hornet 4 Drive    21.4 110
## Hornet Sportabout 18.7 175
## Valiant           18.1 105

rm(df)
```

##### Logical subsetting

The most common subsetting action for data frames is passing a logical vector
to the row dimension of `[`. This allows for multiple conditions accross column vectors
using `(` grouping for clarity, and vector boolean operators `&` (and), `|` (or), 
`!` (not), `%in%` (in), etc.


```r
# select cars that get over 20 mpg, are not 4 cylinder, and are not the Honda Civic 
# or Hornet 4 Drive
mtcars[(mtcars$mpg > 20) & 
         (mtcars$cyl != 4) & 
         !(rownames(mtcars) %in% c('Honda Civic',
                                   'Hornet 4 Drive')),]
##               mpg cyl disp  hp drat    wt  qsec vs am gear carb
## Mazda RX4      21   6  160 110  3.9 2.620 16.46  0  1    4    4
## Mazda RX4 Wag  21   6  160 110  3.9 2.875 17.02  0  1    4    4
```

##### Boolean algebra and sets

Logical subsetting (boolean) and integer subsetting (sets) are naturally 
equivocal. However, integer sets may have an advantage when you specifically need the location 
of the first or last `TRUE` element of a set, if the logical set contains a high ratio 
of `FALSE`:`TRUE` (you may save on memory), or if you specifically want the results to drop
`NA`s.

You can use `which()` to convert a logical set to an integer set:

```r
# a logical set
l <- c(FALSE, TRUE, FALSE, FALSE, TRUE, TRUE, TRUE, NA)

# an integer set--which() gives the TRUE indices of a logical vector and *drops all NA*
i <- which(l)
i
## [1] 2 5 6 7
```

You might convert a logical to an integer set to easily find the first or last `TRUE` elements:

```r
# first TRUE
i[1]
## [1] 2
# last TRUE
i[length(i)]
## [1] 7

rm(l); rm(i)
```

**Boolean compared to set oprations**


```r
# set x; create logical and integer versions
x1 <- c(TRUE, FALSE, TRUE, FALSE, TRUE, FALSE)
x2 <- which(x1); x2
## [1] 1 3 5

# set y; create logical and integer versions
y1 <- c(TRUE, FALSE, FALSE, TRUE, FALSE, FALSE)
y2 <- which(y1); y2
## [1] 1 4
```

*Intersection:* `&` $\Leftrightarrow$ `intersect()`


```r
# both must be TRUE
x1 & y1
## [1]  TRUE FALSE FALSE FALSE FALSE FALSE

intersect(x2, y2)
## [1] 1
```

*Union:* `|` $\Leftrightarrow$ `union()`


```r
# either can be TRUE
x1 | y1
## [1]  TRUE FALSE  TRUE  TRUE  TRUE FALSE

union(x2, y2)
## [1] 1 3 5 4
```

*Complement:* `& !` $\Leftrightarrow$ `setdiff()`


```r
# exclusive TRUE in x1
x1 & !y1
## [1] FALSE FALSE  TRUE FALSE  TRUE FALSE

setdiff(x2, y2)
## [1] 3 5
```

*Symmetric difference:* `xor()` $\Leftrightarrow$ `setdiff(union(), intersect())`


```r
# exclusive or (either is TRUE but only one)
xor(x1, y1)
## [1] FALSE FALSE  TRUE  TRUE  TRUE FALSE

setdiff(union(x2, y2), intersect(x2, y2))
## [1] 3 5 4
```

