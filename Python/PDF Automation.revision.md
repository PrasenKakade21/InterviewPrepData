# PDF Automation — Revision Notes (Intermediate)
> One-line mental model: Read with PyMuPDF → reshape with pypdf → create with reportlab.
## Core recall
- Tools: PyMuPDF (`fitz`) fast `get_text()`/search; pypdf merge/split/metadata; reportlab new PDFs.
- Extract: `doc = fitz.open("r.pdf"); doc.page_count; doc[0].get_text()`; scanned `""` → OCR/Tesseract.
- Metadata: `PdfReader(...).metadata.title`, `len(reader.pages)`; `PdfWriter.add_metadata({"/Title":...})`.
- Merge: `PdfWriter()` + `add_page(reader.pages[i])` loop → `writer.write(f)`; monthly reports combine.
- Split: one `PdfWriter` per output; `for i,page in enumerate(reader.pages)` per-page files.
- Create: `Canvas("o.pdf"); drawString(x,y,text); setFont; line(); save()`; origin bottom-left points.
- Tables: `reportlab.platypus` (`SimpleDocTemplate`, `Table`, `Paragraph`) for real layouts.
- Report pipe: pandas aggregate → reportlab render rows → pypdf merge cover/terms.
## Must-remember code
```python
import fitz
doc = fitz.open("demo_data/hello.pdf")
print(doc.page_count, doc[0].get_text().strip())
```
```python
from pypdf import PdfWriter, PdfReader
w = PdfWriter()
w.add_page(PdfReader("demo_data/a.pdf").pages[0])
with open("demo_data/merged.pdf", "wb") as f: w.write(f)
```
```python
from reportlab.pdfgen import canvas
c = canvas.Canvas("demo_data/hello.pdf")
c.drawString(100, 750, "Hello Kakade"); c.save()
```
```python
import pandas as pd
from reportlab.pdfgen import canvas
df = pd.DataFrame([{"name": "Kakade", "amount": 5000}])
c = canvas.Canvas("demo_data/summary.pdf")
y = 750
for _, r in df.iterrows(): c.drawString(100, y, f"{r['name']}: Rs {r['amount']}"); y -= 20
c.save()
```
## Interview traps
- `page.get_text()==""` ≠ empty PDF → scanned image needs OCR, mention Tesseract.
- reportlab `Canvas` low-level only → tables need `platypus`, not manual drawString loops.
- Forgetting `save()`/`write(f)` → 0-byte PDF; coords origin bottom-left, y=750 top.
- pypdf can't render text well → don't use for extraction, use PyMuPDF.
- Overwriting source while merging → write new file, then replace.
## 3 things to remember
1. Extract: `fitz.open()[0].get_text()`.
2. Merge/split: `PdfWriter.add_page` loop.
3. Create: `Canvas.drawString + save`; pandas → reportlab → merge.
