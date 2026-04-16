# 🤖 Billing BOT

> Automated billing system that identifies overdue customers, generates payment slips/links, and sends reminders via WhatsApp.






***

## 📋 About

**Billing BOT** is a Python automation that runs a three-step pipeline on a schedule:

1. **Check overdue customers** — reads and filters customers past their due date from a CSV or SQLite database using pandas
2. **Generate payment slip / link** — integrates with the EFÍ Bank (Gerencianet) API via requests to issue bank slips or Pix charges
3. **Send WhatsApp reminder** — dispatches personalized messages to each overdue customer via PyWhatKit

The bot runs on a configurable schedule, records structured logs, and automatically updates each billing record's status.

***

## 🏗️ Project Structure

```
billing-bot/
├── src/
│   ├── __init__.py
│   ├── main.py               # Entry point / scheduler
│   ├── checker.py            # Overdue customer reader & filter
│   ├── payment.py            # Boleto/Pix generation via API
│   ├── whatsapp.py           # WhatsApp message dispatcher
│   └── logger.py             # Log configuration (loguru)
├── data/
│   ├── customers.csv         # Sample customer database
│   └── billing.db            # SQLite (auto-generated)
├── tests/
│   ├── test_checker.py
│   ├── test_payment.py
│   └── test_whatsapp.py
├── .env.example              # Environment variables template
├── .gitignore
├── requirements.txt
├── pyproject.toml
└── README.md
```

***

## ⚙️ Execution Flow

```
[Scheduler (APScheduler)]
         │
         ▼
[1. checker.py]
  Reads customers.csv / SQLite via pandas
  Filters due_date < today AND status != "paid"
         │
         ▼
[2. payment.py]
  For each overdue customer:
  → Calls EFÍ Bank API via requests
  → Receives boleto URL / Pix QR Code
  → Saves link to the database
         │
         ▼
[3. whatsapp.py]
  For each customer with a generated link:
  → Builds personalized message
  → Sends via PyWhatKit
         │
         ▼
[4. logger.py]
  Logs each step result
  Updates customer status in the database
```

***

## 🧰 Tech Stack

| Layer | Library | Version |
|---|---|---|
| Data reading & manipulation | `pandas` | 2.x |
| HTTP / API requests | `requests` | 2.x |
| WhatsApp messaging | `pywhatskit` | latest |
| Task scheduling | `APScheduler` | 3.x |
| Environment variables | `python-dotenv` | 1.x |
| Structured logging | `loguru` | 0.7.x |
| Local database | `sqlite3` + `SQLAlchemy` | built-in / 2.x |
| Config validation | `pydantic-settings` | 2.x |
| Testing | `pytest` | 8.x |

***

## 🚀 Getting Started

### Prerequisites

- Python 3.10+
- pip or uv
- [EFÍ Bank (Gerencianet)](https://efipay.com.br/) account for API credentials
- WhatsApp Web active in the browser (required by PyWhatKit)

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/your-username/billing-bot.git
cd billing-bot

# 2. Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
.venv\Scripts\activate     # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Set up environment variables
cp .env.example .env
# Edit .env with your credentials

# 5. Run the bot
python src/main.py
```

***

## 🔐 Environment Variables

Create a `.env` file at the project root based on `.env.example`:

```env
# === Payment API (EFÍ Bank / Gerencianet) ===
EFI_CLIENT_ID=your_client_id
EFI_CLIENT_SECRET=your_client_secret
EFI_SANDBOX=true   # set to false in production

# === WhatsApp Settings ===
WHATSAPP_WAIT_TIME=15   # seconds to wait for WhatsApp Web to open

# === Database ===
DATABASE_URL=sqlite:///data/billing.db

# === Scheduling ===
# Cron format: minute hour day_of_month month day_of_week
SCHEDULE_CRON=0 9 * * 1-5   # Weekdays at 9:00 AM

# === General ===
LOG_LEVEL=INFO
LOG_FILE=logs/bot.log
DRY_RUN=false   # true = simulate without sending messages
```

***

## 📊 Customer Data Format

The `data/customers.csv` file must follow this structure:

```csv
id,name,phone,email,amount,due_date,status
1,John Smith,5511999990001,john@email.com,350.00,2026-04-10,pending
2,Jane Doe,5511999990002,jane@email.com,120.00,2026-03-28,pending
3,Bob Jones,5511999990003,bob@email.com,890.00,2026-04-15,paid
```

| Field | Type | Description |
|---|---|---|
| `id` | int | Unique identifier |
| `name` | string | Customer full name |
| `phone` | string | Number with country code (55) + area code + number |
| `email` | string | Customer e-mail |
| `amount` | float | Outstanding amount in BRL |
| `due_date` | date | Due date (YYYY-MM-DD) |
| `status` | string | `pending`, `paid`, `sent`, `error` |

***

## 📝 Usage Example

```python
# Run a billing cycle manually for testing
from src.checker import get_overdue_customers
from src.payment import generate_charge
from src.whatsapp import send_reminder

# Fetch overdue customers
overdue = get_overdue_customers("data/customers.csv")
print(f"[INFO] {len(overdue)} overdue customers found")

# Process each customer
for customer in overdue:
    link = generate_charge(customer)
    send_reminder(customer, link)
```

**Expected output:**
```
[INFO] 2 overdue customers found
[INFO] Charge generated for John Smith → https://pix.efipay.com.br/...
[INFO] WhatsApp sent to +5511999990001 ✓
[INFO] Charge generated for Jane Doe → https://pix.efipay.com.br/...
[INFO] WhatsApp sent to +5511999990002 ✓
```

***

## 🗺️ Roadmap

| Milestone | Version | Status |
|---|---|---|
| Proof of Concept — CSV reading + terminal logging | v0.1 | 🔄 In Progress |
| MVP — payment link generation + WhatsApp sending | v0.2 | ⏳ Backlog |
| Scheduler, full logs, `.env`, complete docs | v1.0 | ⏳ Backlog |
| Bug fixes & polish post-release | v1.1 | ⏳ Backlog |
| Web dashboard + reports | v2.0 | ⏳ Backlog |

***

## 🧪 Testing

```bash
# Run all tests
pytest

# With coverage report
pytest --cov=src tests/

# Specific test file
pytest tests/test_checker.py -v
```

***

## 🤝 Contributing

1. Fork the project
2. Create your feature branch: `git checkout -b feature/feature-name`
3. Commit your changes: `git commit -m 'feat: add new feature'`
4. Push to the branch: `git push origin feature/feature-name`
5. Open a Pull Request

> Every contribution must have an **associated issue** before development begins.

***

## 📄 License

Distributed under the MIT License. See `LICENSE` for more information.

***

## 📬 Contact

Project managed via **GitHub Projects** with an issue-based methodology.  
Open an [issue](https://github.com/your-username/billing-bot/issues) to report bugs, suggest features, or ask questions.
