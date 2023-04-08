# Section Header Formatting

## Note Title - Header 2

### Major Section - Header 3

#### Minor Section - Header 4

# Table of Contents

Insert table of contents with `[TOC]` as shown below:

[TOC]

Shows headings H1-H6 by default, but can be limited by modifying CSS.  To make it only print headers 1 & 2, I modified `github.css` (the default theme) and added this beneath the `.md-toc` code:

```css
.md-toc-h3 {
  display: none;
}

.md-toc-h4 {
  display: none;
}

.md-toc-h5 {
  display: none;
}

.md-toc-h6 {
  display: none;
}
```

# Typora Export Formatting

Note: Typora exports PDF's using A4 size, in case you are trying to merge them with Word and want the pages to be the same size.  

## CSS Files

CSS can be added to one of the CSS override files -- putting  it in `base.user.css` will affect all themes.

## PDF Export Margins

Add this to your theme (or override file) to narrow the margins of the PDF export

```css
@media print {
    html {
        font-size: 12px!important;
        padding: 0 !important;
        margin: 0 !important;
    }
    body.typora-export {
        padding: 0 !important;
        margin: 0 !important;
    }
    .typora-export #write {
        padding: 0 !important;
        margin-top: -2mm !important;
        margin-right: 0mm !important;
        margin-bottom: -2mm !important;
        margin-left: 0mm !important;
    }
}
```

Another option

```css
@media print {
    #write{
        max-width: 100%;
    }
    @page {
        size: A4;
        margin-left: 0;
        margin-right: 0;
    }
}
```

## Page breaks

To insert a page break on PDF export, use the code below (move cursor down to see code)

<div style="page-break-after: always; break-after: page;"></div>

This line will be on a new page thanks to the invisible HTML code above!

## Splitting Tables

By default, if a table is too big to fit on the page, it will put it on the next page (even if the rest of the table still has to span another page).  To fix this and make Typora split up the table, use this CSS

```css
@media print {
  table {
   page-break-inside: auto;
  }
}
```

