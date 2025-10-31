

<h1 align="center">📈 NSE NIFTY Stock Screener with Automated Trading and Google Sheets Integration</h1>

<p align="center">
  <b>Automated stock screening and trading workflow built using n8n, Zerodha Kite API, and Google Sheets</b>
</p>

<p align="center">
  <a href="https://n8n.io" target="_blank">
    <img src="https://img.shields.io/badge/Built%20with-n8n-00cc99?style=for-the-badge&logo=n8n&logoColor=white" alt="n8n Badge"/>
  </a>
  <a href="https://developers.kite.trade/" target="_blank">
    <img src="https://img.shields.io/badge/Trading%20API-Zerodha%20Kite-blue?style=for-the-badge&logo=zerodha" alt="Zerodha Badge"/>
  </a>
  <a href="https://workspace.google.com/marketplace/app/google_sheets/584050775912" target="_blank">
    <img src="https://img.shields.io/badge/Integrated%20with-Google%20Sheets-34A853?style=for-the-badge&logo=googlesheets&logoColor=white" alt="Google Sheets Badge"/>
  </a>
</p>

<p align="center">
  <a href="#-features">Features</a> •
  <a href="#-setup-guide">Setup</a> •
  <a href="#-workflow-overview">Workflow</a> •
  <a href="#-trading-logic-summary">Logic</a> •
  <a href="#-author">Author</a>
</p>

---

## 🚀 Features

- 🕒 **Automated Scheduling**
  - Runs every trading day at **09:30 AM IST** using an n8n Schedule Trigger.
  - Can also be triggered manually for instant screening.

- 📊 **Live Market Data Integration**
  - Fetches stock data from the **NSE India API** for:
    - NIFTY 50  
    - NIFTY Midcap 100  
    - NIFTY Smallcap 100
  - Merges all indices into a single dataset for analysis.

- 📈 **Technical Analysis**
  - Pulls price and volume data from **Yahoo Finance**.
  - Calculates:
    - **RSI (Relative Strength Index)**
    - **Volume Spike Detection**
    - **Support and Resistance Levels**
  - Generates clear signals:
    - `BUY` → RSI oversold / near support  
    - `SELL` → RSI overbought / near resistance  
    - `HOLD` → Neutral conditions

- ⚙️ **Customizable Settings**
  | Parameter | Default | Description |
  |------------|----------|-------------|
  | `autoTradingEnabled` | `false` | Enables/disables live trading |
  | `zerodhaApiKey` | `"YOUR_ZERODHA_API_KEY"` | Zerodha Kite Connect API key |
  | `zerodhaAccessToken` | `"YOUR_ZERODHA_ACCESS_TOKEN"` | Valid access token |
  | `googleSheetId` | *(empty)* | Google Sheet ID for trade logs |
  | `rsiOversoldThreshold` | `30` | RSI threshold for BUY |
  | `rsiOverboughtThreshold` | `70` | RSI threshold for SELL |
  | `volumeSpikeMultiplier` | `1.5` | Volume spike detection multiplier |

- 🧾 **Google Sheets Logging**
  - Appends screened stocks with key fields like:
    - `Stock Name`, `TIKR`, `CMP`, `Target`, `Stop Loss`, `Action`, and `Trigger Reason`.
  - Automatically updates timestamps and categories.

- 🤖 **Automated Trading (Zerodha Integration)**
  - Uses **Kite Connect API** to place live orders:
    ```
    POST https://api.kite.trade/orders/regular
    ```
  - Supports both **BUY** and **SELL** based on screening output.
  - Executes **LIMIT CNC** orders on **NSE**.
  - Disabled by default for safety.

---

## 🧩 Workflow Overview

### Node Sequence
1. **Schedule Stock Screening** – Triggers daily at 09:30 AM IST.  
2. **Workflow Configuration** – Loads API keys, thresholds, and flags.  
3. **Fetch NIFTY 50 Stocks**  
4. **Fetch NIFTY Midcap 100 Stocks**  
5. **Fetch NIFTY Smallcap 100 Stocks**  
6. **Merge All Stocks** – Combines all indices into one list.  
7. **Get Stock Quote Data** – Fetches last 1-month daily data from Yahoo Finance.  
8. **Apply Screening Logic** – Calculates RSI, detects volume spikes, and finds support/resistance.  
9. **Check If Opportunities Found** – Filters actionable stocks.  
10. **Update Google Sheet** – Logs results to the configured sheet.  
11. **Check Auto-Trading Enabled** – Ensures live trading toggle is ON.  
12. **Place Order via Zerodha Kite API** – Executes orders for `BUY` or `SELL` signals.

---

## ⚙️ Setup Guide

### 1. Import Workflow into n8n
- Open your n8n instance.
- Import the file:
````

NSE NIFTY Stock Screener with Automated Trading and Google Sheets Integration.json

```

### 2. Configure Credentials
- **Google Sheets OAuth2 API:** Connect your Google account in n8n.  
- **Zerodha Kite Connect API:**  
- Generate an API key and access token from [Zerodha Developer Console](https://developers.kite.trade/).  
- Update these in the **Workflow Configuration node**.

### 3. Prepare Google Sheet
Create a sheet with the following columns:
```

Sector | Stock Name | TIKR | CMP | Percentage change today | Entry | Target | Stop loss | Action | Trigger Reason

````
Then paste your sheet ID in the configuration node.

### 4. Optional: Enable Auto Trading
Set:
```bash
autoTradingEnabled = true
````

> ⚠️ Only after testing the workflow in paper mode.

### 5. Run the Workflow

* Test manually first.
* Enable schedule trigger to auto-run daily at 09:30 AM IST.

---

## 📊 Example Output (Google Sheet)

| Sector           | Stock Name      | TIKR       | CMP     | Target  | Stop Loss | Action | Trigger Reason                         |
| ---------------- | --------------- | ---------- | ------- | ------- | --------- | ------ | -------------------------------------- |
| NIFTY 50         | INFOSYS LTD     | INFY       | 1585.30 | 1640.00 | 1550.00   | BUY    | RSI oversold (28.4), Near support      |
| NIFTY Midcap 100 | TATA MOTORS LTD | TATAMOTORS | 972.40  | 1015.00 | 950.00    | SELL   | RSI overbought (75.2), Near resistance |

---

## 💹 Trading Logic Summary

| Condition             | Signal  | Description                    |
| --------------------- | ------- | ------------------------------ |
| RSI < 30              | BUY     | Oversold – possible reversal   |
| RSI > 70              | SELL    | Overbought – likely correction |
| Volume > 1.5× avg     | CONFIRM | High market participation      |
| Price near support    | BUY     | Bounce potential               |
| Price near resistance | SELL    | Possible rejection             |

---

## 🔐 Safety and Reliability

* **Paper Mode:** Enabled by default (`autoTradingEnabled = false`).
* **Error Handling:** Skips invalid or incomplete API data gracefully.
* **Logging:** All activity recorded in Google Sheets.
* **Sequential API Calls:** Prevents hitting rate limits on Yahoo Finance.

---

## 🧰 Dependencies

| Service                      | Purpose                    |
| ---------------------------- | -------------------------- |
| **n8n**                      | Workflow automation engine |
| **NSE API**                  | Fetch NIFTY indices        |
| **Yahoo Finance API**        | Historical & live data     |
| **Google Sheets API**        | Logging and tracking       |
| **Zerodha Kite Connect API** | Trade execution            |

---

## 🧑‍💻 Author

**Someshvar Sayapaneni Gopi**

🎓 M.Sc. in Communication Engineering with Machine Learning, Data Science and Artificial Intelligence – Aalto University

💡 Interests: Data Science, Machine Learning, DevOps and Intelligent Automation

🔗 [LinkedIn](https://www.linkedin.com/in/someshvar)

---

## ⚠️ Disclaimer

This project is for **educational and research purposes only**.
Stock trading involves financial risk. The author holds **no responsibility for losses** incurred through this workflow.
Always backtest and run in **paper mode** before live trading.




