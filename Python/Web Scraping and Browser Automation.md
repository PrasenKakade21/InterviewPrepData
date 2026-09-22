# Web Scraping and Browser Automation

Static pages need only **requests + BeautifulSoup**. JavaScript-heavy pages need a real browser — **Playwright**.

The core idea:

```text
Static HTML → requests + parse
    ↓
Dynamic JS → Playwright drives browser
    ↓
extract → paginate → store
```

Install:

```bash
pip install requests beautifulsoup4 playwright
playwright install chromium
```

---

# 1. HTTP Requests

Scraping starts with a GET + headers. Sites block bare scripts.

```python
import requests

r = requests.get(
    "https://example.com",
    headers={"User-Agent": "KakadeBot/1.0"},
    timeout=10,
)

print(r.status_code)
print("Example Domain" in r.text)
```

Output:

```text
200
True
```

Always set `User-Agent` + `timeout`. Check `robots.txt` and terms before scraping.

---

# 2. HTML Parsing

`BeautifulSoup` turns HTML string into a searchable tree.

```python
from bs4 import BeautifulSoup

html = "<h1>Hello Kakade</h1><p>Prep notes</p>"
soup = BeautifulSoup(html, "html.parser")

print(soup.h1.text)
print(soup.p.text)
```

Output:

```text
Hello Kakade
Prep notes
```

Think:

```text
requests → raw HTML string
BeautifulSoup → tag tree you can query
```

---

# 3. CSS Selectors

`select()` uses CSS syntax — same as browser DevTools.

```python
from bs4 import BeautifulSoup

html = "<ul><li class='user'>Kakade</li><li class='user'>Asha</li></ul>"
soup = BeautifulSoup(html, "html.parser")

print([li.text for li in soup.select("li.user")])
print(soup.select_one("li.user").text)
```

Output:

```text
['Kakade', 'Asha']
Kakade
```

| Selector | Meaning |
| -------- | ------- |
| `.user` | class |
| `#main` | id |
| `div a` | links inside div |

Copy selector from DevTools → paste into `select()`.

---

# 4. Finding Elements

Two APIs, same result: `find/find_all` (Pythonic) vs `select` (CSS).

```python
from bs4 import BeautifulSoup

html = "<div><a href='/u/1'>Kakade</a><a href='/u/2'>Asha</a></div>"
soup = BeautifulSoup(html, "html.parser")

print(soup.find("a").text)
print([a.text for a in soup.find_all("a")])
```

Output:

```text
Kakade
['Kakade', 'Asha']
```

Prefer `select` for complex nesting, `find` for quick single-tag grabs.

---

# 5. Extracting Text

Clean whitespace — sites pad with `\n` and spaces.

```python
from bs4 import BeautifulSoup

html = "<div>  Hello <b>Kakade</b>  \n </div>"
soup = BeautifulSoup(html, "html.parser")

print(repr(soup.div.text))
print(repr(soup.div.get_text(strip=True)))
```

Output:

```text
'  Hello Kakade  \n '
'Hello Kakade'
```

`get_text(strip=True)` is the default cleaner for every scraper.

---

# 6. Extracting Links

```python
from bs4 import BeautifulSoup

html = "<a href='/users/kakade'>Kakade</a>"
soup = BeautifulSoup(html, "html.parser")
a = soup.a

print(a["href"])
print(a.text)
```

Output:

```text
/users/kakade
Kakade
```

Make absolute with `urljoin`:

```python
from urllib.parse import urljoin

print(urljoin("https://example.com", "/users/kakade"))
```

Output:

```text
https://example.com/users/kakade
```

---

# 7. Extracting Attributes

Any attribute is dict access on the tag.

```python
from bs4 import BeautifulSoup

html = "<img src='kakade.jpg' alt='Kakade photo' data-id='5'>"
img = BeautifulSoup(html, "html.parser").img

print(img["src"])
print(img.get("alt"))
print(img.get("data-id"))
```

Output:

```text
kakade.jpg
Kakade photo
5
```

Use `.get()` for optional attributes — returns `None` instead of crashing.

---

# 8. Handling Pagination

Most listings split across `?page=1,2,3...`. Loop until empty.

```python
import requests
from bs4 import BeautifulSoup

def get_titles(page: int):
    # demo: parse local HTML instead of hitting network
    html = f"<h2>Post {page}a</h2><h2>Post {page}b</h2>" if page <= 2 else ""
    return [h.text for h in BeautifulSoup(html, "html.parser").select("h2")]

all_titles = []
page = 1
while True:
    titles = get_titles(page)
    if not titles:
        break
    all_titles.extend(titles)
    page += 1

print(all_titles)
```

Output:

```text
['Post 1a', 'Post 1b', 'Post 2a', 'Post 2b']
```

Real loop: `requests.get(url, params={"page": page})` + polite `time.sleep(1)` between pages.

---

# 9. Browser Automation

Playwright drives a real browser — clicks, forms, JS rendering.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("https://example.com")
    print(page.title())
    browser.close()
```

Output:

```text
Example Domain
```

Use when `requests` returns empty shells (`<div id="root">` with no data = needs JS).

---

# 10. Launching Browsers

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    context = browser.new_context(user_agent="KakadeBot/1.0")
    page = context.new_page()
    page.goto("https://example.com")
    print(page.url)
    browser.close()
```

Output:

```text
https://example.com/
```

`headless=False` to watch it work while debugging. Contexts isolate cookies like incognito profiles.

---

# 11. Locators

Locators auto-retry until element appears — prefer over `query_selector`.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.set_content("<button>Login as Kakade</button>")
    print(page.get_by_role("button", name="Login as Kakade").text_content())
    browser.close()
```

Output:

```text
Login as Kakade
```

| Locator | Use |
| ------- | --- |
| `get_by_role()` | Buttons, headings |
| `get_by_text()` | Visible text |
| `locator("#id")` | CSS fallback |

---

# 12. Clicking Elements

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.set_content("<button onclick=\"this.textContent='Clicked'\">Go</button>")
    page.get_by_role("button", name="Go").click()
    print(page.get_by_role("button").text_content())
    browser.close()
```

Output:

```text
Clicked
```

Click auto-waits for visibility. Add `expect_navigation` wrapper when click loads a new page.

---

# 13. Typing into Forms

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.set_content("<input name='user' placeholder='name'>")
    page.get_by_placeholder("name").fill("Kakade")
    print(page.input_value("input[name=user]"))
    browser.close()
```

Output:

```text
Kakade
```

`fill()` clears then types. `type()` simulates keystrokes (slower, triggers key events).

---

# 14. Login Flows

Standard pattern: fill → click → wait for post-login element.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.set_content(
        "<input name='u'><input name='p' type='password'>"
        "<button onclick=\"document.body.innerHTML='<h1>Hi Kakade</h1>'\">Login</button>"
    )
    page.fill("input[name=u]", "kakade")
    page.fill("input[name=p]", "secret")
    page.get_by_role("button", name="Login").click()
    page.wait_for_selector("h1")
    print(page.get_by_role("heading").text_content())
    browser.close()
```

Output:

```text
Hi Kakade
```

Save session with `context.storage_state(path="auth.json")` to skip login next run.

---

# 15. Waiting for Elements

Never `time.sleep()` — wait for the thing you need.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.set_content("<h1>Hi Kakade</h1>")
    page.wait_for_selector("h1")
    print("ready")
    browser.close()
```

Output:

```text
ready
```

| Wait | Use |
| ---- | --- |
| `wait_for_selector()` | Element exists |
| `expect_response()` | API call finished |
| `wait_for_load_state("networkidle")` | All requests done |

---

# 16. Screenshots

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.set_content("<h1>Report for Kakade</h1>")
    page.screenshot(path="demo_data/shot.png")
    print("saved")
    browser.close()
```

Output:

```text
saved
```

Full page: `page.screenshot(path="full.png", full_page=True)`. Element-only: `locator("table").screenshot()`.

---

# 17. Page Navigation

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("https://example.com")
    print(page.url)
    page.go_back()
    page.go_forward()
    page.reload()
    browser.close()
```

Output:

```text
https://example.com/
```

`page.goto(url, wait_until="domcontentloaded")` for speed, `"networkidle"` for completeness.

---

# 18. Extracting Dynamic Content

Wait for JS, then read rendered DOM.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.set_content("<ul><li>Kakade</li><li>Asha</li></ul>")
    print(page.locator("li").all_text_contents())
    browser.close()
```

Output:

```text
['Kakade', 'Asha']
```

Pipeline: `wait_for_selector` → `all_text_contents()` → save to CSV with `pandas`.

---

# Real-World Uses

```python
Price tracker → requests poll → BeautifulSoup parse → pandas CSV → Playwright fallback for JS pages
```

Job board digest, login-and-download statements, screenshot regression — same extract-store loop.

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **HTTP requests** | `User-Agent` + `timeout` |
| **HTML parsing** | `BeautifulSoup(html, "html.parser")` |
| **CSS selectors** | `select()` / `select_one()` |
| **Finding** | `find` vs `select` |
| **Text** | `get_text(strip=True)` |
| **Links** | `a["href"]` + `urljoin` |
| **Attributes** | `tag.get()` safe access |
| **Pagination** | Loop `?page=` until empty |
| **Browser automation** | Real browser for JS |
| **Launching** | `chromium.launch(headless=)` |
| **Locators** | `get_by_role/text` auto-retry |
| **Clicking** | Auto-waits |
| **Typing** | `fill()` vs `type()` |
| **Login** | Fill → click → wait + `storage_state` |
| **Waiting** | `wait_for_selector`, not sleep |
| **Screenshots** | `page/element.screenshot()` |
| **Navigation** | `goto/go_back/reload` |
| **Dynamic content** | Wait then `all_text_contents()` |

## The Mental Model

```text
        STATIC?
       yes → requests + BeautifulSoup
        │      select → text/links → paginate
        │
        no (JS)
        ↓
     PLAYWRIGHT
 launch → goto → locator
   │        │        │
   ↓        ↓        ↓
 fill/click → wait → extract
   │                 │
   ↓                 ↓
 login state    screenshot / CSV
```

### The 3 things to remember

**1. Static scrape:**

```python
soup = BeautifulSoup(requests.get(url, timeout=10).text, "html.parser")
soup.select("li.user")
```

**2. Clean text:**

```python
tag.get_text(strip=True)
```

**3. Dynamic fallback:**

```python
with sync_playwright() as p:
    page = p.chromium.launch().new_page()
    page.goto(url)
    page.wait_for_selector("li")
```

> **Scrape static HTML with requests plus selectors; drive dynamic pages with Playwright locators, explicit waits, and saved login state.**
