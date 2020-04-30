## Advanced R

[Advanced R](https://adv-r.hadley.nz/) 2nd ed. Hadley Wickham

### Binding

#### Names and Values
In R, names are assigned a value (e.g. a vector doesn't have a name, rather, a name(s) points to a vector). The actual address of an object is exposed by `lobstr::obj_addr()`.


```r
x <- 1:10
obj_addr(x)
## [1] "0x16b0ed38"

# notice x and y are pointers to the same object
y <- x
obj_addr(y)
## [1] "0x16b0ed38"

# this applies to function arguments in their environment as well
fn <- function(z){
  z
}
# fn() returns the argument z at the same address
obj_addr(fn(x))
## [1] "0x16b0ed38"
```

#### Syntactic Names

Names in R can be letters, numbers, `.` or `_`; but cannot start with a number or `_` or contain `?Reserved` words.


```r
# assigning a vector a name startign with `_` throws an error
_x <- 1:10
## Error: <text>:2:1: unexpected input
## 1: # assigning a vector a name startign with `_` throws an error
## 2: _
##    ^
```


#### Copy-on-Modify {#copyonmod}

R objects are generally immutable. A new copy is made when you modify an object. There are two [Modify-in-Place](#modinplace) exceptions.


```r
# create vector x
x <- 1:10
obj_addr(x)
## [1] "0x18ffbb98"

# modify first value in x
x[[1]] <- 0

# note x now points to a new object
obj_addr(x)
## [1] "0x1904b6e8"
```


##### Trace Copying of Objects

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

##### Lists

Just like variables, each element of a list also points to a value. [Copy-on-modify](#copyonmod) and [modify-in-place](#modinplace) applies here as well. R creates a **shallow** copy of the list. Meaning 
the list object and its bindings are copied; however, the underlying values are not.

In a deep copy, like prior to R 3.1.0, the underlying values are also copied.


```r
# create a list named `l`
l <- list(1:10, TRUE, c('Apple','Broccoli','Chowder'))

# show the address of the third list element
obj_addr(l[[3]])
## [1] "0x19570368"

# modify the third list element
l[[3]] <- c('Appricot','Basmati Rice','Cheese')

# the third list element's address is changed
obj_addr(l[[3]])
## [1] "0x19638798"
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
## o [1:0x19992778] <list> 
## +-[2:0x1981fb38] <dbl> 
## +-[3:0x1981fb00] <dbl> 
## \-[4:0x1981fac8] <dbl> 
##  
## o [5:0x19a0edd8] <list> 
## +-[2:0x1981fb38] 
## +-[3:0x1981fb00] 
## \-[6:0x1981f9b0] <dbl>
```

##### Data Frames

A data frame is simply a list where each element is a vector of the same length.


```r
# create a data frame with three columns
df <- data.frame(col1 = TRUE, 
                 col2 = 1:10, 
                 col3 = rep(x = c('On','Off'), 
                            length.out = 10))

# show the address of the data frame and each element (i.e. column)
ref(df)
## o [1:0x1ac74148] <df[,3]> 
## +-col1 = [2:0x1ac5a2e0] <lgl> 
## +-col2 = [3:0x1ac4e278] <int> 
## \-col3 = [4:0x1ac5a430] <fct>
```

When a **column** is modified only one element is copied:


```r
# modify a column and note the addresses of the unchanged columns remain the same
# because they point to the same objects as before
df$col1 <- FALSE; ref(df)
## o [1:0x1afb5fc0] <df[,3]> 
## +-col1 = [2:0x1afa33d0] <lgl> 
## +-col2 = [3:0x1ac4e278] <int> 
## \-col3 = [4:0x1ac5a430] <fct>
```

This has important consequences for memory when you update **rows** of data where each 
element gets copied:


```r
# modify the first row of the data frame
df[1,] <- c(TRUE, 0, 'Off')

# *all* elements are copied to new addresses
ref(df)
## o [1:0x1b3b9a48] <df[,3]> 
## +-col1 = [2:0x1b3ae198] <chr> 
## +-col2 = [3:0x1b3c3d70] <chr> 
## \-col3 = [4:0x1b3b2728] <fct>
```

##### Character Vectors

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
```

```
## o [1:0x16a8ac68] <chr> 
## +-[2:0x1586dac8] <string: "Quarter"> 
## +-[3:0x1586dba8] <string: "Dime"> 
## +-[4:0x1586dcc0] <string: "Nickle"> 
## +-[5:0x1586dd30] <string: "Penny"> 
## \-[5:0x1586dd30]
```

#### Object Size

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


#### Modify-in-Place {#modinplace}

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
## [1] "<0000000019644608>"

# use a for loop to increment the dataframe values
# R copies the object 12 times!!!
for (i in 1:3){
  df[[1]][i] <- df[[1]][i] + 1
}
## tracemem[0x0000000019644608 -> 0x00000000198a3c48]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x00000000198a3c48 -> 0x00000000198a3a88]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x00000000198a3a88 -> 0x00000000198b3f58]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x00000000198b3f58 -> 0x00000000198b3dd0]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x00000000198b3dd0 -> 0x00000000198b3c48]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x00000000198b3c48 -> 0x00000000198b3af8]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x00000000198b3af8 -> 0x00000000198b3a18]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x00000000198b3a18 -> 0x00000000198b3890]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x00000000198b3890 -> 0x00000000198b3708]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x00000000198b3708 -> 0x00000000198b35b8]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x00000000198b35b8 -> 0x00000000198b34d8]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
## tracemem[0x00000000198b34d8 -> 0x00000000198b3350]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local

# turn of object memory trace
untracemem(df)
```

2. Environments

Environments are always modified in place and all objects within the environment
keep the same reference.

#### Unbinding and Garbage Collector (GC)

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

#### Session Info


```r
print(x = sessionInfo(), 
      local = F)
```

```
## R version 3.6.1 (2019-07-05)
## Platform: x86_64-w64-mingw32/x64 (64-bit)
## Running under: Windows 10 x64 (build 18363)
## 
## Matrix products: default
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] lobstr_1.1.1
## 
## loaded via a namespace (and not attached):
##  [1] Rcpp_1.0.1      bookdown_0.16   digest_0.6.20   crayon_1.3.4   
##  [5] magrittr_1.5    evaluate_0.14   pillar_1.4.2    rlang_0.4.5    
##  [9] stringi_1.4.3   vctrs_0.2.4     rmarkdown_1.13  tools_3.6.1    
## [13] stringr_1.4.0   xfun_0.8        yaml_2.2.0      compiler_3.6.1 
## [17] htmltools_0.4.0 knitr_1.23
```
