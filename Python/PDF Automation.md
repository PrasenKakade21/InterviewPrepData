# PDF Automation

PDFs look opaque, but automation is just three verbs: **read text → reshape pages → write new PDF**.

The core idea:

```text
Input PDFs
    ↓
extract / merge / split
    ↓
reportlab creates new PDF
```

Install:

```bash
pip install pypdf reportlab pymupdf
```

---

# 1. PyMuPDF

`PyMuPDF` (`fitz`) is the fastest text extractor, especially for scanned-layout PDFs.

```python
import fitz

doc = fitz.open("demo_data/sample.pdf") if False else None

print(fitz.__doc__)
```

Output:

```text
PyMuPDF - fast PDF rendering and text extraction
```

Real usage:

```python
doc = fitz.open("demo_data/report.pdf")
print(doc.page_count)
print(doc[0].get_text()[:100])
```

Rule: `PyMuPDF` for reading/searching, `pypdf` for merging/splitting, `reportlab` for creating.

---

# 2. pypdf

`pypdf` edits structure — merge, split, rotate — without rendering.

```python
import pypdf

reader = pypdf.PdfReader("demo_data/styled.pdf") if False else None

print(pypdf.__version__)
```

Output:

```text
6.4.0
```

Think:

```text
PyMuPDF → what does it say?
pypdf → how are pages arranged?
reportlab → make a new one
```

---

# 3. reportlab

`reportlab` builds PDFs from code — invoices, certificates, reports.

```python
from reportlab.pdfgen import canvas

c = canvas.Canvas("demo_data/hello.pdf")
c.drawString(100, 750, "Hello Kakade")
c.save()

print("created")
```

Output:

```text
created
```

Coordinates: `(0,0)` bottom-left, units in points. `drawString(x, y, text)` places text.

---

# 4. Extracting PDF Text

```python
from reportlab.pdfgen import canvas

c = canvas.Canvas("demo_data/hello.pdf")
c.drawString(100, 750, "Hello Kakade Report")
c.save()

import fitz
doc = fitz.open("demo_data/hello.pdf")

print(doc[0].get_text().strip())
```

Output:

```text
Hello Kakade Report
```

Pattern for scanned PDFs: `page.get_text()` returns `""` → fall back to OCR (out of scope, mention Tesseract in interviews).

---

# 5. Reading PDF Metadata

```python
from pypdf import PdfReader

reader = PdfReader("demo_data/hello.pdf")

print(reader.metadata.title)
print(len(reader.pages))
```

Output:

```text
None
1
```

Set metadata at creation:

```python
from pypdf import PdfReader, PdfWriter

writer = PdfWriter()
writer.add_page(PdfReader("demo_data/hello.pdf").pages[0])
writer.add_metadata({"/Title": "Kakade Report", "/Author": "Kakade"})

with open("demo_data/meta.pdf", "wb") as f:
    writer.write(f)
```

---

# 6. Merging PDFs

Combine monthly reports into one file.

```python
from pypdf import PdfWriter, PdfReader
from reportlab.pdfgen import canvas

for name, text in [("a.pdf", "Page Kakade"), ("b.pdf", "Page Asha")]:
    c = canvas.Canvas(f"demo_data/{name}")
    c.drawString(100, 750, text)
    c.save()

writer = PdfWriter()
for name in ["a.pdf", "b.pdf"]:
    writer.add_page(PdfReader(f"demo_data/{name}").pages[0])

with open("demo_data/merged.pdf", "wb") as f:
    writer.write(f)

print(len(PdfReader("demo_data/merged.pdf").pages))
```

Output:

```text
2
```

---

# 7. Splitting PDFs

Opposite: one file → many.

```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader("demo_data/merged.pdf")
writer = PdfWriter()
writer.add_page(reader.pages[0])

with open("demo_data/first_only.pdf", "wb") as f:
    writer.write(f)

print(len(PdfReader("demo_data/first_only.pdf").pages))
```

Output:

```text
1
```

Loop pattern: `for i, page in enumerate(reader.pages)` → one writer per page for per-page split.

---

# 8. Creating PDFs

Slightly richer invoice with lines and font sizes:

```python
from reportlab.pdfgen import canvas

c = canvas.Canvas("demo_data/invoice.pdf")
c.setFont("Helvetica-Bold", 16)
c.drawString(100, 750, "Invoice — Kakade Dev")
c.setFont("Helvetica", 12)
c.drawString(100, 720, "Item: Automation Report — Rs 5000")
c.line(100, 710, 400, 710)
c.save()

print("invoice created")
```

Output:

```text
invoice created
```

For tables, use `reportlab.platypus` (`SimpleDocTemplate`, `Table`, `Paragraph`) — mention by name in interviews.

---

# 9. Generating Reports

Data → PDF pipeline:

```python
import pandas as pd
from reportlab.pdfgen import canvas

df = pd.DataFrame([
    {"name": "Kakade", "amount": 5000},
    {"name": "Asha", "amount": 3000},
])

c = canvas.Canvas("demo_data/summary.pdf")
c.setFont("Helvetica-Bold", 14)
c.drawString(100, 750, "Sales Summary")
y = 720
for _, row in df.iterrows():
    c.setFont("Helvetica", 12)
    c.drawString(100, y, f"{row['name']}: Rs {row['amount']}")
    y -= 20
c.save()

print(f"rows: {len(df)}")
```

Output:

```text
rows: 2
```

Real jobs: `pandas` aggregates → `reportlab` renders → `pypdf` merges cover page.

---

# Real-World Uses

```python
Invoices folder → extract totals → pandas summary → reportlab PDF → merge with terms.pdf
```

Resume parser, invoice digest, certificate bulk-generator — same extract→create flow.

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **PyMuPDF** | Fast `get_text()`, `page_count` |
| **pypdf** | Merge/split/metadata |
| **reportlab** | `Canvas`, `drawString`, `save` |
| **Extract text** | `doc[0].get_text()` |
| **Metadata** | `reader.metadata`, `add_metadata` |
| **Merging** | `PdfWriter.add_page` loop |
| **Splitting** | One writer per output |
| **Creating** | Fonts, lines, coords |
| **Reports** | pandas → reportlab → merge |

## The Mental Model

```text
        READ
  PyMuPDF get_text / metadata
            │
     ┌──────┴──────┐
     ↓             ↓
   MERGE         SPLIT
 pypdf writer   per-page writer
     │             │
     └──────┬──────┘
            ↓
         CREATE
   reportlab Canvas → PDF
            │
            ↓
   pandas summary → report
```

### The 3 things to remember

**1. Extract:**

```python
import fitz
doc = fitz.open("demo_data/hello.pdf")
doc[0].get_text()
```

**2. Merge:**

```python
from pypdf import PdfWriter, PdfReader
w = PdfWriter()
w.add_page(PdfReader("demo_data/a.pdf").pages[0])
```

**3. Create:**

```python
from reportlab.pdfgen import canvas
c = canvas.Canvas("out.pdf")
c.drawString(100, 750, "Hello Kakade")
c.save()
```

> **PDF automation is extract-reshape-create: read with PyMuPDF, merge/split with pypdf, generate fresh reports with reportlab.**
