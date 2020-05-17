

### Vectors{#arv}

R has two types of vectors: **atomic** vectors and **list** vectors.

Atomic vectors have values all of the same type, while lists can have values of 
different types.

**NULL** is not a vector, but is often used in R to represent a zero length 
vector. For practical purposes you can consider it a vector.

Vectors can have *attributes*; the most important are **dimension** and **class**.

* dimension: turns a vector into matrices and arrays
* class: critical for S3 objects like factors, dates & date-time, data frames, and tibbles

#### Atomic Vectors {}

##### Types{}

There are four primary types of atomic vectors, and their **scalars** (*i.e. individual value*) can 
be instantiated as:

1. **Logical**: `TRUE`/`T`;`FALSE`/`F`
2. **Double**: decimal `2.345`, scientific `2e10`, hexadecimal `3425h`, infinite `Inf`/`-Inf`, Not a Number `NaN`
3. **Integer**: a non-fractional double followed by `L` (e.g. `7L`, `2e7L`, `3425hL` )
4. **Character**: strings surrounded by single/double quotes; `'Cats'` or `"Dogs"`

Doubles and Integers belong to meta-type **numeric**. `is.numeric()` tests if a vector can be interpreted
as a number: `is.numeric(2.2)` = TRUE = `is.numeric(2L)` = TRUE
 
##### Combine scalars{}

Scalars can be combined to make longer vectors using `c()`:

```r
c(1,2,3)
## [1] 1 2 3
c(T,T,F)
## [1]  TRUE  TRUE FALSE
```

R will flatten atomic vectors when combined with other atomic vectors:

```r
c(1,c(2,c(2.1,2.2,2.3)),4)
## [1] 1.0 2.0 2.1 2.2 2.3 4.0
```

`typeof()` and `length()` will return the type and length of an atomic vector:

```r
x <- TRUE; y <- c(1L, 2L, 3L, 4L)

typeof(x); length(x)
## [1] "logical"
## [1] 1
typeof(y); length(y)
## [1] "integer"
## [1] 4
typeof(NULL); length(NULL)
## [1] "NULL"
## [1] 0
```
 
##### Missing values{}

Missing values are represented by `NA` (Not Applicable). There are logical (`NA`),
integer (`NA_integer_`), double (`NA_real_`), and character (`NA_character_`) types
but R will parse the appropriate `NA` needed.

Many logistical tests in R containing `NA` will return `NA`:


```r
# not false returns true; not NA returns NA
!FALSE; !NA
## [1] TRUE
## [1] NA
```

For this reason, finding missing values in a vector is not obvious:


```r
x <- c(0,NA,NA,3,4)
# a logistical test for NA returns all NA
x==NA
## [1] NA NA NA NA NA
```

Instead, use `is.na()` to test for missing values:


```r
is.na(x)
## [1] FALSE  TRUE  TRUE FALSE FALSE
```

The exception is when all possible values would not change the result:


```r
# this logical statement is true regardless of all possible NA values
NA | TRUE
## [1] TRUE
```

##### Testing and coercing vectors {}

There are many `is.*()` pattern functions in R to test vector type. `is.logical()`, 
`is.integer()`, `is.double()`, and `is.character()` all test for the four primary
atomic vector types:


```r
is.logical(x = c(T,T,F,F))
## [1] TRUE
is.integer(x = c(2.2, 3.0))
## [1] FALSE
```

However, some `is.*()` pattern functions behave unexpectedly. `is.numeric()`, 
for example, returns false for factors, Date, POSIXt and difftime because they 
have their own methods for detection; even though they are numbers and arithmetic
generally makes sense. So be sure to read the documentation carefully (`?is.numeric()`).

---

*coercing types*

It is possible to coerce vectors to a new type, or two combine two types of vectors into
one. The types will be coerced in this order: *character > double > integer > logical*

Use `as.*()` functions to specifically coerce types. R can also implicitly coerce types
when needed. R will give a warning message if it fails or `NA`s are introduced.


```r
# combining a double and a character returns a character vector
str(c(1,"2"))
##  chr [1:2] "1" "2"

# implicitly return the sum of a logical as integer 
str(sum(c(T,T,F)))
##  int 2

# explicitly coerce doubles to characters
str(as.character(c(2e1, 200)))
##  chr [1:2] "20" "200"
```


#### Attributes {}

Other data-structures in R are built on top of these atomic vectors using attributes. 
For example, the **dimension** attribute transforms vectors into matrices and arrays.

##### Getting and setting attributes {}

Attributes are like meta-data you assign in name-value pairs:

* `attr()` to retrieve and set single attributes 
* `attributes()` to retrieve multiple attributes 
* `structure()` to set multiple attributes 


```r
fruit <- 1:10

# create fruit attributes type and days old
attr(x = fruit, which = 'type') <- c('orange','apple','pear')
attr(fruit,'type')
## [1] "orange" "apple"  "pear"

attr(x = fruit, which = 'days old') <- 2
str(attributes(x = fruit))
## List of 2
##  $ type    : chr [1:3] "orange" "apple" "pear"
##  $ days old: num 2

# simultaneously create object and attributes
fruit <- structure(
  1:10,
  type = c('orange','apple','pear'),
  'days old' = 2
)

str(attributes(fruit))
## List of 2
##  $ type    : chr [1:3] "orange" "apple" "pear"
##  $ days old: num 2
```

##### Names{}

`names` is a special and important attribute in R. Although it is not enforced,
names should be unique and complete. They can be set several ways:


```r
# on creation
x <- c(a=1,b=2,c=3)
attr(x, which='names')
## [1] "a" "b" "c"

# assigning a vector of names
x <- 1:3
names(x) <- letters[1:3]
attr(x, which='names')
## [1] "a" "b" "c"

# inline with setNames()
x <- setNames(1:3, letters[1:3])
attr(x, which='names')
## [1] "a" "b" "c"

# with attr()
attr(x, which = 'names') <- letters[1:3]
attr(x, which='names')
## [1] "a" "b" "c"
```

##### Dimensions{}

The `dim` attribute transforms `NULL` dimensional vectors into 2d matrices and multi-dimensional arrays, and can be set by `dim()`, `matrix()`, and `array()`:


```r
x <- 1:6

dim(x) <- c(2,3)
x
##      [,1] [,2] [,3]
## [1,]    1    3    5
## [2,]    2    4    6

matrix(data = 1:6, nrow = 1, ncol = 6)
##      [,1] [,2] [,3] [,4] [,5] [,6]
## [1,]    1    2    3    4    5    6

array(data = 1:6, dim = c(2,3))
##      [,1] [,2] [,3]
## [1,]    1    3    5
## [2,]    2    4    6

# non-idiomatic
attr(x, 'dim') <- c(3,2)
x
##      [,1] [,2]
## [1,]    1    4
## [2,]    2    5
## [3,]    3    6
```

Vector functions usually have matrix or array counterparts:


```r
# vector, matrix, and array bindings
v <- structure(.Data = 1:9, 
               names = letters[1:9])

m <- matrix(data = 1:9, 
            nrow = 3, 
            ncol = 3, 
            dimnames = list(letters[1:3], 
                            letters[1:3]))
                                                            
a <- array(data = 1:9, 
           dim = c(3,3), 
           dimnames = list(letters[1:3], 
                           letters[1:3]))

# Names
names(v)
## [1] "a" "b" "c" "d" "e" "f" "g" "h" "i"
rownames(m); colnames(m)
## [1] "a" "b" "c"
## [1] "a" "b" "c"
dimnames(a)
## [[1]]
## [1] "a" "b" "c"
## 
## [[2]]
## [1] "a" "b" "c"

# Lengths

length(v)
## [1] 9
nrow(m);ncol(m)
## [1] 3
## [1] 3
dim(a)
## [1] 3 3

# Combine

c(v, c(j=10))
##  a  b  c  d  e  f  g  h  i  j 
##  1  2  3  4  5  6  7  8  9 10
rbind(m, 10:12);cbind(m, 10:12)
##    a  b  c
## a  1  4  7
## b  2  5  8
## c  3  6  9
##   10 11 12
##   a b c   
## a 1 4 7 10
## b 2 5 8 11
## c 3 6 9 12
abind::abind(a, 10:12, along=1); abind::abind(a, 10:12, along=2)
##    a  b  c
## a  1  4  7
## b  2  5  8
## c  3  6  9
##   10 11 12
##   a b c   
## a 1 4 7 10
## b 2 5 8 11
## c 3 6 9 12

# Transpose

t(m)
##   a b c
## a 1 2 3
## b 4 5 6
## c 7 8 9
aperm(a)
##   a b c
## a 1 2 3
## b 4 5 6
## c 7 8 9

# Type test

is.vector(v, mode='any')
## [1] TRUE
is.matrix(m)
## [1] TRUE
is.array(a)
## [1] TRUE
```

Vectors may have different dimensions. They may print similar but will behave differently. Pay 
attention to [] syntax:


```r
# 1d vector
str(1:10)
##  int [1:10] 1 2 3 4 5 6 7 8 9 10

# column vector
str(matrix(1:10, ncol = 1))
##  int [1:10, 1] 1 2 3 4 5 6 7 8 9 10

# row vector
str(matrix(1:10, nrow = 1))
##  int [1, 1:10] 1 2 3 4 5 6 7 8 9 10

# 1d "array" vector
str(array(1:10, 10))
##  int [1:10(1d)] 1 2 3 4 5 6 7 8 9 10
```

#### S3 Atomic Vectors{}

The attribute `class` is what makes an object part of the S3 object system in R, and 
changes how generic functions behave compared to regular vectors. S3 objects are built
on top of atomic vectors.

##### Factors{}

Factors are a ubiquitous S3 object in base R. They handle categorical variables where the 
vector must contain only known values. 

Factors are built on atomic vectors of type `integer`, and only have two attributes: `class` "factor" and `levels`, which defines the set, and sometimes order, of possible values.

Use `factor()` create a factor:

* *x*: a vector coercible to `character`
* *levels*: a unique vector of *expected* values in x; which can contain values not found in x or, conversely, exclude values found in x
* *labels*: a vector of name *aliases* for levels (in the same order as levels) or scalar


```r
# create a factor from a vector of state abbreviations
# tell R to expect values MO, NY, AK, HI and their aliases are Missouri, New York, Alaska, 
# & Hawaii
states <- factor(x = c('MO','NY','AK','MO','AK','IL'),
                 levels = c('MO','NY','AK','HI'),
                 labels = c('Missouri','New York','Alaska','Hawaii'))

# factor() converts the vector to integer values under-the-hood. These are rarely visible 
# but can be exposed using c() (or unclass() but then you no longer have a factor). The 
# integer refers to the sequence of levels given, starting with 'MO' in this case.
str(c(states))
##  int [1:6] 1 2 3 1 3 NA

# usually factors print their levels attribute
states
## [1] Missouri New York Alaska   Missouri Alaska   <NA>    
## Levels: Missouri New York Alaska Hawaii
# notice it did not print the levels given in factor(), but rather the label aliases 
# were given to attribute `levels`. If no labels argument is given it defaults to same
# as levels.
attributes(states)
## $levels
## [1] "Missouri" "New York" "Alaska"   "Hawaii"  
## 
## $class
## [1] "factor"

# if no levels argument was given, factor() would have defaulted to the unique set of values 
# in x in ascending order. Note this would include Illinois (which we originally excluded and
# values in states became `NA`), and would have excluded Hawaii (which we may have wanted
# even though it is not in the current vector x).
sort(unique(as.character(c('MO','NY','AK','MO','AK','IL'))))
## [1] "AK" "IL" "MO" "NY"

# notice tabulated results appear in level order and include counts even for values not
# present in states
table(states)
## states
## Missouri New York   Alaska   Hawaii 
##        2        1        2        0

# many base R functions convert character vectors to factor by default. This may not be 
# desirable as the vector may not contain the complete set of values, or infer their
# correct order.
class(data.frame(x=c('red','yellow','blue'))[[1]])
## [1] "factor"

# use arguement stringsAsFactor to inhibit default factor creation
class(data.frame(x=c('red','yellow','blue'),
                 stringsAsFactors = FALSE)[[1]])
## [1] "character"
```

Some categorical variables have order. Methods and modeling functions usually
treat ordered and unordered factors very differently. Use `ordered()` to create an 
object of `class` "ordered factor", where the order in levels implies the order 
between factor levels.


```r
priority <- ordered(x = c('low','medium','high'),
                    levels = c('low','medium','high'))

str(priority)
##  Ord.factor w/ 3 levels "low"<"medium"<..: 1 2 3
```

Finally, some functions coerce factors into strings (e.g. `grepl()`) while others
(e.g. `c()`) return the integer substrate. If you want string-like behavior, best to 
coerce `as.character()` before performing string operations.

##### Dates{}

Date vectors are S3 objects built on type `double`, and have only attribute: `class` "Date". They
represent days from the [UNIX Epoch](https://en.wikipedia.org/wiki/Unix_time), 01-Jan-1970; and take
into account leap-days, but not leap-seconds.


```r
# today's date
today <- Sys.Date()
typeof(today)
## [1] "double"
class(today)
## [1] "Date"
# unclass the Date object to see the underlying double
unclass(today)
## [1] 18399
```

##### Date-times{}

S3 Date-times come in two flavors: POSIX^[Portable Operating System Interface standard.]ct *(calendar time)* & POSIXlt *(local time)*. POSIXct is build on type `double` whereas POSIXlt is type `list`,
which we'll discuss later.

POSIXct represents the number of seconds since the UNIX Epoch, 01-JAN-1970, and 
has the attributes `class` & `tzone`. Pass the `tz` argument a [tz database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) time zone 
to change attribute `tzone` from the default local time. This does not change the underlying
`double`, only how it prints.


```r
ct <- as.POSIXct(x = "2020-04-28 05:00:00", tz='UTC')
# the calendar time object
ct
## [1] "2020-04-28 05:00:00 UTC"

# built on doubles
typeof(ct)
## [1] "double"
# has attributes class and tzone
attributes(ct)
## $class
## [1] "POSIXct" "POSIXt" 
## 
## $tzone
## [1] "UTC"

# change tzone attribute to EDT, CDT, JST; notice midnight does not print a time
attr(ct, which = 'tzone') <- 'America/New_York'; ct;
## [1] "2020-04-28 01:00:00 EDT"
attr(ct, which = 'tzone') <- 'America/Chicago'; ct
## [1] "2020-04-28 CDT"
attr(ct, which = 'tzone') <- 'Asia/Tokyo'; ct
## [1] "2020-04-28 14:00:00 JST"

# the underlying double type atomic vector
unclass(ct)[1]
## [1] 1588050000
```

##### Difftimes{}

Difftimes are `doubles` which represent a duration of time (interpreted by attribute `units`) between
two dates/datetimes. `units` includes "secs", "mins", "hours", "days", and "weeks" and are *not*
sensitive to daylight savings time.

Limited math methods and arithmetic is available for difftime. See `?difftime`.


```r
# create difftime directly
day <- as.difftime(1, units = 'days')
day
## Time difference of 1 days
# create difftime by arithmetic
day_alt <- as.Date("2020-04-28") - as.Date("2020-04-27")
day_alt
## Time difference of 1 days

typeof(day)
## [1] "double"
class(day)
## [1] "difftime"
attributes(day)
## $class
## [1] "difftime"
## 
## $units
## [1] "days"
# retrive/set units directly using units()
units(day) <- 'hours'
# day is now a 24 hour timediff
day
## Time difference of 24 hours

# add difftime to Date to return Date
as.Date("2020-04-28") + (2*day)
## [1] "2020-04-30"

# add difftime to difftime to return difftime
as.difftime(.5, units='mins') + as.POSIXct('2020-03-02 00:00:00')
## [1] "2020-03-02 00:00:30 CST"

# transform difftime by numeric to return difftime
day/2
## Time difference of 12 hours

# transform difftime by difftime to return difftime
day+day
## Time difference of 48 hours

# math methods are to describe difftime vectors
mean(c(as.Date("2020-04-28") - as.Date("2020-04-27"), 
       as.Date("2020-04-28") - as.Date("2020-04-26")))
## Time difference of 1.5 days

# round difftime etc.
round(day, -1)
## Time difference of 20 hours

# the underlying double
unclass(day)[1]
## [1] 24
```

#### Lists{}

[Earlier](#arlobjects) we learned lists are just vectors of references to objects of any
type. They have only attribute `class` "list".

Lists are created with `list()`. Notice they can contain references to other list, and so 
are called *recursive* vectors:

```r
l <- list(1,
          letters[1:10], 
          c(T,T,F,F), 
          list(1))

str(l)
## List of 4
##  $ : num 1
##  $ : chr [1:10] "a" "b" "c" "d" ...
##  $ : logi [1:4] TRUE TRUE FALSE FALSE
##  $ :List of 1
##   ..$ : num 1
```

`c()` will convert vectors into lists before combining them:


```r
c(list(letters[1:3]), 1:3) %>% str()
## List of 4
##  $ : chr [1:3] "a" "b" "c"
##  $ : int 1
##  $ : int 2
##  $ : int 3
```

##### Testing and coercing lists {}


```r
# test for list type
is.list(l)
## [1] TRUE
typeof(l)
## [1] "list"

# already seen list(), but as.list() exists as well
as.list(1:4)
## [[1]]
## [1] 1
## 
## [[2]]
## [1] 2
## 
## [[3]]
## [1] 3
## 
## [[4]]
## [1] 4

# to coerce list to vector
unlist(l)
##  [1] "1"     "a"     "b"     "c"     "d"     "e"     "f"     "g"    
##  [9] "h"     "i"     "j"     "TRUE"  "TRUE"  "FALSE" "FALSE" "1"
```

#### Data frames and tibbles{}

Data frames are the most important data structure for data analysis in R. They are
built on top of lists with attributes `class` "data.frame", "names" *(for columns)*,
and "row.names"; and, crucially, requires each element be a vector of equal length.

Tibbles are data frames that were later developed as part of the [Tidyverse](https://www.tidyverse.org/). They try and optimize the default 
behavior of `class` data.frame in the following ways.

##### Creating data.frame or tbl_df{}

`data.frame()` creates a data frame object from ... name = vector pairs:


```r
# create character vector and add names attribute 'Color'
color <- rep(x = c('red','blue','yellow'), times = 2)
names(color) <- 'Color'

df <- data.frame(sock_id = 1:6, # use name = vector pairs
                 color, # or vectors with attribute "names"
                 6:1, # or let R use defaults
                 `1` = 1 # but non-syntactic names will be renamed without warning
                 ) 

typeof(df)
## [1] "list"
str(df)
## 'data.frame':	6 obs. of  4 variables:
##  $ sock_id: int  1 2 3 4 5 6
##  $ color  : Factor w/ 3 levels "blue","red","yellow": 2 1 3 2 1 3
##  $ X6.1   : int  6 5 4 3 2 1
##  $ X1     : num  1 1 1 1 1 1
```

Notice data.frame automatically creates factors from `character` vectors. Inhibit
with `stringsAsFactors` arguement:


```r
data.frame(letters[1:3],
           stringsAsFactors = FALSE) %>% str()
## 'data.frame':	3 obs. of  1 variable:
##  $ letters.1.3.: chr  "a" "b" "c"
```

Like lists and matrices, data frames and tibbles have row and column names and dimensional length:


```r
# column names
names(df); colnames(df)
## [1] "sock_id" "color"   "X6.1"    "X1"
## [1] "sock_id" "color"   "X6.1"    "X1"

# column length
length(df); ncol(df)
## [1] 4
## [1] 4

# row names
rownames(df)
## [1] "1" "2" "3" "4" "5" "6"

# row length
nrow(df)
## [1] 6

# row and column dimensions
dim(df)
## [1] 6 4
```

Tibbles have `class` "tbl_df" (as well as data.frame) which modifies their behavior. They
are created similarly to data frames:


```r
library(tibble)

tbl <- tibble(sock_id = 1:6, # use name = vector pairs
                 color, # or vectors with attribute "names"
                 6:1, # or let R use defaults
                 `1` = 1, # allows non-syntactic names
                 `1*2` = `1` * 2 # allows reference to new variables
              )

typeof(tbl)
## [1] "list"
str(tbl)
## Classes 'tbl_df', 'tbl' and 'data.frame':	6 obs. of  5 variables:
##  $ sock_id: int  1 2 3 4 5 6
##  $ color  : chr  "red" "blue" "yellow" "red" ...
##  $ 6:1    : int  6 5 4 3 2 1
##  $ 1      : num  1 1 1 1 1 1
##  $ 1*2    : num  2 2 2 2 2 2
```
Tibbles can use the same methods as data tables, for example names and dimensional length:


```r
colnames(tbl); ncol(tbl)
## [1] "sock_id" "color"   "6:1"     "1"       "1*2"
## [1] 5
rownames(tbl); nrow(tbl)
## [1] "1" "2" "3" "4" "5" "6"
## [1] 6
```


Notice `tibble()` did not coerce factors by default, allowed non-syntactic names 
(enclosed in ````), and let you reference new columns as they are created.

Both data frames and tibbles require equal length vector columns and will recycle
values if the vector length is smaller than the max column length; however, data
frames will recyle any vector that is an integer multiple of the longest column, while
tibbles will only recyle scalar values.


```r

data.frame(1:4, # the longest column vector of length 4
           1, # a scalar recyled 4 times
           1:2 # a length 2 vector recyled 2 times
           )
##   X1.4 X1 X1.2
## 1    1  1    1
## 2    2  1    2
## 3    3  1    1
## 4    4  1    2

tibble(1:4, # the longest column vector of length 4
           1, # a scalar recyled 4 times
           1:2 # a length 2 vector recyled 2 times
           )
## Error: Tibble columns must have consistent lengths, only values of length one are recycled:
## * Length 2: Column `1:2`
## * Length 4: Column `1:4`
```

##### Row Names {}

Row names, a unique vector of `character`, can be assigned to Data frames. This idea
probably arose due to data frame's close association to `numerical` matrices, where
storing `character` data along with the matrix is useful. Get and set row names with `rownames()`, or argument `row.names` on creation:


```r
df <- data.frame(idx = 1:3,
                 row.names = c('Spider Man','Batman','Red Robin'))
df
##            idx
## Spider Man   1
## Batman       2
## Red Robin    3
```

Row names can be used to subset data frames:


```r
df["Batman",]
## [1] 2
```

Tibbles do not store row names for reasons you can read [here](https://adv-r.hadley.nz/vectors-chap.html#rownames). Rather, they treat 
rownames as another feature of the data. The `rownames` argument in `as_tibble()`
or `rownames_to_column()` can transform row names to a column vector.


```r
as_tibble(df,
          rownames = "Super Hero")
## # A tibble: 3 x 2
##   `Super Hero`   idx
##   <chr>        <int>
## 1 Spider Man       1
## 2 Batman           2
## 3 Red Robin        3
```

##### Subsetting {}

Data frames can be subset either one dimensionaly, like a list, or two dimensionally,
like a matrix.

Data frame subsetting syntax can behave unexpectedly:

1. `data.frame()[,vars]` will return a data frame unless vars selects only one column; it returns a vector.

2. selecting a single column with `data.frame$var` will return a column starting with `\$var` if `$var` doesn't exist.

Tibbles always return tibbles, and vectors can be returned when desired with `tibble[[var]]`. If
a column is not found it returns an error.


```r
# data frame will select columns starting with cotton when $cotton not found
df <- data.frame(cotton_candy = 'Yum')
df$cotton
## [1] Yum
## Levels: Yum

# tibbles do not match columns starting with cotton
as_tibble(df)$cotton
## Warning: Unknown or uninitialised column: 'cotton'.
## NULL
```

##### Testing and coercion{}

Type | Testing | Coercion 
:--- | :--- | :---
data frame | `is.data.frame()` | `as.data.frame()` 
tibble | `is_tibble()` | `as_tibble()` 

Note that `is.data.frame()` will return true for tibbles as well, since tibbles
are `class` data.frame.

##### List Columns{}

Because data frames are built on lists, they themsevles can contain lists. So it
is possible in R to have a data frame containing data frames, making it easy
to organize related datasets:


```r
df <- data.frame(x = 1)
df$nested <- list(data.frame(cool = 1:3))

class(df$nested[[1]])
## [1] "data.frame"
df$nested[[1]]
##   cool
## 1    1
## 2    2
## 3    3
```

To include lists on creation use `I()`, which inhibits interpetation/conversion of objects
by setting the `class` to "as is":


```r
data.frame(x= 1,
           I(list(data.frame(1:3))))
##   x list.data.frame.1.3..
## 1 1                   1:3
```

Or use tibbles which can handle lists on creation without `I()`:


```r
tibble(x = 1,
       dfs = list(tibble(1:3)))
## # A tibble: 1 x 2
##       x dfs             
##   <dbl> <list>          
## 1     1 <tibble [3 x 1]>
```

#### NULL

`NULL` is a special type in R. It has no attributes and is always zero length. It
is primarily used for two things:

1. represent an empty vector

```r
c()
## NULL
```
2. represent a missing vector (e.g. often default function arguments are `NULL`). This is different
than `NULL` in SQL, which is more like `NA`, because SQL `NULL` and R `NA` represent missing
*elements* of a vector, rather than the vector itself.

Test for NULL with `is.null()`:


```r
is.null(NULL)
## [1] TRUE
```
