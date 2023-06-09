<h1 align='center'> Overview of Markdown formatting</h1>

Note the title uses HTML `h1` tags to center align the text.

# H1 (top level section)

## H2 (subsection)

### H3 (subsubsection)

#### H4

##### H5

###### H6

## Paragraph formatting

Paragraphs are separated by blank lines.
This does nothing.

This, however, starts a new paragraph.

<!-- Comments are written like this. -->

> Quote using angle brackets. 
> Similar to paragraphs, this continues the same paragraph.
>
> Add a blank line to break paragraphs.
> 
>> Quotes can be nested with additional angle brackets.

## Text styling

**Bold text uses double asterisks.**

*Italic text uses single astersisks.*

***Bold and italic uses triple asterisks.***

**Bold can be used with _nested italic_.**

*Italic can be used with __nested bold__.*

~~Double tildes create strikethrough text.~~

Sub tags create <sub>subscript text.</sub>

Sup tags create <sup>superscript text</sup>.

## Links and referencing.

Links to websites use [square brackets](https://en.wikipedia.org/wiki/Cat).

Relative link to [another file](/README.md).

Relative link with [a heading specified](/practical-matters/installing-python.md#download-and-installation).

![Some alt text for an image.](https://upload.wikimedia.org/wikipedia/commons/b/bb/Kittyply_edit1.jpg)

Text can have footnotes[^1].

[^1]: The reference to the footnote above. Note that this always appears at the end of the file.

## Lists

1. Numbers create an ordered list.
2. Add items as follows
   1. Add a sublist as follows.
      1. Third level of indentation.

- Dashes create an unordered list.
  - This, too can be indented.
    - The third level of indentation.

- [ ] Task lists can have checkboxes
- [x] A ticked box.

## Tables

| One column | A second column | A third column |
| :--- | :---: | ---: |
Some text in a column that is left aligned. | Some center aligned text. | Some right aligned text. |

## Other formatting

### Math

Math can be written inline with single \$ characters: $a^2 + b^2 = c^2$

Or in a full math block with double \$\$ characters:

$$
cos^2 \theta + \sin^2 \theta = 1
$$

### Collapsed sections

<details>

<summary>A collapsed section.</summary>

Section contents.

</details>

### Code

`Inline monospaced code` can be surrounded with backticks.

```
A code block consists of triple backticks.
```

```python
# Specific syntax highlighting

a = 1
b = 2
print(f'{a + b = })
```

### Diagrams

```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```
