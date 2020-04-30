## Markdown/bookdown tips

### Syntax
#### Inline Formatting {-}

Element | Code | Notes
--- | --- | ---
*Italic* | `*Italic* or _Italic_` | single asterisks or underscores
**Bold** | `**Bold** or __Bold__` | double asterisks or underscores
***Bold Italic*** | `***Bold Italic***` | triple asterisks or underscores
__Underline__ | `__Underline__` | two underscores
~~Strike-through~~ | `~~Strike-through~~` | double tildes
Sub~script~ | `Sub~script~` | single tildes
Super^script^ | `Super^script^` | single carets
`inline_code()` | `` `inline_code()` `` | single back-ticks
R, Inline, Code | `` `r_ c('R','Inline','Code')` `` | *no underscore*
[Hyperlink](https://www.google.com) | `[Hyperlink](https://www.google.com)` | `[text](link)`
![R Logo](./Rlogo.png) | `![R Logo](./Rlogo.png)` | `![text](link)`
Footnote^[Example footnote.] | `^[Example footnote.]` | 
@example | `@example` | \@tag ref to BibTeX citation
[@example] | `[@example]` | alternative style

#### Section Headers {-}

```
# level 1 header
## level 2 header
### level 3 header etc...
```

Add `{-}` after the header to remove section numbering:

```
# Preface {-}
```
#### Unordered Lists {-}

* item 1
* item 2
    * sub-item 1

Unordered list items start with `*`, `-`, or `+`; and can be nested by indenting the next item by four spaces:

```
* item 1
* item 2
    * sub-item 1
```
#### Ordered lists {-}

1. item 1
2. item 2
    1. sub-item 1

Ordered lists begin with a number and a period. Same indenting rules as unordered lists:

```
1. item 1
2. item 2
    1. sub-item 1
```

#### Block Quotes {-}

> "You miss 100% of the shots you don't take.
>
> --- Wayne Gretzky"
>
> <div style="text-align: right"> --- Michael Scott</div>

Block quotes begin with a `>`. You can use markdown formatting or HTML tags to right align the author:

```
> "You miss 100% of the shots you don't take.
>
> --- Wayne Gretzky"
>
> <div style="text-align: right"> --- Michael Scott</div>
```



#### Code Blocks {-}

```
    ```
    Use three back ticks to display text without markdown formatting.
    ```
```
#### Horizontal Rules {-}

***

Three or more `***`, `---`, or `___` will produce a horizontal rule; as will the `<hr>` HTML tag.

### Cache large code chunks

` ```{r chunk-id, cache=TRUE} `

### Resources {-}
[Bookdown: Authoring Books with R Markdown](https://bookdown.org/yihui/bookdown) by its creator, Yihui Xie.

[Pandoc](https://pandoc.org/)

### Chapter Session Info {-}

```
## R version 3.6.1 (2019-07-05)
## Platform: x86_64-w64-mingw32/x64 (64-bit)
## Running under: Windows 10 x64 (build 18363)
## 
## Matrix products: default
## 
## locale:
## [1] LC_COLLATE=English_United States.1252 
## [2] LC_CTYPE=English_United States.1252   
## [3] LC_MONETARY=English_United States.1252
## [4] LC_NUMERIC=C                          
## [5] LC_TIME=English_United States.1252    
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## loaded via a namespace (and not attached):
##  [1] compiler_3.6.1  magrittr_1.5    bookdown_0.16   tools_3.6.1    
##  [5] htmltools_0.4.0 yaml_2.2.0      Rcpp_1.0.1      stringi_1.4.3  
##  [9] rmarkdown_1.13  knitr_1.23      stringr_1.4.0   xfun_0.8       
## [13] digest_0.6.20   rlang_0.4.5     evaluate_0.14
```
