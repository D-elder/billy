# Billy
AI Finance Manager Bot to track expenses, manage budgets, and automate your personal finances

# Billy: The Agentic Financial Coach ⏱️💸

**Turn your 2026 financial resolutions into reality with proactive, agentic coaching.**

> *Traditional apps perform a "financial autopsy"—telling you where things went wrong after the money is gone. Billy focuses on "financial health," intervening proactively to keep you on track.*

![Billy Banner](https://img.shields.io/badge/Status-MVP-success) ![n8n](https://img.shields.io/badge/Orchestration-n8n-ff6e5c) ![OpenAI](https://img.shields.io/badge/AI-GPT--4o--mini-green) ![Telegram](https://img.shields.io/badge/Interface-Telegram-blue)

## 📖 Overview

**Billy** is a frictionless, AI-powered financial assistant that lives entirely in **Telegram**. It removes the barrier to entry for budgeting by allowing natural language logging and receipt scanning.

Unlike standard bots that annoy you with constant notifications, Billy uses **Agentic Silence**. It reasons about your "Safe Spending Pace" vs. your "Actual Spending" and only speaks when you are at risk of breaking your budget, preserving your attention for when it matters most.

## ✨ Key Features

* **💬 Frictionless Ingestion:** Log expenses by texting ("Spent $15 on lunch") or sending a photo of a receipt. No new apps to install.
* **🧠 Intelligent Intent Classification:** Automatically distinguishes between setting a budget, logging a transaction, or general chatter.
* **👁️ AI Vision (OCR):** Uses GPT-4o-mini to scan receipts, extracting date, merchant, total amount, and line items automatically.
* **📉 Proactive Pacing Logic:** Calculates your **Time Ratio** (how far into the month you are) vs. **Spend Ratio** (how much budget is used).
    * *On Track:* Silence (No message sent).
    * *Mild Risk:* Gentle nudge.
    * *High Risk:* Clear intervention with actionable advice.
* **📂 Automated Audit Trail:** Receipt images are automatically uploaded to Google Drive, and data is structured into a Google Sheet ledger.
* **🛡️ Agentic & Safe:** The agent helps you make decisions but has no direct access to move your funds.

## 🏗️ Architecture

Billy runs on a modular **n8n** workflow that decouples data ingestion (real-time) from reasoning (scheduled).

1.  **The Input (Telegram):** Receives text messages or images.
2.  **The Brain (GPT-4o-mini):** Classifies intent, extracts entities to JSON, and handles OCR for images.
3.  **The Ledger (Google Sheets):** Stores structured transaction data and budget limits.
4.  **The Conscience (Scheduled Async Agent):** Runs daily (e.g., 9:30 AM). It checks the month's progress against the budget and decides *if* it needs to speak based on a severity scale.

## 🛠️ Tech Stack

* **Orchestration:** [n8n](https://n8n.io/) (Workflow Automation)
* **LLM & Vision:** OpenAI (GPT-4o-mini) & Google Gemini (Flash-Lite for messaging).
* **Interface:** Telegram Bot API.
* **Database:** Google Sheets.
* **Storage:** Google Drive (for receipt images).

## 🚀 Setup & Installation

### Prerequisites
* An instance of **n8n** (Cloud or Self-Hosted).
* **Telegram Bot Token** (via @BotFather).
* **OpenAI API Key**.
* **Google Cloud Service Account** (with Google Sheets and Drive API enabled).

### Step 1: Import the Workflow
1.  Download the `billy.json` file from this repository.
2.  Open your n8n instance.
3.  Click **Import workflow** (top right) and select the JSON file.

### Step 2: Configure Credentials
You need to set up the following credentials in n8n:
* `Telegram Api`: Your Bot Token (Referenced as "BillsBilly" in workflow).
* `OpenAi Api`: Your OpenAI Key.
* `Google Sheets OAuth2 API`: Authenticated Google account.
* `Google Drive OAuth2 API`: Authenticated Google account.
* `Google PaLM API` (Optional): If using Gemini for message generation as configured in the "Message a model" nodes.

### Step 3: Set Up Google Sheet
Create a new Google Sheet with two tabs:
1.  **Ledger:** Columns: `Transaction Type`, `Date`, `Amount`, `Invoice Id`, `Source or Reason`, `Category`, `Sub category`, `Merchant`, `essentiality`, `Budget bucket`.
2.  **Monthly Data:** Columns for `Global Key`, `Global Value`, `Month`, `Budget`, `Spent`.

*Update the "Sheet Name" and "Document ID" in all Google Sheets nodes in the workflow to match your specific sheet.*

### Step 4: Hardcoded ID Configuration ⚠️
To ensure privacy, the workflow filters for specific Telegram User IDs.
1.  Find the **"Authenticate Chat ID"** node (Node ID: `4de38a4b...`).
2.  Change the `rightValue` to your personal Telegram User ID.
3.  Update the **"Send a text message"** nodes that have hardcoded Chat IDs (look for the sticky notes in the workflow indicating where to change this).

## 📊 How the Logic Works

Billy uses a simple but powerful heuristic to determine intervention:

```javascript
// Simplified Logic from the "Compute Pace" Node
const timeRatio = daysElapsed / daysInMonth;
const spendRatio = spent / budget;

if (spendRatio > 0.9 && timeRatio < 0.9) {
   severity = "High Risk"; // Clear Intervention
} else if (spendRatio > timeRatio) {
   severity = "Mild Risk"; // Gentle Nudge
} else {
   severity = "On Track"; // Silence
}
