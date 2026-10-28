# Tables — Revision Notes (Beginner)
> One-line mental model: tables encode row-column relationships — caption, grouped rows, and `th` with `scope` make that relationship readable.

## Core recall
- Tables are for tabular data only; layout belongs to CSS grid/flexbox.
- `caption` is the table's accessible name and must be the first child.
- `thead` = header rows, `tbody` = data rows, `tfoot` = summary/total rows.
- `tr` row, `th` header cell, `td` data cell; headers must be `th`, never bold `td`.
- `colspan` merges across columns, `rowspan` down rows; later rows then hold fewer cells.
- `scope="col"` / `"row"` (also `colgroup`, `rowgroup`) associates headers with cells for screen readers.
- `colgroup` + `col` apply column-level grouping/styling.
- Responsive tables: wrap in `overflow-x: auto`; avoid `display: block` on table elements.
- A11y baseline: caption + `th` + `scope` + logical merged cells.

## Must-remember code
```html
<table>
  <caption>Kakade's study hours</caption>
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
```html
<tr>
  <th scope="col" colspan="2">Sessions</th>
</tr>
<tr>
  <th scope="row" rowspan="2">Tuesday</th>
  <td>JavaScript</td>
</tr>
```
```html
<div class="table-scroll">
  <table>...</table>
</div>
<style>
  .table-scroll { overflow-x: auto; }
</style>
```

## Interview traps
- Using tables for page layout.
- Missing `caption` and `scope`.
- Bold `td` instead of `th` — no header semantics.
- Forgetting that `rowspan`/`colspan` reduce cells in later rows.
- `display: block` on table elements — roles and header associations disappear.
- Multiple `tbody` without grouping logic; `tfoot` placed before `tbody` for styling reasons.

## 3 things to remember
1. `caption` + `th` + `scope` = accessible table.
2. `colspan` merges across, `rowspan` merges down — count the grid.
3. Scroll for mobile; never destroy native table display.
