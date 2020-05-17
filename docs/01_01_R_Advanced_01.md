
## Advanced R{}

[Advanced R](https://adv-r.hadley.nz/) 2nd ed. Hadley Wickham  

*Selected Chapters*  
[Names and Values](#arnandv)  
[Vectors](#arv)  
[Subsetting](#arsub)  
[Control Flow](#arconflow)  
[Functions](#arfunc)  
[Functionals](#arfunctionals)  

### Names and Values{#arnandv}

#### Binding{#arbinding}
In R, names are assigned a value (e.g. a vector doesn't have a name, rather, a name(s) points to a vector). The actual address of an object is exposed by `lobstr::obj_addr()`.


```r
x <- 1:10
obj_addr(x)
## [1] "0x1517f8b0"

# notice x and y are pointers to the same object
y <- x
obj_addr(y)
## [1] "0x1517f8b0"

# this applies to function arguments in their environment as well
fn <- function(z){
  z
}
# fn() returns the argument z at the same address
obj_addr(fn(x))
## [1] "0x1517f8b0"
```

#### Syntactic Names{#arsyntacticnames}

Names in R can be letters, numbers, `.` or `_`; but cannot start with a number or `_` or contain `?Reserved` words.


```r
# assigning a vector a name startign with `_` throws an error
_x <- 1:10
## Error: <text>:2:1: unexpected input
## 1: # assigning a vector a name startign with `_` throws an error
## 2: _
##    ^
```


#### Copy-on-Modify {#arcopyonmod}

R objects are generally immutable. A new copy is made when you modify an object. There are two [Modify-in-Place](#armodinplace) exceptions.


```r
# create vector x
x <- 1:10
obj_addr(x)
## [1] "0x180fe9d8"

# modify first value in x
x[[1]] <- 0

# note x now points to a new object
obj_addr(x)
## [1] "0x1806d148"
```


##### Trace Copying of Objects {#artracecopy}

`base::tracemem()` will mark an object and print a message whenever it is copied. This is
a major cause of hard-to-predict memory usage.


```r
# tell R to trace copies of object reference `x`
tracemem(x = x)

# modify the first element in vector `x`
# R prints a tracemem message to show the object is copied to a new address
x[[1]] <- 2
#> tracemem[0x000002379ca5a190 -> 0x000002379cd8dbb0]:

# tell R to stop tracing this object
base::untracemem(x = x)
```

##### List Objects {#arlobjects}

Just like variables, each element of a list also points to a value. [Copy-on-modify](#arcopyonmod) and [modify-in-place](#armodinplace) applies here as well. R creates a **shallow** copy of the list. Meaning 
the list object and its bindings are copied; however, the underlying values are not.

In a deep copy, like prior to R 3.1.0, the underlying values are also copied.


```r
# create a list named `l`
l <- list(1:10, TRUE, c('Apple','Broccoli','Chowder'))

# show the address of the third list element
obj_addr(l[[3]])
## [1] "0x18641c28"

# modify the third list element
l[[3]] <- c('Appricot','Basmati Rice','Cheese')

# the third list element's address is changed
obj_addr(l[[3]])
## [1] "0x186db4c0"
```

Use `lobstr::ref()` to see common values between lists:


```r
# create a list object
listA <- list(1,2,3)

# create a new pointer to the same list object
listB <- listA

# modify the third list element of listB, creating a shallow copy of the list object 
listB[[3]] <- 4

# ref() lists the address of each list element. Notice two values are still shared 
# between `listA` and `listB`.
ref(listA, listB)
## o [1:0x18ad0ff0] <list> 
## +-[2:0x189423a8] <dbl> 
## +-[3:0x18942370] <dbl> 
## \-[4:0x18942338] <dbl> 
##  
## o [5:0x18b87d90] <list> 
## +-[2:0x189423a8] 
## +-[3:0x18942370] 
## \-[6:0x18942220] <dbl>
```

##### Data Frames {#ardataframes}

A data frame is simply a list where each element is a vector of the same length.


```r
# create a data frame with three columns
df <- data.frame(col1 = TRUE, 
                 col2 = 1:10, 
                 col3 = rep(x = c('On','Off'), 
                            length.out = 10))

# show the address of the data frame and each element (i.e. column)
ref(df)
## o [1:0x195b9330] <df[,3]> 
## +-col1 = [2:0x19580628] <lgl> 
## +-col2 = [3:0x195aaaf0] <int> 
## \-col3 = [4:0x19580778] <fct>
```

When a **column** is modified only one element is copied:


```r
# modify a column and note the addresses of the unchanged columns remain the same
# because they point to the same objects as before
df$col1 <- FALSE; ref(df)
## o [1:0x198fd0d8] <df[,3]> 
## +-col1 = [2:0x198b3110] <lgl> 
## +-col2 = [3:0x195aaaf0] <int> 
## \-col3 = [4:0x19580778] <fct>
```

This has important consequences for memory when you update **rows** of data where each 
element gets copied:


```r
# modify the first row of the data frame
df[1,] <- c(TRUE, 0, 'Off')

# *all* elements are copied to new addresses
ref(df)
## o [1:0x15fded80] <df[,3]> 
## +-col1 = [2:0x160c22b0] <chr> 
## +-col2 = [3:0x158b5f90] <chr> 
## \-col3 = [4:0x160189b8] <fct>
```

##### Character Vectors {#archarvar}

R uses a *global string pool* to store unique character objects so they are not duplicated 
unnecessarily. Use the `character` argument to show their address in the global string pool 
with `lobstr::ref()`:


```r
# create a character vector
chr <- c('Quarter',
         'Dime',
         'Nickle',
         'Penny',
         'Penny')

# show addresses in the global string pool. Notice the shared value for 'Penny'
ref(chr, 
    character = TRUE)
## o [1:0x17e3a7f8] <chr> 
## +-[2:0x15cd8930] <string: "Quarter"> 
## +-[3:0x15cd88c0] <string: "Dime"> 
## +-[4:0x15cd8818] <string: "Nickle"> 
## +-[5:0x15cd87a8] <string: "Penny"> 
## \-[5:0x15cd87a8]
```

#### Object Size {#arobjsize}

`lobstr::obj_size()` shows the amount of memory an object takes, while `obj_sizes()`
breaks down multiple objects into their individual contribution to *total* memory.

Due to binding names, global string pools, and ALTREP (alternate representation where some vectors
are stored in a compact manor; e.g. 1:1000 only stores the first and last numbers, not 1,000 numbers)
the size of objects may surprise you.


```r
# create a length one chracter vector and a length 100 character vector
x <- 'Charlie'
y <- rep(x = 'Charlie', times = 100)

# y is only ~8x the size of x, not 100x
obj_size(x); obj_size(y)
## 112 B
## 904 B


z <- 1:10

# create a list with one element `z`
list1 <- list(z)
# create a list with three elements, all `z`
list2 <- list(z, z, z)

# list2 only contributes an extra 80 Bytes to the total memory between the two
obj_sizes(list1, list2)
## * 736 B
## *  80 B
```


#### Modify-in-Place {#armodinplace}

There are two places where R will optimize memory by modfiying an object "in place"
(i.e. does not make a copy).

1. objects with a single binding

```
> hats <- c('red','brown','blue')
> obj_addr(hats)
[1] "0x1d51fe614d8"
> 
> hats[[1]] <- 'orange'
> obj_addr(hats)
[1] "0x1d51fe614d8"

# note this is output from the R Console. RStudio runs everything within an Environment
# which breaks modify-in-place behavior
```
This is one reason why looping in R is inefficient, the user is unwittingly copying 
objects many times over:


```r
# create a dataframe
df <- data.frame(col1 = c(1:3))

# trace copies of this dataframe
tracemem(df)
## [1] "<00000000187C6918>"

# use a for loop to increment the dataframe values
# R copies the object 12 times!!!
for (i in 1:3){
  df[[1]][i] <- df[[1]][i] + 1
}
## tracemem[0x00000000187c6918 -> 0x0000000018a09a00]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x0000000018a09a00 -> 0x0000000018a09840]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x0000000018a09840 -> 0x0000000018a19d10]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x0000000018a19d10 -> 0x0000000018a19b88]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x0000000018a19b88 -> 0x0000000018a19a00]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x0000000018a19a00 -> 0x0000000018a198b0]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x0000000018a198b0 -> 0x0000000018a197d0]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x0000000018a197d0 -> 0x0000000018a19648]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x0000000018a19648 -> 0x0000000018a194c0]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x0000000018a194c0 -> 0x0000000018a19370]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x0000000018a19370 -> 0x0000000018a19290]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x0000000018a19290 -> 0x0000000018a19108]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local

# turn of object memory trace
untracemem(df)
```

2. Environments

Environments are always modified in place and all objects within the environment
keep the same reference.

#### Unbinding and Garbage Collector (GC) {#argc}

The (GC) garbage collector deletes objects that are no longer used and requests
more memory from the operating system as needed to create objects.

For example:

```r
# create three objects all bound to `x`
x <- 1:10 
x <- c('one','two','three')
x <- TRUE

# removing `x` causes GC to delete the three objects b/c they have no other bindings
remove(x)
```
You can call GC yourself with `base::gc()`, but the user should not ever have the need.
