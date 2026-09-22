# Strings — Revision Notes (Intermediate)
> One-line mental model: Immutable Unicode sequence — index/slice to read, methods to build new strings.

## Core recall
- Creation: `"Kakade"` / `'Kakade'` equal; embed quotes via alternation or `\`; `len("")==0`; Unicode native.
- Indexing: `s[0]`, `s[-1]` last char; out-of-range → `IndexError`.
- Slicing `s[start:stop:step]`, stop excluded; `s[:3]`, `s[3:]`, `s[:]`, `s[::2]`, `s[::-1]` reverse.
- Immutable: `s[0]="J"` → `TypeError`; methods return new strings — must reassign (`s = s.upper()`).
- Methods: `upper/lower/casefold`, `strip/lstrip/rstrip` (edges only), `replace(old,new[,count])`, `split(sep[,maxsplit])` → list, `sep.join(iter)` (separator joins), `startswith/endswith` (tuple allowed).
- Formatting: prefer f-strings `f"{name}"`; `{price:.2f}`, `{n:,}`, `{s:>10/<10/^10}`, debug `f"{name=}"`; `%` legacy, `.format()` ok.
- Escapes: `\n \t \\ \' \"`; raw `r"C:\Users\Kakade"` for paths/regex; triple `"""` + f-string for multiline reports.

## Must-remember code
```python
s = "Kakade"
print(s[0], s[-1], s[0:3], s[::-1])  # K e Kak edakaK
```
```python
username = "  Kakade  "
print(username.strip().lower())  # kakade
print("a-b-c-d".split("-", 2))  # ['a','b','c-d']
print(", ".join(["Python", "Kakade"]))  # Python, Kakade
```
```python
name, price = "Kakade", 1234567.891
print(f"₹{price:,.2f} {name=}")  # ₹1,234,567.89 name='Kakade'
print("photo.png".endswith((".jpg", ".png")))  # True
```
```python
text = "  hello   world  "
print(" ".join(text.split()))  # hello world (normalize spaces)
data = "Kakade,25,Developer"
n, a, r = data.split(","); print(n, int(a), r)
```

## Interview traps
- Forget reassign: `s.replace(...)` alone discards result → `s = s.replace(...)`.
- `strip("abc")` strips char-set from edges, not substring; inner spaces untouched — use `" ".join(s.split())` to collapse.
- `join` is `SEP.join(list)`, items must be `str` → `",".join(map(str,[1,2]))`; `",".join([1,2])` → `TypeError`.
- `split()` (no arg) splits any whitespace & collapses multiples; `split(" ")` does not.
- Slicing stop excluded: `"Python"[0:3]=="Pyt"`; negative step needed for reverse.

## 3 things to remember
1. Immutable — every transform creates a new string; reassign.
2. `split` ⇄ `join` are inverses; `join` lives on the separator.
3. Clean input with `strip().lower()`; format with f-strings.
