# Lead Capture & Notification System (Demo Workflow)

🇺🇸 English | 🇺🇦 [Українська](README_UA.md)

---

## Overview

This project demonstrates a simple **Lead Capture & Notification automation workflow** built using **n8n**.

The workflow receives leads via a webhook, processes and prioritizes them using JavaScript, stores them in Google Sheets, and sends notifications via Telegram.

⚠️ This project was created as a **learning and demonstration workflow**, not a production-ready system.

The purpose of this project is to demonstrate:

- automation of lead processing
- lead prioritization logic
- validation of incoming data
- integration with external services
- basic error handling in automation workflows

---

## Workflow Architecture

![Workflow](workflow.png)

The workflow consists of the following steps:

1. Webhook receives lead data  
2. JavaScript node processes and prioritizes the data  
3. Lead is stored in Google Sheets  
4. Telegram notification is sent  
5. Error handling is triggered if something fails

---

## Step-by-Step Workflow

### 1. Webhook — Receive Lead Data

The **Webhook node** receives incoming lead data from external services such as forms or integrations.

Example payload:

```json
{
  "name": "John Doe",
  "email": "john@email.com",
  "phone": "+123456789",
  "source": "facebook",
  "message": "I am interested in your service"
}
```

---

### 2. JavaScript Node — Data Processing & Prioritization

The **Code node** performs several operations:

- data structuring
- validation of required fields
- lead prioritization
- timestamp generation

Priority logic:

- **Facebook / Instagram → High priority**
- **Website → Medium priority**
- **Other sources → Low priority**

Example code:

```javascript
const source = $json.body.source;

const priority = source === "facebook" || source === "instagram"
? "High"
: source === "website"
? "Medium"
: "Low";

const priority_telegram = source === "facebook" || source === "instagram"
? "🔥 High"
: source === "website"
? "⚡ Medium"
: "📩 Low";

if (!$json.body.name || !$json.body.email) {
  throw new Error("Missing required fields: name or email");
};

return {
  name: $json.body.name,
  email: $json.body.email,
  phone: $json.body.phone,
  source: $json.body.source,
  priority: priority,
  priority_telegram: priority_telegram,
  message: $json.body.message,
  timestamp: new Date().toISOString()
};
```

---

### 3. Google Sheets — Lead Storage

After processing, the lead is saved in **Google Sheets**, which is used as a simple database.

Stored fields include:

- name
- email
- phone
- source
- priority
- message
- timestamp

Google Sheets allows easy tracking and manual inspection of collected leads.

---

### 4. Telegram Notification

If the lead is successfully stored, a **Telegram notification** is sent.

Example notification:

```
🔔 New Lead!

Name: John Doe
Email: john@email.com
Phone: +123456789
Source: facebook
Priority: 🔥 High
Message: I am interested in your service
Time: 12.04.2026 14:21
```

This allows managers to react quickly to incoming leads.

---

### 5. Error Handling

The workflow includes simple error handling mechanisms.

#### Google Sheets Write Error

If writing to Google Sheets fails, a **Telegram alert** is sent with lead information and the error details.

Example:

```
⚠️ Google Sheets write error!

Lead phone: +123456789
Email: john@email.com
Time: 12.04.2026 14:21
Error: <error message>
```

---

#### Telegram Notification Error

If the Telegram notification fails:

- the lead remains saved in Google Sheets
- an **error record is written in the sheet**

This helps track notification failures and prevents data loss.

---

## Technologies Used

- **n8n** — workflow automation platform
- **Webhooks** — receiving lead data
- **JavaScript (Code Node)** — lead processing and prioritization
- **Google Sheets** — lead storage
- **Telegram Bot API** — notifications

---

## Project Purpose

This project was created as a **learning exercise** to practice:

- automation workflows
- API integrations
- data validation
- lead prioritization
- basic error handling in automation systems

---

## Possible Improvements

Future improvements could include:

- CRM integration
- duplicate lead detection
- automatic lead assignment to managers
- retry logic for failed notifications
- analytics dashboard
- advanced lead scoring
