## Markdown/bookdown tips

### Syntax
#### Inline Formatting {}

Element | Code | Notes
--- | --- | ---
*Italic* | `*Italic* or _Italic_` | single asterisks or underscores
**Bold** | `**Bold** or _Bold_` | double asterisks or underscores
***Bold Italic*** | `***Bold Italic***` | triple asterisks or underscores
__Underline__ | `__Underline__` | two underscores
~~Strike-through~~ | `~~Strike-through~~` | double tildes
Sub~script~ | `Sub~script~` | single tildes
Super^script^ | `Super^script^` | single carets
$\LaTeX$ | `$\LaTeX$` | enclose syntax with `$`
`inline_code()` | `` `inline_code()` `` | single back-ticks
R, Inline, Code | `` `r c('R','Inline','Code')` `` | single back-ticks beginning r *space*
[Hyperlink](https://www.google.com) | `[Hyperlink](https://www.google.com)` | `[text](link)`
![R Logo](./Rlogo.png) | `![R Logo](./Rlogo.png)` | `![text](link)`
Footnote^[Example footnote.] | `^[Example footnote.]` | <span style="color: red;">requires a </span> `.`
@example | `@example` | \@tag ref to BibTeX citation
[@example] | `[@example]` | alternative style

#### Line Returns

To force a line return use two spaces instead of one.

#### Section Headers {}

```
# level 1 header
## level 2 header
### level 3 header etc...
```

Add `{-}` after the header to remove section numbering:

```
# Preface {-}
```
#### Unordered Lists {}

* item 1
* item 2
    * sub-item 1

Unordered list items start with `*`, `-`, or `+`; and can be nested by indenting the next item by four spaces:

```
* item 1
* item 2
    * sub-item 1
```
#### Ordered lists {}

1. item 1
2. item 2
    1. sub-item 1

Ordered lists begin with a number and a period. Same indenting rules as unordered lists:

```
1. item 1
2. item 2
    1. sub-item 1
```

#### Block Quotes {}

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

#### Code Blocks {}

```
    ```
    Use three back ticks to display a block of text without markdown formatting.
    ```
```

##### Syntax highlighting

Include the language name after the initial backticks:

```
    ```r
        data.frame(x = 1:10)
    ``` 
```
renders as :

```r
    data.frame(x = 1:10)
```


#### Tables

##### Grid Tables

Basic tables can be used by aligning `+`,`-`, & `|` in a grid as below. One
advantage is the ability to put more than one markdown element in a cell. No
alignment options are available and you may have to reformat your table manually.

```
+------------+--------+--------+
|Date        |Event   |   Times|
+------------+--------+--------+
|30-Apr-2020 |Swimming|* 3:00pm|
|            |        |* 4:30pm|
+------------+--------+--------+
```
+------------+--------+--------+
|Date        |Event   |   Times|
+------------+--------+--------+
|30-Apr-2020 |Swimming|* 3:00pm|
|            |        |* 4:30pm|
+------------+--------+--------+

##### Pipe Tables

Pipe tables are more flexible on formatting and allow alignment. Space must be left before and 
after the table, but pipes do not have to align. The dashes indicate the header and colon for 
aligning the text.

```

Show *(left)* | Character *(center)* | Color *(right)*
:--- | :---: | ---:
Paw Patrol | Rocky | Green
Paw PatrolRubble | Yellow
Paw Patrol | Chase | Blue
Paw Patrol | Marshall | Red
Paw Patrol | Zuma | Orange
Paw Patrol | Sky | Pink

```

returns:

Show | Character | Color
:--- | :---: | ---:
Paw Patrol | Rocky | Green
Paw Patrol | Rubble | Yellow
Paw Patrol | Chase | Blue
Paw Patrol | Marshall | Red
Paw Patrol | Zuma | Orange
Paw Patrol | Sky | Pink


#### Horizontal Rules {}
Three or more `***`, `---`, or `___` will produce a horizontal rule; as will the `<hr>` HTML tag.

#### Task lists
For checked and unchecked tasklist items you can write an unordered list with `[ ]` or `[x]`.
Notice the space between and surrounding brackets with text following.

```
- [ ] Garbage
- [x] Dishes
```

- [ ] Garbage
- [x] Dishes

### Cache large code chunks

` ```{r chunk-id, cache=TRUE} `

### Chunk Options

#### Collapse

collapse=TRUE results in 

```r
x <- 1:10; print(x)
##  [1]  1  2  3  4  5  6  7  8  9 10
```
rather than

```r
x <- 1:10; print(x)
```

```
##  [1]  1  2  3  4  5  6  7  8  9 10
```

#### Error

error=TRUE 

### Resources {}
[Bookdown: Authoring Books with R Markdown](https://bookdown.org/yihui/bookdown) by its creator, Yihui Xie.

[Pandoc](https://pandoc.org/)
