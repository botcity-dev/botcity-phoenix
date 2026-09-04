# UiPath Actions → Python Equivalence

Reference table consulted during Step 3 of the `botcity-phoenix` skill.
Keep this file open while performing the transcription.

## 1. Web UI Automation (UiPath.UIAutomation / UiPath.Web)

| UiPath Action | Python Equivalent | Suggested Stable Version |
|---|---|---|
| Open Browser / Use Application Browser | `selenium` (webdriver) or `playwright` | selenium ≥ 4.27.0 · playwright ≥ 1.49 |
| Click / Type Into / Hover | `selenium.webdriver.ActionChains` · `page.click()` / `page.fill()` in Playwright | — |
| Get Text / Get Attribute | `element.text` / `element.get_attribute()` (Selenium) · `page.text_content()` (Playwright) | — |
| Find Element / Find Children | `find_element(By.XPATH, ...)` · `page.locator()` | — |
| Wait Element Appear / Vanish | `WebDriverWait` + `expected_conditions` · `page.wait_for_selector()` | — |
| Extract Structured Data / Data Scraping | `beautifulsoup4` + `lxml`, or `scrapy` for large volumes | bs4 ≥ 4.12.3 · scrapy ≥ 2.12 |
| Inject JS / Execute Script | `driver.execute_script()` · `page.evaluate()` | — |
| Web Automation with Computer Vision | `botcity-framework-web` | ≥ 0.20.0 |

- Basic setup using BotCity's web framework, ask user if it wants a proper python library to manage webdriver and ask it which Web Browser will be used:

```
def init_webbot():
    STATE.webbot = WebBot()
    bot = STATE.webbot
    bot.browser = Browser.CHROME
    bot.driver_path = ChromeDriverManager().install()
    bot.headless = False
    bot.browse(RPA_CHALLENGE_URL)
    bot.find_element(START_BUTTON_SELECTOR, By.CSS_SELECTOR, ensure_clickable=True).click()

```

## 2. Desktop Automation (UiPath.UIAutomation Desktop)

| UiPath Action | Python Equivalent | Suggested Stable Version |
|---|---|---|
| Click / Type Into / Send Hotkey (desktop) | `pyautogui`, `pynput` | pyautogui ≥ 0.9.54 · pynput ≥ 1.7.7 |
| Window automation (native Windows) | `pywinauto` (control via Win32/UIA API) | ≥ 0.6.8 |
| Image-based automation / Computer Vision Activities | `botcity-framework-core` (combines template matching + OCR) | ≥ 0.20.0 |
| Get OCR Text / Screen OCR | `pytesseract` (Tesseract wrapper) or `easyocr` | pytesseract ≥ 0.3.13 · easyocr ≥ 1.7.2 |
| Take Screenshot | `pyautogui.screenshot()` or `mss` (faster) | mss ≥ 9.0.1 |
| Mouse Move / Scroll | `pyautogui.moveTo()`, `pyautogui.scroll()` | — |

## 3. Excel / Spreadsheets (UiPath.Excel.Activities)

| UiPath Action | Python Equivalent | Suggested Stable Version |
|---|---|---|
| Excel Application Scope / Use Excel File | `openpyxl` (≥ 3.1.5) for pure `.xlsx` · `xlwings` when you need Excel open with macros | openpyxl ≥ 3.1.5 · xlwings ≥ 0.33.0 |
| Read Range / Write Range | `pandas.read_excel()` / `df.to_excel()` | pandas ≥ 2.2.0 · openpyxl ≥ 3.1.5 |
| Read Cell / Write Cell | `ws["A1"].value` in `openpyxl` | — |
| Filter / Sort Table | `pandas` (`df.query()`, `df.sort_values()`) | — |
| Append Range | `pandas.concat()` + `to_excel()` or `openpyxl` in append mode | — |
| Pivot Table | `pandas.pivot_table()` | — |
| Format Cells (styles, colors) | `openpyxl.styles` | — |

## 4. PDF (UiPath.PDF.Activities)

| UiPath Action | Python Equivalent | Security Note |
|---|---|---|
| Read PDF Text | `pdfplumber` or `pypdf` ≥ 6.7.2 | `pypdf < 6.7.2` has DoS via infinite loop (CVE-2026-27628). Use ≥ 6.7.2 |
| Read PDF With OCR | `ocrmypdf` or `pdf2image` + `pytesseract` | ocrmypdf ≥ 16.0 · pdf2image ≥ 1.17 |
| Extract Images from PDF | `pikepdf` (based on qpdf, robust against malformed PDFs) | pikepdf ≥ 9.5 |
| Merge / Split PDF | `pypdf` ≥ 6.7.2 or `pikepdf` | pypdf ≥ 6.7.2 |
| Manipulate PDF (rotate, watermark) | `pypdf` ≥ 6.7.2 · `reportlab` to generate from scratch | reportlab ≥ 4.2 |
| Fill PDF Form | `pdfforms` or `pikepdf` | pdfforms ≥ 0.2.0 |

## 5. Email (UiPath.Mail.Activities)

| UiPath Action | Python Equivalent | Suggested Stable Version |
|---|---|---|
| Get IMAP Mail Messages | `imapclient` + `email` (stdlib) | imapclient ≥ 3.0.1 |
| Get Outlook Mail Messages | `pywin32` (`win32com.client`) on Windows with Outlook installed | pywin32 ≥ 308 |
| Send SMTP Mail Message | `smtplib` + `email.mime` (stdlib) — no external dependencies | — |
| Send Exchange / O365 | `O365` (package `python-o365`) via Microsoft Graph | python-o365 ≥ 2.0.34 |
| Save Attachments | iteration over `message.iter_attachments()` | — |

## 6. System, Files and Workflow (UiPath.System.Activities)

| UiPath Action | Python Equivalent |
|---|---|
| Copy File / Move File / Delete File | `shutil`, `os`, `pathlib` (stdlib) |
| Create Directory / Path Exists | `pathlib.Path.mkdir()`, `.exists()` |
| Read Text File / Write Text File | `pathlib.Path.read_text()` / `write_text()` |
| Wait / Delay | `time.sleep()` |
| If / Switch / For Each | native Python constructs |
| Try Catch | `try/except/finally` |
| Invoke Workflow / Invoke Code | `import` of modules or `subprocess.run()` |
| Log Message | `logging` (stdlib) |
| Get Environment Variable | `os.environ` or `python-dotenv` for `.env` files |
| Run Python Script | native |

## 7. Data and Programming (UiPath.System – Data activities)

| UiPath Action | Python Equivalent | Suggested Stable Version |
|---|---|---|
| Build Data Table / Filter Data Table | `pandas.DataFrame` | pandas ≥ 2.2.0 |
| For Each Row in Data Table | `df.iterrows()` or `df.itertuples()` (faster) | — |
| Merge Data Table / Join | `pandas.merge()`, `pandas.concat()` | — |
| Output Data Table (CSV) | `df.to_csv()` | — |
| String manipulation (Trim, Split, Replace) | native `str` methods | — |
| Regex (Matches, Replace) | `re` module (stdlib) | — |
| Date manipulation | `datetime` (stdlib) or `pendulum` | pendulum ≥ 3.0 |

## 8. HTTP / APIs (UiPath.WebAPI.Activities)

| UiPath Action | Python Equivalent | Suggested Stable Version |
|---|---|---|
| HTTP Request | `httpx` (recommended — supports async/HTTP-2) or `requests` | httpx ≥ 0.28 · requests ≥ 2.32 |
| Deserialize JSON | `json.loads()` (stdlib) | — |
| SOAP Request | `zeep` | zeep ≥ 4.3 |
| OAuth2 | `authlib` or `requests-oauthlib` | authlib ≥ 1.4 · requests-oauthlib ≥ 1.4 |

## 9. Database (UiPath.Database.Activities)

| UiPath Action | Python Equivalent | Suggested Stable Version |
|---|---|---|
| Connect / Execute Query / Execute Non-Query | `sqlalchemy` (universal engine) | sqlalchemy ≥ 2.0.36 |
| SQL Server | `pyodbc` or `pymssql` | pyodbc ≥ 5.2 · pymssql ≥ 2.3 |
| Oracle | `oracledb` (replaced cx_Oracle) | oracledb ≥ 2.5 |
| PostgreSQL | `psycopg` (v3) | psycopg ≥ 3.2 |
| MySQL | `mysqlclient` or `mysql-connector-python` | mysqlclient ≥ 2.3 · mysql-connector-python ≥ 9.0 |
| Full ORM | `sqlalchemy` 2.x or `SQLModel` | SQLModel ≥ 0.0.22 |

## 10. OCR and Document Understanding (UiPath.DocumentUnderstanding)

| UiPath Action | Python Equivalent | Suggested Stable Version |
|---|---|---|
| Digitize Document | `pdfplumber` + `pytesseract` or `easyocr` | — |
| Classify Document | `transformers` (LayoutLM, Donut models) | transformers ≥ 4.48 |
| Extract Data (forms / invoices) | `donut-python`, `layoutparser`, or APIs (Azure Document Intelligence, AWS Textract) | layoutparser ≥ 0.3.4 |
| Validate Extraction Results | pure Python logic + `pydantic` for schema validation | pydantic ≥ 2.10 |

## 11. Enterprise System Integrations (Integration Service)

| UiPath Action | Python Equivalent | Suggested Stable Version |
|---|---|---|
| SAP Integration | `pyrfc` (SAP NetWeaver RFC) or UI automation via `botcity-framework-core` | pyrfc ≥ 2.5 |
| Active Directory | `ldap3` | ldap3 ≥ 2.9 |
| AWS | `boto3` | boto3 ≥ 1.36 |
| Azure | `azure-sdk-for-python` (various subpackages) | — |
| Google Workspace | `google-api-python-client` | google-api-python-client ≥ 2.154 |
| Salesforce | `simple-salesforce` | simple-salesforce ≥ 1.12 |
| ServiceNow | `pysnow` | pysnow ≥ 0.9 |
| Slack | `slack-sdk` | slack-sdk ≥ 3.35 |
| Microsoft Teams / Graph | `msgraph-sdk` | msgraph-sdk ≥ 1.0 |

## 12. Orchestration (UiPath Orchestrator)

| UiPath Functionality | Python Stack Equivalent |
|---|---|
| Orchestrator (queues, assets, jobs, logs) | **BotCity Maestro** (native Python orchestrator) |
| Bot scheduling | BotCity Maestro SDK or API |
| Credentials / Assets | `keyring` (local vault) + Maestro for enterprise |
| Work queue | Maestro queues, or `celery` + `redis` for high volumes |