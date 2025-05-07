# 💸 SpendSense - Automated Expense Tracking Workflow

This repository contains an n8n workflow designed to streamline expense tracking by capturing expense data from multiple channels, processing it using AI, storing it in Airtable, and generating automated monthly reports.

## Overview

The workflow is divided into two main sections:
1.  **Expense Data Capture & AI Extraction:** This part handles the collection of expense information from various sources like Email, a dedicated Web Form, and a Telegram Bot. It then uses Google Gemini AI to extract key details from the provided text.
2.  **Monthly Expense Report Generation & Delivery:** This section runs on a schedule (e.g., monthly) to gather all recorded expenses from Airtable, generate a comprehensive HTML report, and send it via email.

## Features

* **Multi-Channel Input:** Capture expenses seamlessly via:
    * **Email:** Monitors a specific Gmail account for expense-related emails.
    * **Web Form (SpendSense):** Allows manual entry of expense details and receipt image uploads.
    * **Telegram Bot:** Enables users to submit expenses via text messages, with an option to upload images.
* **AI-Powered Data Extraction:** Leverages Google Gemini AI to intelligently parse text descriptions and extract crucial information such as:
    * Amount
    * Date
    * Merchant/Vendor
    * Category
    * Payment Method
    * Notes
* **Data Cleaning & Validation:** Includes steps to clean and validate the JSON output from the AI for consistency.
* **Centralized Storage:** Saves all processed expense data to an Airtable base for easy access and management.
* **Automated Monthly Reporting:**
    * Triggers automatically on a predefined schedule (e.g., 9 AM on the 1st of each month).
    * Calculates the date range for the previous month.
    * Fetches all relevant expenses from Airtable.
    * Generates a detailed HTML report including:
        * Total spending
        * Breakdown by category
        * Top expenses
    * Delivers the report via Gmail to a specified email address.

**Important Note:** The AI processing currently uses only the textual input for data extraction and does not analyze the content of uploaded images.

## Workflow Breakdown

### Section 1: Expense Data Capture & AI Extraction

This section is triggered by incoming data from one of the configured input channels.

1.  **Input Triggers:**
    * `Gmail Trigger - Expense Emails`: Monitors a configured Gmail account for new emails. Configured to poll every minute.
    * `On form submission (SpendSense Form)`: A web form trigger that accepts a receipt file and a text description of the expense.
    * `Telegram Trigger`: (Implied by description, specific node details not in provided JSON) Accepts text input and optional image uploads from a Telegram bot.

2.  **Data Aggregation & Contextualization:**
    * (Implied) A step to aggregate text from the input source and add any necessary context for the AI.

3.  **AI-Powered Extraction:**
    * `Google Gemini AI Node`: (Implied by description, specific node details not in provided JSON) The aggregated text is sent to Google Gemini AI to extract structured expense data (Amount, Date, Merchant, Category, etc.).

4.  **Data Cleaning & Validation:**
    * `Code - Clean & Validate Email Extraction`: This node takes the raw output from the AI (expected to be a JSON string, possibly within Markdown code fences), cleans it up by removing extraneous characters, and parses it into a valid JSON object. It includes robust error handling for parsing issues.

5.  **Data Storage:**
    * `Airtable - Save Data`: (Implied by description, specific node for saving not detailed in provided JSON for this section, but is a logical next step and mentioned in the outcome) The cleaned and structured JSON data is then saved as a new record in an Airtable base.

### Section 2: Monthly Expense Report Generation & Delivery

This section runs on a predefined schedule.

1.  **Scheduled Trigger:**
    * `Schedule Trigger - Monthly Report`: Configured to run at 9 AM on the first day of every month.

2.  **Date Calculation:**
    * `Code - Calculate Prev Month Dates`: This node calculates the start and end dates of the previous calendar month (e.g., if run on July 1st, it will calculate June 1st to June 30th).

3.  **Fetch Expense Data:**
    * `Airtable - Get Prev Month Data`: Retrieves records from a configured Airtable base and table where the `Date` field falls within the calculated previous month's date range. Data is sorted by `submittedAt`.

4.  **HTML Report Generation:**
    * `Code - Generate HTML Report`: This sophisticated node processes the fetched expense data to generate a comprehensive HTML report. It includes:
        * Calculations for total spending, category totals, payment method totals, top merchants, average daily spending, and highest expense.
        * Formatting for currency (₹), dates, and text.
        * Generation of tables and visual bars for category and payment method breakdowns.
        * A summary section with key metrics.
        * A list of top N expenses.
        * Spending observations.
        * Enhanced HTML and CSS for a well-formatted email report.
        * Error handling for cases with no data.

5.  **Email Delivery:**
    * `Gmail - Send Report`: (Implied by description, specific node details not in provided JSON for sending email) The generated HTML report is sent via Gmail to a configured recipient email address.

## ✅ Outcome

* **Unified Expense Capture:** A single system to collect expenses from Email, a dedicated Form, and Telegram.
* **AI-Driven Detail Extraction:** Automated and intelligent extraction of expense details, reducing manual data entry.
* **Organized Storage:** All expenses are systematically stored in Airtable.
* **Automated Reporting:** Monthly expense reports are generated and delivered automatically, providing timely financial insights.

## Prerequisites

To use and set up this workflow, you will need:
* An n8n instance (cloud or self-hosted).
* Credentials for:
    * **Gmail:** For both triggering email reads and sending report emails (OAuth2).
    * **Airtable:** Personal Access Token or OAuth2 for accessing your base and table.
    * **Google Gemini AI:** API Key for the AI processing.
    * **Telegram:** A Telegram Bot Token if using the Telegram input channel.
* An Airtable base set up with the necessary fields (e.g., Amount, Date, Merchant, Category, Payment Method, Notes, submittedAt).

## Key n8n Nodes Used (based on provided JSON)

* `n8n-nodes-base.stickyNote`: For comments and documentation within the workflow.
* `n8n-nodes-base.gmailTrigger`: To start the workflow when new emails arrive.
* `n8n-nodes-base.formTrigger`: To start the workflow upon submission of a web form.
* `n8n-nodes-base.code`: For custom JavaScript code (data cleaning, date calculations, HTML report generation).
* `n8n-nodes-base.scheduleTrigger`: To run the workflow on a schedule.
* `n8n-nodes-base.airtable`: To interact with Airtable (fetch data).

*(Note: Some nodes like the Google Gemini AI node, Telegram Trigger, and the Gmail node for sending emails are inferred from the description as they are essential for the described functionality but not fully detailed in the provided JSON snippet.)*

## Setup & Customization

1.  **Import Workflow:** Import the workflow JSON into your n8n instance.
2.  **Configure Credentials:** Set up the necessary credentials for Gmail, Airtable, Google Gemini, and Telegram in n8n.
3.  **Update Node Parameters:**
    * **Gmail Trigger:** Select your Gmail credential and adjust filter settings (e.g., sender email, subject keywords) as needed.
    * **Form Trigger:** Note the webhook URL for your form.
    * **Code - Clean & Validate Email Extraction:** Review the `inputData` path (e.g., `$input.first().json.output`) and adjust if your AI node outputs data differently.
    * **Airtable - Get Prev Month Data:** Select your Airtable credential, and ensure the Base ID and Table ID match your setup. Verify the `filterByFormula` correctly references your date field.
    * **Code - Calculate Prev Month Dates:** No immediate changes typically needed unless date handling logic needs to be altered.
    * **Code - Generate HTML Report:**
        * Update `CURRENCY_SYMBOL` if needed.
        * Adjust `LOCALE` for date and number formatting.
        * Modify HTML/CSS if custom styling is desired.
    * **(Implied) AI Node:** Configure with your Google Gemini API key and desired model/prompt.
    * **(Implied) Airtable Save Node:** Configure with your Airtable credential, Base ID, Table ID, and map the fields correctly from the AI output.
    * **(Implied) Gmail Send Node:** Select your Gmail credential, set the recipient email, subject, and ensure the HTML body is correctly mapped from the report generation node.
4.  **Activate Workflow:** Once all configurations are set, activate the workflow.

Remember to test each part of the workflow thoroughly after setup.
