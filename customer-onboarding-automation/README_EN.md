![n8n](https://img.shields.io/badge/n8n-automation-orange)
![CRM](https://img.shields.io/badge/CRM-Airtable-yellow)
![Automation](https://img.shields.io/badge/Workflow-Onboarding-blue)

# Customer Onboarding Automation (n8n Workflow)

🇺🇸 English | 🇺🇦 [Українська](README_UA.md) 

---

## Overview

This project demonstrates an **automation workflow for customer onboarding**, built using **n8n** and **Airtable**.

The system receives customer data via a webhook, checks whether the customer already exists in the CRM, and automatically:

- creates a new customer or updates an existing one  
- sends an email to the customer  
- notifies a manager via Telegram  
- handles errors using a fallback system  

This project was created as a **learning / demo CRM onboarding automation system** to demonstrate:

- CRM automation
- lead deduplication
- error handling
- fallback logic
- email automation
- multi-branch workflows

---

## Workflow Architecture

![Workflow](workflow.png)

The workflow consists of the following stages:

1. **Webhook** — receiving an array of customers  
2. **customer-info** — converting the array into individual objects  
3. **HTTP Request (Airtable API)** — checking if the customer exists  
4. **Error Handling Flow** — fallback in case of API failure  
5. **existRecords** — checking if a record exists  
6. **IF Condition**
   - **True** → existing customer  
   - **False** → new customer  
7. **Airtable Update / Create** — updating or creating a record  
8. **Email Automation (Gmail)** — sending an email to the customer  
9. **Telegram Notification** — notifying the manager  
10. **Wait + Loop** — controlling error processing  

---

## How the System Works

### 1. Webhook

The workflow receives customer data through a **Webhook**.

For testing purposes:

```
https://reqbin.com
```

---

### 2. Data Processing (customer-info)

The node converts the array into individual objects.

```javascript
const data = $input.all();

const body = data.flatMap(item =>
  item.json.body.map(itemBody => ({
    name: itemBody.name,
    phone: itemBody.phone,
    email: itemBody.email,
    source: itemBody.source,
    message: itemBody.message
  }))
);

return body;
```

---

### 3. CRM Check

An HTTP Request is sent to the **Airtable API**.

Using:

```
filterByFormula
```

```
AND({Email} = "{{ $json.email }}", {Phone} = {{ $json.phone }})
```

---

## Error Handling (Fallback Flow)

If the **Airtable API is unavailable**:

1. **Loop Over Items (batch = 1)** is triggered  
2. A **timestamp** is generated  
3. Data is saved to **Google Sheets**  
4. The field `ErrorType` is set to:

```
API-Airtable
```

5. A Telegram alert is sent:

```
‼️ Error-API-Airtable
```

6. A delay is added (**Wait node**)  
7. The process repeats for each affected customer  

Even if Google Sheets fails, Telegram will still receive the alert.

---

## 4. Record Check

```javascript
const records = $json.records;

return {
  exist: records.length > 0,
  recordId: records.length > 0 ? records[0].id : null
};
```

---

## 5. Conditional Logic

The **IF node** checks:

```
recordId is not empty
```

---

## If the Customer Exists

### Data Preparation

```javascript
const timestamp = new Date().toISOString();

return {
  name: $('customer-info').item.json.name,
  phone: $('customer-info').item.json.phone,
  email: $('customer-info').item.json.email,
  source: $('customer-info').item.json.source,
  message: $('customer-info').item.json.message,
  recordId: $('existRecords').item.json.recordId,
  timestamp: timestamp
};
```

---

### CRM Update

- the Airtable record is updated  
- status is changed to:

```
In Progress
```

---

### Email

The customer receives an email:

- confirmation that the request is being processed  

---

### Telegram

The manager receives a notification:

```
follow-up notification
```

---

## If the Customer is New

### Data Preparation

```javascript
const timestamp = new Date().toISOString();

return {
  name: $('customer-info').item.json.name,
  phone: $('customer-info').item.json.phone,
  email: $('customer-info').item.json.email,
  source: $('customer-info').item.json.source,
  message: $('customer-info').item.json.message,
  timestamp: timestamp
};
```

---

### CRM Create

- a new record is created in Airtable  
- status:

```
New
```

---

### Email (Onboarding)

The customer receives a welcome email:

- introduction to the company  
- basic onboarding information  

---

## Technologies Used

- **n8n** — automation workflow platform  
- **Webhook** — data ingestion  
- **JavaScript (Code nodes)** — data processing  
- **Airtable API** — CRM database  
- **HTTP Request** — integration  
- **Google Sheets API** — fallback storage  
- **Gmail API** — email automation  
- **Telegram Bot API** — notifications  
- **Loop + Wait** — execution control  

---

## Possible Improvements

- lead scoring system  
- automatic customer assignment  
- retry mechanism for Airtable  
- analytics dashboard  
- SLA tracking for managers  

---

## Setup Notes

This workflow is a **demo CRM onboarding automation system**.

To run it, you need to:

- configure an **Airtable API Token**  
- create an **Airtable Base**  
- connect **Google Sheets**  
- configure **Gmail integration**  
- connect a **Telegram Bot**  
- import the workflow into **n8n**
