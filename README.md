# billy
AI Finance Manager Bot to track expenses, manage budgets, and automate your personal finances


# Billy: The Agentic Financial Coach ⏱️💸

**Turn your 2026 financial resolutions into reality with proactive, agentic coaching.**

> *Traditional apps perform a "financial autopsy"—telling you where things went wrong after the money is gone. [cite_start]Billy focuses on "financial health," intervening proactively to keep you on track.* [cite: 8]

![Billy Banner](https://img.shields.io/badge/Status-MVP-success) ![n8n](https://img.shields.io/badge/Orchestration-n8n-ff6e5c) ![OpenAI](https://img.shields.io/badge/AI-GPT--4o--mini-green) ![Telegram](https://img.shields.io/badge/Interface-Telegram-blue)

## 📖 Overview

**Billy** is a frictionless, AI-powered financial assistant that lives entirely in **Telegram**. [cite_start]It removes the barrier to entry for budgeting by allowing natural language logging and receipt scanning[cite: 20].

[cite_start]Unlike standard bots that annoy you with constant notifications, Billy uses **Agentic Silence**[cite: 32]. [cite_start]It reasons about your "Safe Spending Pace" vs. your "Actual Spending" and only speaks when you are at risk of breaking your budget, preserving your attention for when it matters most[cite: 33].

## ✨ Key Features

* **💬 Frictionless Ingestion:** Log expenses by texting ("Spent $15 on lunch") or sending a photo of a receipt. [cite_start]No new apps to install[cite: 29].
* [cite_start]**🧠 Intelligent Intent Classification:** Automatically distinguishes between setting a budget, logging a transaction, or general chatter[cite: 82].
* [cite_start]**👁️ AI Vision (OCR):** Uses GPT-4o-mini to scan receipts, extracting date, merchant, total amount, and line items automatically[cite: 100].
* [cite_start]**📉 Proactive Pacing Logic:** Calculates your **Time Ratio** (how far into the month you are) vs. **Spend Ratio** (how much budget is used)[cite: 31, 158].
    * [cite_start]*On Track:* Silence (No message sent)[cite: 172].
    * [cite_start]*Mild Risk:* Gentle nudge[cite: 174].
    * [cite_start]*High Risk:* Clear intervention with actionable advice[cite: 175].
* [cite_start]**📂 Automated Audit Trail:** Receipt images are automatically uploaded to Google Drive, and data is structured into a Google Sheet ledger[cite: 67].
* [cite_start]**🛡️ Agentic & Safe:** The agent helps you make decisions but has no direct access to move your funds[cite: 188].

## 🏗️ Architecture

[cite_start]Billy runs on a modular **n8n** workflow that decouples data ingestion (real-time) from reasoning (scheduled)[cite: 69, 78].

1.  [cite_start]**The Input (Telegram):** Receives text messages or images[cite: 70].
2.  [cite_start]**The Brain (GPT-4o-mini):** Classifies intent, extracts entities to JSON, and handles OCR for images[cite: 80].
3.  [cite_start]**The Ledger (Google Sheets):** Stores structured transaction data and budget limits[cite: 72].
4.  **The Conscience (Scheduled Async Agent):** Runs daily (e.g., 9:30 AM). [cite_start]It checks the month's progress against the budget and decides *if* it needs to speak based on a severity scale[cite: 75, 76].

## 🛠️ Tech Stack

* [cite_start]**Orchestration:** [n8n](https://n8n.io/) (Workflow Automation) [cite: 192]
* **LLM & Vision:** OpenAI (GPT-4o-mini) & Google Gemini (Flash-Lite for messaging).
* [cite_start]**Interface:** Telegram Bot API[cite: 195].
* [cite_start]**Database:** Google Sheets[cite: 198].
* [cite_start]**Storage:** Google Drive (for receipt images)[cite: 200].

## 🚀 Setup & Installation

### Prerequisites
* An instance of **n8n** (Cloud or Self-Hosted).
* **Telegram Bot Token** (via @BotFather).
* **OpenAI API Key**.
* **Google Cloud Service Account** (with Google Sheets and Drive API enabled).

### Step 1: Import the Workflow
1.  Download the `runcraft.json` file from this repository.
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
