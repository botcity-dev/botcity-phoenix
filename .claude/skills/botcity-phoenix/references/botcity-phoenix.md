# Equivalência de ações UiPath → Python

Tabela de referência consultada durante a Etapa 3 da skill `botcity-phoenix`.
Mantenha esse arquivo aberto enquanto faz a transcrição.

## 1. Automação de UI Web (UiPath.UIAutomation / UiPath.Web)

| Ação UiPath | Equivalente Python | Versão segura sugerida |
|---|---|---|
| Open Browser / Use Application Browser | `selenium` (webdriver) ou `playwright` | selenium ≥ 4.14.0 · playwright ≥ 1.50 |
| Click / Type Into / Hover | `selenium.webdriver.ActionChains` · `page.click()` / `page.fill()` no Playwright | — |
| Get Text / Get Attribute | `element.text` / `element.get_attribute()` (Selenium) · `page.text_content()` (Playwright) | — |
| Find Element / Find Children | `find_element(By.XPATH, ...)` · `page.locator()` | — |
| Wait Element Appear / Vanish | `WebDriverWait` + `expected_conditions` · `page.wait_for_selector()` | — |
| Extract Structured Data / Data Scraping | `beautifulsoup4` + `lxml`, ou `scrapy` para volumes grandes | bs4 ≥ 4.12 · scrapy ≥ 2.11 |
| Inject JS / Execute Script | `driver.execute_script()` · `page.evaluate()` | — |
| Web Automation com Computer Vision | `botcity-framework-web` | ≥ 0.13 |

## 2. Automação Desktop (UiPath.UIAutomation Desktop)

| Ação UiPath | Equivalente Python |
|---|---|
| Click / Type Into / Send Hotkey (desktop) | `pyautogui`, `pynput` |
| Window automation (Windows nativas) | `pywinauto` (controle por API Win32/UIA) |
| Image-based automation / Computer Vision Activities | `botcity-framework-core` (combina template matching + OCR) |
| Get OCR Text / Screen OCR | `pytesseract` (wrapper do Tesseract) ou `easyocr` |
| Take Screenshot | `pyautogui.screenshot()` ou `mss` (mais rápido) |
| Mouse Move / Scroll | `pyautogui.moveTo()`, `pyautogui.scroll()` |

## 3. Excel / Planilhas (UiPath.Excel.Activities)

| Ação UiPath | Equivalente Python |
|---|---|
| Excel Application Scope / Use Excel File | `openpyxl` (≥ 3.1.5) para `.xlsx` puro · `xlwings` quando precisa do Excel aberto com macros |
| Read Range / Write Range | `pandas.read_excel()` / `df.to_excel()` |
| Read Cell / Write Cell | `ws["A1"].value` no `openpyxl` |
| Filter / Sort Table | `pandas` (`df.query()`, `df.sort_values()`) |
| Append Range | `pandas.concat()` + `to_excel()` ou `openpyxl` em modo append |
| Pivot Table | `pandas.pivot_table()` |
| Format Cells (estilos, cores) | `openpyxl.styles` |

## 4. PDF (UiPath.PDF.Activities)

| Ação UiPath | Equivalente Python | Observação de segurança |
|---|---|---|
| Read PDF Text | `pdfplumber` ou `pypdf` ≥ 6.7.2 | `pypdf < 6.7.2` tem DoS por loop infinito (CVE-2026-27628). Use ≥ 6.7.2 |
| Read PDF With OCR | `ocrmypdf` ou `pdf2image` + `pytesseract` | — |
| Extract Images from PDF | `pikepdf` (baseado em qpdf, robusto contra PDFs malformados) | pikepdf ≥ 9.x |
| Merge / Split PDF | `pypdf` ≥ 6.7.2 ou `pikepdf` | — |
| Manipulate PDF (rotate, watermark) | `pypdf` ≥ 6.7.2 · `reportlab` para gerar do zero | — |
| Fill PDF Form | `pdfforms` ou `pikepdf` | — |

## 5. Email (UiPath.Mail.Activities)

| Ação UiPath | Equivalente Python |
|---|---|
| Get IMAP Mail Messages | `imapclient` + `email` (stdlib) |
| Get Outlook Mail Messages | `pywin32` (`win32com.client`) no Windows com Outlook instalado |
| Send SMTP Mail Message | `smtplib` + `email.mime` (stdlib) — sem dependências externas |
| Send Exchange / O365 | `O365` (pacote `python-o365`) via Microsoft Graph |
| Save Attachments | iteração em `message.iter_attachments()` |

## 6. Sistema, Arquivos e Workflow (UiPath.System.Activities)

| Ação UiPath | Equivalente Python |
|---|---|
| Copy File / Move File / Delete File | `shutil`, `os`, `pathlib` (stdlib) |
| Create Directory / Path Exists | `pathlib.Path.mkdir()`, `.exists()` |
| Read Text File / Write Text File | `pathlib.Path.read_text()` / `write_text()` |
| Wait / Delay | `time.sleep()` |
| If / Switch / For Each | construções nativas do Python |
| Try Catch | `try/except/finally` |
| Invoke Workflow / Invoke Code | `import` de módulos ou `subprocess.run()` |
| Log Message | `logging` (stdlib) |
| Get Environment Variable | `os.environ` ou `python-dotenv` para `.env` files |
| Run Python Script | nativo |

## 7. Dados e Programação (UiPath.System – Data activities)

| Ação UiPath | Equivalente Python |
|---|---|
| Build Data Table / Filter Data Table | `pandas.DataFrame` |
| For Each Row in Data Table | `df.iterrows()` ou `df.itertuples()` (mais rápido) |
| Merge Data Table / Join | `pandas.merge()`, `pandas.concat()` |
| Output Data Table (CSV) | `df.to_csv()` |
| Manipulação de strings (Trim, Split, Replace) | métodos nativos de `str` |
| Regex (Matches, Replace) | módulo `re` (stdlib) |
| Date manipulation | `datetime` (stdlib) ou `pendulum` |

## 8. HTTP / APIs (UiPath.WebAPI.Activities)

| Ação UiPath | Equivalente Python |
|---|---|
| HTTP Request | `httpx` (recomendado — suporta async/HTTP-2) ou `requests` |
| Deserialize JSON | `json.loads()` (stdlib) |
| SOAP Request | `zeep` |
| OAuth2 | `authlib` ou `requests-oauthlib` |

## 9. Banco de Dados (UiPath.Database.Activities)

| Ação UiPath | Equivalente Python |
|---|---|
| Connect / Execute Query / Execute Non-Query | `sqlalchemy` (engine universal) |
| SQL Server | `pyodbc` ou `pymssql` |
| Oracle | `oracledb` (substituiu cx_Oracle) |
| PostgreSQL | `psycopg` (v3) |
| MySQL | `mysqlclient` ou `mysql-connector-python` |
| ORM completo | `sqlalchemy` 2.x ou `SQLModel` |

## 10. OCR e Document Understanding (UiPath.DocumentUnderstanding)

| Ação UiPath | Equivalente Python |
|---|---|
| Digitize Document | `pdfplumber` + `pytesseract` ou `easyocr` |
| Classify Document | `transformers` (modelos LayoutLM, Donut) |
| Extract Data (forms / invoices) | `donut-python`, `layoutparser`, ou APIs (Azure Document Intelligence, AWS Textract) |
| Validate Extraction Results | lógica Python pura + `pydantic` para validação de schema |

## 11. Integrações com sistemas corporativos (Integration Service)

| Ação UiPath | Equivalente Python |
|---|---|
| SAP Integration | `pyrfc` (SAP NetWeaver RFC) ou automação UI via `botcity-framework-core` |
| Active Directory | `ldap3` |
| AWS | `boto3` |
| Azure | `azure-sdk-for-python` (vários subpacotes) |
| Google Workspace | `google-api-python-client` |
| Salesforce | `simple-salesforce` |
| ServiceNow | `pysnow` |
| Slack | `slack-sdk` |
| Microsoft Teams / Graph | `msgraph-sdk` |

## 12. Orquestração (UiPath Orchestrator)

| Funcionalidade UiPath | Equivalente no stack Python |
|---|---|
| Orchestrator (queues, assets, jobs, logs) | **BotCity Maestro** (orquestrador nativo Python) |
| Agendamento de bots | SDK ou API Maestro BotCity|
| Credentials / Assets | `keyring` (cofre local) + Maestro para corporativo |
| Fila de trabalho | filas do Maestro, ou `celery` + `redis` para volumes altos |