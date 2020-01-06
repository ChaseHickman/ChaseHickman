# Markdown/bookdown tips

## Syntax
### Inline Formatting {-}

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
<span style="font-variant:small-caps;">Small Caps</span> | `<span style="font-variant:small-caps;">Small Caps</span>` |
[Hyperlink](https://www.google.com) | `[Hyperlink](https://www.google.com)` | `[text](link)`
![R Logo](./Rlogo.png) | `![R Logo](./Rlogo.png)` | `![text](link)`
Footnote^[Example footnote.] | `^[Example footnote.]` | 
@example | `@example` | \@tag ref to BibTeX citation
[@example] | `[@example]` | alternative style

### Section Headers {-}

```
# level 1 header
## level 2 header
### level 3 header etc...
```

Add `{-}` after the header to remove section numbering:

```
# Preface {-}
```
### Unordered Lists {-}

* item 1
* item 2
    * sub-item 1

Unordered list items start with `*`, `-`, or `+`; and can be nested by indenting the next item by four spaces:

```
* item 1
* item 2
    * sub-item 1
```
### Ordered lists {-}

1. item 1
2. item 2
    1. sub-item 1

Ordered lists begin with a number and a period. Same indenting rules as unordered lists:

```
1. item 1
2. item 2
    1. sub-item 1
```

### Block Quotes {-}

> "You miss 100% of the shots you don't take. -Wayne Gretzky"
>
> --- Michael Scott

Block quotes begin with a `>`:

```
> "You miss 100% of the shots you don't take. -Wayne Gretzky"
>
> --- Michael Scott
```

You can use markdown formatting or HTML tags to right align the author:

> "You miss **100%** of the shots you don't take. -Wayne Gretzky"
>
> ## <div style="text-align: right">--- Michael Scott</div>

```
> "You miss **100%** of the shots you don't take. -Wayne Gretzky"
>
> ## <div style="text-align: right"> --- Michael Scott</div>
```

### Code Blocks {-}

```
    ```
    Use three back ticks to display text without markdown formatting.
    ```
```
### Horizontal Rules {-}

***

Three or more `***`, `---`, or `___` will produce a horizontal rule; as will the `<hr>` HTML tag.

## Cache large code chunks

` ```{r chunk-id, cache=TRUE} `

## Resources {-}
[Bookdown: Authoring Books with R Markdown](https://bookdown.org/yihui/bookdown) by its creator, Yihui Xie.

[Pandoc](https://pandoc.org/)

## Chapter Session Info {-}

```
## R version 3.3.2 (2016-10-31)
## Platform: x86_64-apple-darwin13.4.0 (64-bit)
## Running under: OS X El Capitan 10.11.6
## 
## locale:
## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
## 
## attached base packages:
## [1] methods   stats     graphics  grDevices utils     datasets  base     
## 
## loaded via a namespace (and not attached):
##  [1] magrittr_1.5    bookdown_0.16   tools_3.3.2     htmltools_0.3.6
##  [5] yaml_2.2.0      Rcpp_1.0.0      stringi_1.4.3   rmarkdown_1.12 
##  [9] knitr_1.22      stringr_1.4.0   xfun_0.11       digest_0.6.10  
## [13] evaluate_0.13
```
