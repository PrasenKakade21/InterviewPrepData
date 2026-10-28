# Tables

Tables are for tabular data — rows and columns where the relationship between cells matters. They are not layout tools. Interviewers check whether you use `caption`, `thead`/`tbody`, `th` with `scope`, and whether you can merge cells without breaking accessibility.

The core idea:

```text
table
    ├── caption        → what the table is
    ├── thead          → header rows
    ├── tbody          → data rows
    └── tfoot          → summary rows
```

A screen reader navigates by cell coordinates, so header cells must be marked with `th`, not styled `td`.

---

# 1. The table Element and caption

`table` wraps tabular data. `caption` gives the table an accessible name and visible title, and must be the first child.

```html
<table>
  <caption>Kakade's 2026 study hours</caption>
  <tr>
    <th scope="col">Month</th>
    <th scope="col">Hours</th>
  </tr>
  <tr>
    <td>January</td>
    <td>42</td>
  </tr>
</table>
```

Output:

```text
Kakade's 2026 study hours
Month     Hours
January   42
```

Think:

```text
caption → "Kakade's 2026 study hours"
th      → header cell (bold, announced as header)
td      → data cell
```

Interview trap: a `<div>` above the table is not a caption — it is not programmatically associated with the table.

---

# 2. thead, tbody, tfoot and tr, th, td

`thead` groups header rows, `tbody` groups data rows, `tfoot` groups summary rows. `tr` is a row; `th` is a header cell; `td` is a data cell.

```html
<table>
  <caption>Monthly study hours</caption>
  <thead>
    <tr>
      <th scope="col">Month</th>
      <th scope="col">HTML</th>
      <th scope="col">CSS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">January</th>
      <td>12</td>
      <td>10</td>
    </tr>
    <tr>
      <th scope="row">February</th>
      <td>14</td>
      <td>11</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <th scope="row">Total</th>
      <td>26</td>
      <td>21</td>
    </tr>
  </tfoot>
</table>
```

Output:

```text
Monthly study hours
Month      HTML  CSS
January    12    10
February   14    11
Total      26    21
```

`tbody` is implied if omitted, but explicit grouping makes styling and semantics clearer.

---

# 3. colspan and rowspan

`colspan` merges cells horizontally; `rowspan` merges vertically. Both change the grid, so the header structure must still make sense to assistive tech.

```html
<table>
  <caption>Team schedule</caption>
  <tr>
    <th scope="col">Day</th>
    <th scope="col" colspan="2">Sessions</th>
  </tr>
  <tr>
    <th scope="row">Monday</th>
    <td>HTML</td>
    <td>CSS</td>
  </tr>
  <tr>
    <th scope="row" rowspan="2">Tuesday</th>
    <td>JavaScript</td>
    <td rowspan="2">Project work</td>
  </tr>
  <tr>
    <td>Testing</td>
  </tr>
</table>
```

Output:

```text
Day        Sessions
Monday     HTML       CSS
Tuesday    JavaScript Project work
           Testing    (same merged cell)
```

Notice: when using `rowspan`, later rows have fewer cells — count carefully or the grid shifts and headers stop lining up.

---

# 4. scope and Table Accessibility

`scope="col"` marks a header for its column, `scope="row"` for its row, and `scope="colgroup"` / `scope="rowgroup"` for groups. This is how screen readers announce "January, HTML, 12" instead of "12".

```html
<table>
  <caption>Study hours by month</caption>
  <thead>
    <tr>
      <th scope="col">Month</th>
      <th scope="col">Hours</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">January</th>
      <td>42</td>
    </tr>
  </tbody>
</table>
```

Output:

```text
Screen reader on "42": "January, Hours, 42"
```

A11y checklist:

- Use `th` for headers, not bold `td`.
- Add `scope` to every header.
- Give the table a `caption`.
- Don't use tables for layout.
- Keep merged cells logical.

---

# 5. colgroup, col and Responsive Tables

`colgroup` and `col` apply column-level semantics or styling without repeating classes on every cell. For narrow screens, wrap the table in a scroll container instead of destroying the table semantics.

```html
<div class="table-scroll">
  <table>
    <caption>Kakade's progress</caption>
    <colgroup>
      <col class="topic-col" />
      <col class="score-col" />
    </colgroup>
    <thead>
      <tr>
        <th scope="col">Topic</th>
        <th scope="col">Score</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>HTML</td>
        <td>92</td>
      </tr>
    </tbody>
  </table>
</div>
```

Output:

```text
Wide screen: full table
Narrow screen: table scrolls horizontally inside its container
```

Bad:

```css
/* Turning the table into blocks kills row/column semantics */
table, tr, td { display: block; }
```

Good:

```css
.table-scroll { overflow-x: auto; }
```

Interview trap: `display: block` on table elements removes the table role in some browsers. Use `role="table"`/`role="row"`/`role="cell"` only as a last resort, or keep native display and scroll.

---

# Real-World Uses

A pricing table uses a caption, header row, row headers, and a scroll container on mobile.

```html
<div class="table-scroll">
  <table>
    <caption>Kakade.dev plans</caption>
    <thead>
      <tr>
        <th scope="col">Plan</th>
        <th scope="col">Price</th>
        <th scope="col">Seats</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <th scope="row">Free</th>
        <td>₹0</td>
        <td>1</td>
      </tr>
      <tr>
        <th scope="row">Pro</th>
        <td>₹499</td>
        <td>5</td>
      </tr>
    </tbody>
    <tfoot>
      <tr>
        <th scope="row">All plans</th>
        <td colspan="2">Cancel anytime</td>
      </tr>
    </tfoot>
  </table>
</div>
```

Output:

```text
Kakade.dev plans
Plan   Price   Seats
Free   ₹0      1
Pro    ₹499    5
All plans      Cancel anytime
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **table** | Tabular data only, not layout |
| **caption** | First child; accessible name |
| **thead, tbody, tfoot** | Header, body, summary groups |
| **tr, th, td** | Row, header cell, data cell |
| **colspan and rowspan** | Merge horizontally / vertically |
| **scope** | `col` / `row` header association |
| **colgroup and col** | Column-level grouping and styling |
| **Table accessibility** | Caption + `th` + `scope` |
| **Responsive tables** | Scroll container, keep display |

## The Mental Model

```text
table
  ├── caption → accessible name
  ├── colgroup → column definitions
  ├── thead → th scope="col"
  ├── tbody → th scope="row" + td
  └── tfoot → totals
        │
        └── merges: colspan (across), rowspan (down)
```

### The 3 things to remember

**1. Caption and headers:**

```html
<table>
  <caption>Kakade's progress</caption>
  <tr><th scope="col">Topic</th><th scope="col">Score</th></tr>
</table>
```

**2. Merge with care:**

```html
<td colspan="2">Cancel anytime</td>
```

**3. Scroll, don't restructure:**

```css
.table-scroll { overflow-x: auto; }
```

> **Tables encode relationships — caption, `th` with `scope`, and grouped rows are what make the grid readable to everyone.**
