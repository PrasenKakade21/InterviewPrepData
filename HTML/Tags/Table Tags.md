# Table Tags

Table tags encode row-column relationships in data. The complete set covers the table, its caption, row groups, header/data cells, and column definitions.

The core idea:

```text
table
    ├── caption      → name
    ├── colgroup/col → columns
    ├── thead        → header rows
    ├── tbody        → data rows
    └── tfoot        → summary rows
```

---

# `<table>`

The table container for tabular data.

```html
<table>
  <caption>Kakade's study hours</caption>
  <tr><th scope="col">Month</th><th scope="col">Hours</th></tr>
  <tr><td>January</td><td>42</td></tr>
</table>
```

Output:

```text
Kakade's study hours
Month     Hours
January   42
```

Gotchas: not for page layout; `display: block` on table elements can remove table semantics.

---

# `<caption>`

The table's accessible name and visible title. Must be the first child.

```html
<table>
  <caption>Kakade.dev plans</caption>
  <tr><th scope="col">Plan</th><th scope="col">Price</th></tr>
</table>
```

Output:

```text
Kakade.dev plans
Plan   Price
```

Gotchas: a heading above the table is not a caption — it is not programmatically associated.

---

# `<thead>`, `<tbody>`, `<tfoot>`

Group header, body, and footer rows. `tbody` is implied if omitted, but explicit grouping is clearer.

```html
<table>
  <caption>Study hours</caption>
  <thead>
    <tr><th scope="col">Month</th><th scope="col">Hours</th></tr>
  </thead>
  <tbody>
    <tr><td>January</td><td>42</td></tr>
  </tbody>
  <tfoot>
    <tr><td>Total</td><td>42</td></tr>
  </tfoot>
</table>
```

Output:

```text
Header row, data row, summary row — each in its own group.
```

Gotchas: `tfoot` is valid before `tbody` in source order for some legacy reasons, but keep it last for clarity.

---

# `<tr>`

A table row. Contains `th` and `td` cells.

```html
<tr>
  <th scope="row">January</th>
  <td>42</td>
</tr>
```

Output:

```text
January   42
```

Gotchas: a row can contain only cells; other content is invalid.

---

# `<th>`

A header cell. Use `scope` to declare what it heads.

| `scope` | Heads |
| ------- | ----- |
| `col` | A column |
| `row` | A row |
| `colgroup` | A column group |
| `rowgroup` | A row group |

```html
<th scope="col">Hours</th>
<th scope="row">January</th>
```

Output:

```text
Screen reader announces "January, Hours, 42" for the data cell.
```

Gotchas: `th` is not "bold td" — it carries header semantics and needs `scope` in non-trivial tables.

---

# `<td>`

A data cell.

```html
<td>42</td>
```

Output:

```text
42
```

Gotchas: use `th` for headers; `td` cannot merge without `colspan`/`rowspan`.

---

# `colspan` and `rowspan`

Cell attributes that merge across columns or down rows.

```html
<tr>
  <th scope="col" colspan="2">Sessions</th>
</tr>
<tr>
  <th scope="row" rowspan="2">Tuesday</th>
  <td>JavaScript</td>
</tr>
<tr>
  <td>Testing</td>
</tr>
```

Output:

```text
Sessions spans two columns.
Tuesday spans two rows; later rows have one fewer cell.
```

Gotchas: merged cells reduce the cell count in later rows; miscounting shifts the grid.

---

# `<colgroup>` and `<col>`

Define column groups and column-level styling without repeating classes on every cell.

```html
<table>
  <caption>Progress</caption>
  <colgroup>
    <col class="topic-col" />
    <col class="score-col" />
  </colgroup>
  <tr><th scope="col">Topic</th><th scope="col">Score</th></tr>
</table>
```

Output:

```text
Two columns defined for styling; cells inherit the column classes.
```

Gotchas: `col` is void and can only carry a limited set of attributes (`span`, `style`, `class`, `width`); it does not affect cell content.

---

# Responsive Tables

Wrap the table in a scroll container instead of changing its display.

```html
<div class="table-scroll">
  <table>...</table>
</div>
<style>
  .table-scroll { overflow-x: auto; }
</style>
```

Output:

```text
Table scrolls horizontally on narrow screens with semantics intact.
```

Gotchas: `display: block` on `table`/`tr`/`td` can remove roles; if you must, add ARIA roles explicitly.

---

# Quick Reference

| Tag | Purpose |
| --- | ------- |
| `table` | Tabular data container |
| `caption` | Accessible table name |
| `thead` | Header rows |
| `tbody` | Data rows |
| `tfoot` | Summary rows |
| `tr` | Row |
| `th` | Header cell (`scope`) |
| `td` | Data cell |
| `colgroup` | Column group |
| `col` | Column definition |

> **Tables encode relationships: caption, row groups, `th` with `scope`, and careful merges make the grid readable to everyone.**
