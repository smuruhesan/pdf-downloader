

# 📚 Cortex Docs - PDF Downloader - User Guide

## 1. Overview

The **Cortex Docs PDF Downloader** is an automated web-scraping and document-generation tool built with Python, Streamlit, and Playwright.

It is specifically designed to interact with the Palo Alto Networks documentation portal (which uses the Fluid-Topics framework). The tool allows users to manage a list of documentation URLs via an Excel file, automatically extract hidden publication dates, and compile multi-page HTML manuals into clean, heavyweight PDF files without floating web widgets obstructing the text.

---
## 2. Prerequisites & Setup

To run this application, you need Python installed on your machine along with a few specific libraries.

**1. Install Required Python Libraries:**
Open your terminal and run:

```bash
pip install streamlit pandas playwright streamlit-aggrid openpyxl

```

**2. Install Playwright Browsers:**
The tool requires the Chromium browser engine to run in the background.

```bash
playwright install chromium

```

**3. Required File Structure:**
Your working folder must contain:

1. `app.py` (The Python script)
2. `sources.xlsx` (The Excel tracking file)

**Excel File Requirements:**
The first row of `sources.xlsx` must contain exactly these headers:

* `File Title` (Can be a clickable hyperlink or a plain text URL starting with `http`)
* `Version` (Used to store the date)
* `Status` (Tracks if it is Pending or Downloaded)

**To launch the app:**

```bash
streamlit run app.py

```

---

## 3. User Interface Guide

When you launch the app, you will see the following controls:

* **🔄 Reload from Excel:** Use this button if you updated the Excel file manually (or if Google Drive took a while to sync). It wipes the app's memory and pulls the freshest data from the spreadsheet.
* **🔍 Quick Search:** Type any word to instantly filter the table by filename, date, or status.
* **Download Filename ✏️:** Double-click any cell in this column to rename the file. The tool will automatically save the PDF using whatever name you type here.
* **💻 Live Execution Logs:** A real-time terminal window inside the app. It tells you exactly what the bot is doing at any given second, preventing you from guessing if the app has frozen.

---


## 4. System Architecture & Step-by-Step Workflow

This diagram breaks down the exact sequence of events from the moment you open the app to the moment your PDF is saved.

```mermaid
graph TD
    %% Define Styles
    classDef userAction fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef appProcess fill:#f3e5f5,stroke:#4a148c,stroke-width:2px;
    classDef botAction fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px;
    classDef dataSave fill:#fff3e0,stroke:#e65100,stroke-width:2px;

    %% Nodes
    Step1["1️⃣ Load Excel Data"]:::appProcess
    Step2["2️⃣ User Selects Guides"]:::userAction
    
    %% Split Actions
    Step3A["3A. Click 'Fetch Dates'"]:::userAction
    Step3B["3B. Click 'Download PDFs'"]:::userAction

    %% Date Flow
    Step4A["4. Bot Opens Hidden Browser"]:::botAction
    Step5A["5. Scraper Pierces Shadow DOM"]:::botAction
    
    %% PDF Flow
    Step4B["4. Bot Opens Hidden Browser"]:::botAction
    Step5B["5. Bot Triggers 'Print All'"]:::botAction
    Step6B["6. Widget Assassin Cleans Page"]:::botAction
    Step7B["7. Binary PDF Generated"]:::botAction

    %% Final Save
    Step8["8️⃣ Save to Excel & Disk"]:::dataSave

    %% Connections
    Step1 --> Step2
    Step2 --> Step3A
    Step2 --> Step3B
    
    Step3A --> Step4A
    Step4A --> Step5A
    Step5A --> Step8
    
    Step3B --> Step4B
    Step4B --> Step5B
    Step5B --> Step6B
    Step6B --> Step7B
    Step7B --> Step8

```

### The Workflow Explained

Here is exactly what happens under the hood during each step of the process:

* **Step 1: Load Excel Data:** When you launch the Streamlit app, it automatically reads your `sources.xlsx` file. It cleans up the "File Title" column to remove illegal characters and `.pdf` extensions, preparing them for download.
* **Step 2: User Selects Guides:** The UI displays an interactive table. You use the checkboxes to select which documents you want to process and can optionally rename the output files directly in the grid.

<img width="1394" height="769" alt="image" src="https://github.com/user-attachments/assets/4aa16100-74c9-4e28-bd07-100386a57bf4" />

* **Step 3: Trigger Automations:** You click either the **Fetch Dates** or **Download PDFs** button.

**If you clicked "Fetch Dates":**

* **Step 4 (Dates):** The Playwright bot launches an invisible Google Chrome browser in the background and navigates to the extracted URL.
* **Step 5 (Dates):** Because enterprise sites hide dates, the bot executes a custom JavaScript function to crawl through the invisible "Shadow DOM" components, extracting the raw text and using a Regex pattern to find the exact publication date. It extracts for each documents. Wait for it to finish fetching the Released/Published dates for all the document links from the Source file.
  
<img width="999" height="716" alt="image" src="https://github.com/user-attachments/assets/c4955064-8fa1-4837-9b63-ae5e8b4cb819" />
<img width="833" height="725" alt="image" src="https://github.com/user-attachments/assets/4a769d0f-97b7-4305-a25f-813944c79711" />
<img width="1285" height="609" alt="image" src="https://github.com/user-attachments/assets/c625f7be-1355-4622-93b7-87463ab310d3" />


**If you clicked "Download PDFs":**

* **Step 4 (PDFs):** The bot launches the invisible browser, navigates to the URL, and waits for the network to fully load.
<img width="1030" height="683" alt="image" src="https://github.com/user-attachments/assets/fbfbb053-7799-4b45-8d4b-2166baac6565" />

* **Step 5 (PDFs):** The bot clicks the on-screen "Print Map" icon, force-checks the "Select All" box, and waits up to 5 minutes for the massive document to compile in a "Ghost Tab".
* **Step 6 (PDFs):** The **Widget Assassin** activates. It injects CSS into the page to permanently hide floating chatboxes, "Scroll to Top" arrows, and feedback icons so they don't block the text on your PDF.
* **Step 7 (PDFs):** The clean page is converted into an A4-sized PDF document.

**The Final Step:**

* **Step 8: Save to Excel & Disk:** For dates, the `Version` column in Excel is updated. For PDFs, the file is saved to your hard drive and the `Status` column in Excel is updated to "✅ Downloaded". The UI terminal logs "SUCCESS" and refreshes the table.

```
```
---

## 5. How the Code Works (Technical Breakdown)

Here is a detailed explanation of the core mechanics inside `app.py`.

### A. Data Management Engine (`load_excel_data`)

* **What it does:** Reads the `sources.xlsx` file.
* **Smart Extraction:** It looks at the `File Title` column. If the text is a clickable hyperlink, it extracts the hidden URL. If it's just raw text starting with "http", it extracts that instead.
* **Sanitization:** It looks for `.pdf` in the filename and strips it out, replacing illegal characters (`/`, `\`, `*`, etc.) so Windows/Mac won't crash when saving the file.

### B. The "Fetch Dates" Engine

Enterprise documentation sites often hide publication dates inside backend metadata rather than printing them plainly on the screen.

* **Shadow DOM Piercing:** The script injects custom JavaScript (`shadow_js`) into the page. This script recursively walks through every HTML node and "Shadow Root" (hidden web components) to pull out all raw text.
* **Regex Extraction:** Python then uses Regular Expressions (`re.search`) to scan the massive block of text for keywords like `Publication Date:`, `ft:lastEdition:`, or `Updated:`, grabbing the date attached to it.
* **Formatting:** It passes the date to `format_date()`, which uses Pandas to intelligently convert formats like `2026-05-03` into `May 03, 2026`.

### C. The "Heavyweight PDF Download" Engine

This is the most complex part of the code, designed to bypass enterprise bot-protection and handle massive 100MB+ document compilations.

1. **Automated Navigation:** The Playwright bot opens the URL and uses JavaScript to locate the hidden "Print Map" icon inside the website's framework.
2. **Context Interception (Ghost Tabs):** When printing a full manual, the website spawns a new tab. The script uses `context.expect_page()` to physically "catch" this new tab in memory so it can control it.
3. **Heavyweight Timeout:** Compiling a 500-page document takes immense memory. The bot is programmed to wait up to **5 full minutes** (`timeout=300000`) for the website's network to go idle before it assumes the document is ready.
4. **The "Widget Assassin":** Standard PDF printing captures floating "Chat" or "Give Feedback" icons, blocking text. The script deploys a defense mechanism before printing:
* It injects CSS to force all `fixed` and `sticky` elements to disappear.
* It runs a recursive JavaScript loop to dig into all closed Shadow DOMs and hide elements matching Palo Alto's specific widget classes (e.g., `ft-feedback-button`, `#pendo-base`).


5. **PDF Generation:** Once the page is clean, it emulates "Print" media and saves the binary file directly to your hard drive, updating the Excel status to `✅ Downloaded`.

---

## 6. Best Practices & Troubleshooting

* **Skipping Files:** If a file is marked `✅ Downloaded` in the Excel file, the PDF engine will automatically skip it to save time. If you want to force a re-download, open Excel, delete the `✅ Downloaded` text, save the file, click **Reload from Excel** in the app, and run it again.
* **Handling Crashes ("Context Destroyed"):** If the log shows `ERROR: Context Destroyed`, it means the document was so massive that the underlying browser ran out of memory and refreshed the page. The script will safely skip it and move to the next file without crashing the whole application.
* **Background Operation:** The bot runs in `headless=True` mode, meaning the Chrome browser operates entirely invisibly in your computer's RAM. You can continue to use your computer normally while the app downloads PDFs in the background.
