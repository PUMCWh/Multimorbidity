# Multimorbidity
# ICD‑10 Code Validation Tool

A lightweight Python utility that batch‑checks whether ICD‑10 codes assigned to Chinese disease names are correct by querying the Alibaba **DashScope** large‑language‑model (LLM) API. It appends the validation outcome to your original spreadsheet, making it easy to spot and fix mismatches.

---

## Features

* **Bulk verification** of disease name ⇄ ICD‑10 code pairs from an Excel sheet.
* **LLM‑powered** judgment using DashScope `Application.call`.
* Graceful error handling with detailed diagnostics when the API returns a non‑200 status.
* Progress bar via **tqdm**.
* Minimal rate‑limit protection with a configurable pause between calls.

---

## Folder structure

```
.
├── validate_icd10.py   # The main script (rename as you like)
├── requirements.txt    # Python dependencies
└── README.md           # You are here 🙂
```

> **Heads‑up:** The script currently hard‑codes input/output paths at the top. Tweak them—or wire up CLI arguments—before running.

---

## Prerequisites

* Python ≥ 3.8
* An Alibaba **DashScope** account and an **app‑specific key** (`api_key`) plus **app ID** (`app_id`).
* The following Python libraries (install via `pip install -r requirements.txt`):

  * pandas
  * openpyxl
  * dashscope
  * tqdm

---

## Installation

```bash
# 1) Grab the code
$ git clone https://github.com/<your‑org>/<your‑repo>.git
$ cd <your‑repo>

# 2) (Recommended) Create and activate a virtual environment
$ python -m venv .venv
$ source .venv/bin/activate  # Windows: .venv\Scripts\activate

# 3) Install dependencies
$ pip install -r requirements.txt
```

---

## Configuration

Set your DashScope credentials as **environment variables** to avoid hard‑coding secrets:

```bash
export DASH_API_KEY="sk‑xxxxxxxxxxxxxxxx"
export DASH_APP_ID="your_app_id"
```

On Windows PowerShell:

```powershell
setx DASH_API_KEY "sk‑xxxxxxxxxxxxxxxx"
setx DASH_APP_ID  "your_app_id"
```

Restart the terminal so the variables are picked up.

Next, open `validate_icd10.py` and adjust:

```python
INPUT_FILE  = r"C:\path\to\EHR_df_judgment.xlsx"
INPUT_SHEET = "无法匹配"          # Sheet name containing pairs to check
OUTPUT_FILE = r"C:\path\to\EHR_df_judgment_llm.xlsx"
OUTPUT_SHEET = "验证结果"          # Where results will be written
SLEEP_TIME = 0.001               # Delay (seconds) between API calls
```

---

## Usage

```bash
python validate_icd10.py
```

If everything is set up, you’ll see a progress bar like:

```
Processing rows: 100%|██████████| 2500/2500 [00:38<00:00, 64.94it/s]
```

When the run finishes, the script writes a new sheet — or updates an existing one — named **“验证结果”** in `EHR_df_judgment_llm.xlsx`. Each row gets a value like:

* `正确`                      → the code is correct.
* `不正确，建议编码：E11.900` → the model proposes a more appropriate ICD‑10 code.
* `API调用失败: …`             → the request hit an exception.
* A multi‑line error message → non‑200 HTTP status, see details inside.

---

## Example

| 疾病名称 (Disease) | 疾病编码 (Existing) | 验证结果 (Validation) |
| -------------- | --------------- | ----------------- |
| 2型糖尿病          | E11             | 正确                |
| 乙型肝炎           | B15             | 不正确，建议编码：B16      |

---

## Customisation ideas

* Swap the hard‑coded paths for **CLI arguments** using `argparse`.
* Cache verified pairs in a local database to avoid repeat API calls.
* Throttle requests automatically based on `response.headers.get('X‑RateLimit‑Remaining')` (if available).
* Add unit tests with `pytest` and mock the DashScope client.

---

## Cost & rate limits

DashScope usage is billed per token. Large batches can rack up charges, so consult your pricing plan and consider adding a **dry‑run** flag for small‑scale testing.

---

## Contributing

Pull requests, issues, and feature ideas are welcome! Please open an issue before major refactors so we can discuss direction.

---

## License

MIT © \<Weihao Shao 2025>
