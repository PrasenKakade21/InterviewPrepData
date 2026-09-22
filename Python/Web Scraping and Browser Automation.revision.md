# Web Scraping and Browser Automation — Revision Notes (Intermediate)
> One-line mental model: Static HTML → requests + selectors; dynamic JS → Playwright locators + waits.
## Core recall
- `requests.get(url, headers={"User-Agent": ...}, timeout=10)`; check `robots.txt`, status codes.
- `BeautifulSoup(html, "html.parser")` → tag tree; `find`/`find_all` vs `select`/`select_one`.
- Selectors: `.user` class, `#main` id, `div a` descendant; copy from DevTools.
- Text: `tag.get_text(strip=True)`; links: `a["href"]` + `urljoin(base, href)`; attrs: `tag.get("src")`.
- Pagination: loop `?page=N` with `params`, `time.sleep(1)`, stop on empty.
- Playwright: `sync_playwright() → chromium.launch(headless=) → new_page() → goto()`.
- Locators auto-retry: `get_by_role`, `get_by_text`, `locator("#id")`; `fill()` vs `type()`.
- Login: fill → click → `wait_for_selector` + `storage_state("auth.json")`; never `time.sleep`.
- Nav/shots: `goto/go_back/reload`, `screenshot(full_page=True)`, `all_text_contents()` → CSV.
## Must-remember code
- ```python
  import requests
  from bs4 import BeautifulSoup
  r = requests.get("https://example.com", headers={"User-Agent": "KakadeBot/1.0"}, timeout=10)
  soup = BeautifulSoup(r.text, "html.parser")
  print([li.get_text(strip=True) for li in soup.select("li.user")])
  ```
- ```python
  from urllib.parse import urljoin
  print(urljoin("https://example.com", "/users/kakade"))
  ```
- ```python
  page = 1
  while titles := ["x"] if page <= 2 else []:
      print("Kakade page", page); page += 1
      if page > 2: break
  ```
- ```python
  from playwright.sync_api import sync_playwright
  with sync_playwright() as p:
      page = p.chromium.launch().new_page()
      page.set_content("<input name='user'>")
      page.fill("input[name=user]", "Kakade")
      print(page.input_value("input[name=user]"))
  ```
- ```python
  from playwright.sync_api import sync_playwright
  with sync_playwright() as p:
      page = p.chromium.launch().new_page()
      page.set_content("<ul><li>Kakade</li></ul>")
      page.wait_for_selector("li")
      print(page.locator("li").all_text_contents())
  ```
## Interview traps
- Bare `requests.get(url)` with no headers/timeout → blocked/hangs → always set both.
- `soup.div.text` keeps `\n` spaces → use `get_text(strip=True)`.
- `tag["data-id"]` on missing attr → KeyError → use `tag.get("data-id")`.
- `time.sleep(2)` for JS → flaky → use `wait_for_selector` / `expect_response`.
- `query_selector` once vs locator → misses late render → prefer `get_by_role`/`get_by_text`.
## 3 things to remember
1. Static scrape = requests + `select()` + `get_text(strip=True)` + `urljoin`.
2. Paginate `?page=` until empty with polite delay.
3. Dynamic = Playwright launch → locator → fill/click → wait → extract + save state.
