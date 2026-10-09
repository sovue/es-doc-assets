# docs/dev

## some symbols

| name | symbols |
| --- | --- |
| arrows | `↑` `↓` `←` `→` |
| tiret | `—` |
| bullets and squares | `∙` `◦` `▪` `▫` |

## tags

### info

:::info
**info tag**

to add additional information
:::

```md
:::info
**info tag**

to add additional information
:::
```

### attention

:::attention
**attention tag**

to underline important details
:::

```md
:::attention
**attention tag**

to underline important details
:::
```

### warning

:::warning
**warning tag**

to warn the user
:::

```md
:::warning
**warning tag**

to warn the user
:::
```

### tip

:::tip
**tip tag**

to give a tip to the user
:::

```md
:::tip
**tip tag**

to give a tip to the user
:::
```

### danger

:::danger
**danger tag**

to warn the user about a hard limit or something that will break
:::

```md
:::danger
**danger tag**

to warn the user about a hard limit or something that will break
:::
```

### details

:::details
**details tag**

to hide long or optional content behind a clickable summary
:::

```md
:::details
**details tag**

to hide long or optional content behind a clickable summary
:::
```

text right after the opening tag replaces the block title

### stub

:::stub
:::

```md
:::stub
:::
```

text between the opening and closing tags replaces the description

### wip

:::wip
:::

```md
:::wip
:::
```

text between the opening and closing tags replaces the description

### outdated

:::outdated
:::

```md
:::outdated
:::
```

text between the opening and closing tags replaces the description

### old

:::old
:::

```md
:::old
:::
```

text between the opening and closing tags replaces the description

### about

::about 1 | 2 | [3](#)

```md
::about 1 | 2 | [3](#)
```

### table

Tables use GitHub Flavored Markdown (GFM): `|` separates cells, the first row
contains headings, and the second row contains `---` for each column.
No `:::table` wrapper is needed.

| 1 | 2 | 3 |
| --- | --- | --- |
| 4 | 5 | 6 |
| 7 | 8 | 9 |

```md
| 1 | 2 | 3 |
| --- | --- | --- |
| 4 | 5 | 6 |
| 7 | 8 | 9 |
```

Use `:---` for left alignment, `:---:` for centered text, and `---:` for right
alignment. Inline formatting, links, and code work in cells:

| Name | Example | Count |
| :--- | :---: | ---: |
| **Dissolve** | `Dissolve(1.5)` | 3 |
| [Text tags](/docs/text_tags) | `a\|b` | 10 |

```md
| Name | Example | Count |
| :--- | :---: | ---: |
| **Dissolve** | `Dissolve(1.5)` | 3 |
| [Text tags](/docs/text_tags) | `a\|b` | 10 |
```

Escape a literal pipe as `\|`, including inside inline code. Semicolons are
ordinary text. Leave a blank line before and after a table; a blank line ends
it. Every row occupies one source line; use `<br>` for a line break inside a
cell. Block elements such as lists and code fences cannot go inside cells.

The header and separator must have the same number of columns. Missing body
cells are filled with empty cells; extra cells are ignored. Outer pipes are
optional, but keeping them makes the source easier to read.

[GFM table specification](https://github.github.com/gfm/#tables-extension-).

### audio

::audio /resource/community/sound/music/disoul.ogg | 140 kilograms of sex - Distorted Souls

```md
::audio /resource/community/sound/music/disoul.ogg | 140 kilograms of sex - Distorted Souls
```

### references

text^1  
other text^2

1^ reference
2^ other reference

```md
text^1  
other text^2

1^ reference
2^ other reference
```
