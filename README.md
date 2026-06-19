Here is your complete, step-by-step **Requirements and Operations Guide**. You can save this as a `README.md` file or a text document in the same folder as your script so you always have it handy.

---

# 📚 Cortex Admin Guide Manager - Operations Guide

This application is a highly advanced, automated web scraper and PDF generator built to bypass enterprise documentation frameworks (like Shadow DOMs and dynamic ghost tabs) to extract dates and full PDF manuals from Palo Alto Networks' documentation portal.

## ⚙️ 1. Prerequisites & Setup

Whenever you set this up on a new computer or environment, you must install the required dependencies.

**1. Install Python Libraries:**
Open your terminal and run:
```bash
pip install streamlit pandas playwright streamlit-aggrid openpyxl
```

**2. Install Playwright Browsers:**
Because the bot needs a real Chromium browser to run the background tasks, you must install Playwright's local browser binaries by running:
```bash
playwright install chromium
```

## 📁 2. Required File Structure

Your folder must contain exactly two files for the app to launch:

1. `app.py` (The main Python script provided above)
2. `sources.xlsx` (Your Excel tracking file)

**Critical Rules for `sources.xlsx`:**
* The sheet must contain the following exact column headers in row 1: **`File Title`**, **`Version`**, and **`Status`**.
* The **`File Title`** cells *must* contain the actual hyperlink to the specific Palo Alto documentation page. (Do not link to "Page Moved" redirects).

## 🚀 3. How to Run the Application

1. Open your terminal.
2. Navigate to the folder containing your `app.py` and `sources.xlsx` files.
3. Execute the following command:
```bash
streamlit run app.py
```
4. A browser window will automatically open to `http://localhost:8501` displaying the application interface.

## 🖱️ 4. How to Use the Interface

### Step 1: Search and Select
* **Quick Search:** Use the search bar at the top to instantly filter your massive Excel list by name, date, or status.
* **Select Guides:** Click the checkboxes on the left side of the table to select which documents you want to process. You can check the box in the header to "Select All" currently filtered rows.

### Step 2: Rename Output Files (Optional)
* To prevent duplicate `.pdf` extensions or confusing file names, double-click any cell in the **`Download Filename ✏️`** column.
* Type your desired file name and press `Enter`. The bot will save the PDF using this exact name.

### Step 3: Run Automations
* **🔍 Fetch Dates:** Clicks through the selected URLs, cracks the Shadow DOM, and attempts to extract the "Last Updated" or "Published" date, saving it directly back to your Excel file.
* **⬇️ Download PDFs:** Forces the Palo Alto website to compile the full manual, removes any floating widgets/chatboxes from the screen, and saves a clean PDF to your local folder.

## ⚠️ Important Operational Rules (The "Do Nots")

1. **DO NOT touch the automated Chrome window!** When you click "Download PDFs," a separate, visible Google Chrome window will pop up. The bot is actively hijacking the mouse and keyboard inputs inside that window. If you minimize it, click inside it, or close its tabs, the automation will fail. Just let it run in the background.
2. **Be Patient on Downloads:** Compiling a 500-page enterprise manual takes the website's backend a lot of time. The bot is programmed to wait up to 45 seconds for a document to render. Watch the "Live Execution Logs" terminal in the app to see exactly what the bot is doing at any given second. 
3. **Keep the App Open:** Do not close the Streamlit browser tab until the Live Execution Logs indicate the job is "SUCCESS" or "Job finished."